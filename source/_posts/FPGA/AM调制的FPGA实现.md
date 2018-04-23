---
title: AM调制的FPGA实现
date: 2018-02-09 19:05:44
categories:
- FPGA
tags:
- FPGA
- AM
- 调制解调
comments: true
---

## 一、说明：

1. 功能：AM调制
2. 平台：Vivado 2016.4 和 Matlab R2017a

## 二、原理：

### 1. AM调制原理

- **AM已调信号的时域表达式：**
 ![ ](https://user-images.githubusercontent.com/29295862/35344947-5391bf16-0169-11e8-97c9-d8eedace8214.png)

- **已调信号的频域表达式：**
 ![ ](https://user-images.githubusercontent.com/29295862/35344965-5af1843a-0169-11e8-8404-a27227df6875.png)
本质上AM调制就是频谱的搬移。

- **AM调制的系统框图**
 ![ ](https://user-images.githubusercontent.com/29295862/35344973-5f419bc4-0169-11e8-9397-0813d525fd8b.png)
将调制信号加上一个直流分量，保证信号的最小值大于零，然后再和载波相乘，得到已调信号。

## 三、AM调制的FPGA实现

### 1.产生调制信号和载波信号

调用ROM IP核在FPGA内部产生两路余弦信号，其中一路信号用于模拟外部输入的调制信号，另一路用作载波信号。
在配置ROM IP核之前，需要用Matlab生.coe文件，存放在ROM核里。

- **Matlab生成.coe文件：**

```
%---------------------------------%
width=8;       %设置rom的位宽
depth=1024;     %设置rom的深度
%---------------------------------%

x=linspace(0,2*pi,depth);       %在一个周期内产生depth个采样点
y_cos=cos(x);                   %生成余弦函数
%y_cos=sin(x);                   %生成正弦函数
%y_cos=round(y_cos*(2^(width-1)-1))+2^(width-1)-1;       %将数据转化成整数,生成无符号数
y_cos=round(y_cos*(2^(width-1)-1));       %将数据转化成整数,生成有符号数

plot(x,y_cos);                  %绘图

fid = fopen('E:\Workspace\DDS\Design\IP_Core\cos.coe','wt');

fprintf(fid,'memory_initialization_radix = 10;\nmemory_initialization_vector = ');
for i = 1 : depth
    if mod(i-1,8) == 0 
        fprintf(fid,'\n');
    end
    fprintf(fid,'%6d,',y_cos(i));
end

fclose(fid);                    %关闭文件
```

生成.coe文件后就可以进行IP核的配置了。
 
- **ROM核具体配置：**
 ![ ](https://user-images.githubusercontent.com/29295862/35349621-6e1186ac-0176-11e8-98e5-a1475ff60d50.png)
 ![ ](https://user-images.githubusercontent.com/29295862/35349629-739fdf24-0176-11e8-800e-aea80c15a60d.png)
 ![ ](https://user-images.githubusercontent.com/29295862/35349637-77da1834-0176-11e8-8b3a-e1ce6b8441a5.png)
 ![ ](https://user-images.githubusercontent.com/29295862/35349644-7c3bb8ec-0176-11e8-9acc-f9033ad57396.png)

配置完IP核后，编写控制模块，产生两路信号。其中，调制信号上叠加的直流分量的大小为调制信号的峰值，这样将得到调制度为100%的已调信号。如果要得到不同的调制度，则需要叠加不同大小的直流分量，同时需要注意定义的数据位宽，防止数据溢出。

- **产生载波和带有直流分量的调制信号：**

```
module cos_make(
	input	clk,
	input	rst_n,
	output	reg	[7:0]	cos_s,
	output	reg	signed	[7:0]	cos_c
);

//------------------------------------//
parameter freq_s = 32'd429497;			//调制信号频率10k
parameter freq_c = 32'd42949673;		//载波频率1M
parameter cnt_width = 8'd32;
//------------------------------------//

//------------------------------------//
reg 	[cnt_width-1:0]	cnt_s = 0;
reg		[cnt_width-1:0]	cnt_c = 0;
wire 	[9:0]	addr_s;
wire	[9:0]	addr_c;
always @(posedge clk or negedge rst_n) begin
	if(!rst_n)	begin
		cnt_s <= 0;
		cnt_c <= 0;
	end
	else	begin
		cnt_s <= cnt_s + freq_s;
		cnt_c <= cnt_c + freq_c;
	end
end

assign	addr_s = cnt_s[cnt_width-1:cnt_width-10];
assign	addr_c = cnt_c[cnt_width-1:cnt_width-10];
//------------------------------------//

//------------调用ROM核----------------//
wire 	signed	[7:0]	cos_s_r;
wire 	signed	[7:0]	cos_c_r;

ROM			ROM_inst(
	.clka	(clk),
	.addra	(addr_s),
	.douta	(cos_s_r),
	.clkb	(clk),
	.addrb	(addr_c),
	.doutb	(cos_c_r)
);

always @(posedge clk or negedge rst_n) begin
	if(!rst_n)	begin
		cos_s <= 0;
		cos_c <= 0;
	end
	else	begin
		cos_s <= cos_s_r + 8 'd128;		//加上大小为峰值的直流分量
		cos_c <= cos_c_r;
	end
end

endmodule

```

### 2.生成AM调制信号

得到两路信号后就可以用乘法器将两路信号相乘，得到已调信号。

- **乘法器具体配置：**
 ![ ](https://user-images.githubusercontent.com/29295862/35351304-722b11ea-017b-11e8-8fbe-fa55d56b9e0c.png)
 ![ ](https://user-images.githubusercontent.com/29295862/35351309-76a5fee2-017b-11e8-876f-0c74fc15beab.png)

- **AM调制的顶层模块：**

```
module modulate(
	input		clk,
	input		rst_n,
	output	signed	[15:0]	AM_mod
);

wire 	[7:0]	cos_s;
wire	signed	[7:0]	cos_c;

//------------调用出波模块------------//
cos_make		cos_make_inst0(
	.clk			(clk),
	.rst_n		(rst_n),
	.cos_s		(cos_s),
	.cos_c		(cos_c)
);
//-----------------------------------//

//------------调用乘法器--------------//
MULT		MULT_inst1(		
  .CLK	(clk),
  .A		(cos_s),
  .B		(cos_c),
  .P		(AM_mod)
);

endmodule

```

### 3.仿真调制结果

以上AM调制过程基本完成，但是正确与否还需要通过仿真来确定，接下来编写仿真用的测试模块。

- **TestBeach的编写：**

```
`timescale 1ns/1ps

module tb_AM();

//---------接口设置----------//
reg 	sclk;
reg		rst_n;
wire 	signed	[15:0]	AM_mod;
//--------------------------//
initial		sclk = 1;
always	#5	sclk = ~sclk;		//100M时钟

initial	begin
	rst_n = 0;
	#500
	rst_n = 1;
end
//--------------------------//
modulate		modulate_inst0(
	.clk		(sclk),
	.rst_n		(rst_n),
	.AM_mod		(AM_mod)
);

endmodule

```

在Vivado中将各个文件添加进工程后，运行仿真。

- **仿真结果如下：**
 ![ ](https://user-images.githubusercontent.com/29295862/35352112-d7de9514-017d-11e8-8bfc-bb974be7840c.png)
 ![ ](https://user-images.githubusercontent.com/29295862/35352115-da652410-017d-11e8-8d84-881a1c2af51f.png)

已调信号能明显看到包络，并且包络的频率同调制信号一致，表明AM调制正确。
