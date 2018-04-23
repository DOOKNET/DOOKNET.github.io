---
title: AM解调的FPGA实现
date: 2018-02-09 19:13:38
categories:
- FPGA
tags:
- FPGA
- AM
- 调制解调
comments: true
---

## 一、说明：

1. 功能：AM解调
2. 平台：Vivado 2016.4 和 Matlab R2017a

## 二、原理：

### 1.AM解调原理

- **模拟电路中采用“包络检波”的方法：**
 ![这里写图片描述](https://user-images.githubusercontent.com/29295862/35344975-615af3f6-0169-11e8-9d69-7cac3196baad.png)

- **数字电路中采用类似的方法：**
 先将已调信号取绝对值，再经过低通滤波器，滤除高频分量（经AM调制的信号包含两个高频分量：载波频率+/-调制信号频率，因此低通滤波器的截止频率小于两个高频分量就可以），得到的就是叠加了直流分量的调制信号，去直流后便可以得到调制信号。

## 三、AM解调的FPGA实现

### 1.将已调制的AM信号取绝对值

关于AM信号的产生，参见上一篇博客：[AM调制的FPGA实现](http://blog.csdn.net/hooknet/article/details/79129451)

简单说明一下对数据取反的思路：如果是无符号数，则不存在符号位，也就是说数据都是正数，不需要取绝对值；如果是有符号数，通过检测最高位的符号位，如果符号位是1，则表示数据是负数，对数据取反，如果符号位是0，则表示数据是正数，不需要取反操作。

- **取绝对值的Verilog实现：**

```
always @(posedge clk or negedge rst_n) begin
	if(!rst_n)	begin
		data_tdata <= 0;
	end
	else if(AM_mod[15] == 1)	begin
		data_tdata <= -{AM_mod};		//如果符号位是1，对数据取反
	end
	else if(AM_mod[15] == 0)	begin
		data_tdata <= AM_mod;			//如果符号位是0，数据不变
	end
	else	begin
		data_tdata <= data_tdata;
	end
end
```

### 2.使用FIR滤波器滤除高频分量

关于Vivado的FIR IP核可以说是功能很强大的，但这里不需要其他复杂的功能，只需要简单的生成一个的低通滤波器就行了。	
类似于ROM核的生成，配置FIR同样需要Matlab配合。可见，Matlab的功能是多么强大。这里Matlab的主要作用是对滤波器的性能进行仿真并生成相应的抽头系数。

- **使用Matlab生成FIR的抽头系数**

  在Matlab的命令行窗口输入：**filterDesigner**（以前是用fdatool命令，不过输入fdatool也可以，只是会提醒你改用新的命令）弹出滤波器设计窗口：
![ ](https://user-images.githubusercontent.com/29295862/35436554-d89264fc-02c9-11e8-86a7-1a3ec42b38a4.png)

  接下来，对滤波器的一些参数进行设置：
![ ](https://user-images.githubusercontent.com/29295862/35436558-dfcb81c2-02c9-11e8-9645-53b557b1e4dc.png)

  参数设置好后，点击**Design Filter** 按钮查看生成滤波器的幅频响应图，通过幅频响应等图来判断滤波器是否达到设计要求：
![ ](https://user-images.githubusercontent.com/29295862/35436563-e5e15244-02c9-11e8-9182-5dfec6896191.png)

  设计的滤波器满足性能指标后需要将抽头系数导出，保存为.coe文件。在导出前需要对系数进行量化。因为需要解调的AM信号也是16位宽，所以这里的位宽设置保持默认值，这些可以根据实际情况自行修改。
![ ](https://user-images.githubusercontent.com/29295862/35436570-eb25ca28-02c9-11e8-8883-bb954ca2632c.png)

  量化过后就能将抽头系数导出为.coe文件了：
![ ](https://user-images.githubusercontent.com/29295862/35436572-ed8ba742-02c9-11e8-950d-d837242dc0c8.png)

- **生成FIR IP核**

  IP核的具体配置如下：
![ ](https://user-images.githubusercontent.com/29295862/35437697-9211cfe0-02ce-11e8-8952-72d5271b127f.png)
![ ](https://user-images.githubusercontent.com/29295862/35437699-927806de-02ce-11e8-97fa-89c4f899561c.png)
![ ](https://user-images.githubusercontent.com/29295862/35437700-92dc2ed4-02ce-11e8-8ba7-9b9bb82c8b15.png)

  其他保持默认即可：
![ ](https://user-images.githubusercontent.com/29295862/35437701-933df1dc-02ce-11e8-854a-abd7a0fb4038.png)
![ ](https://user-images.githubusercontent.com/29295862/35437702-93c467f8-02ce-11e8-8c03-142055f69c5e.png)

  同样，在IP核配置界面也可以查看滤波器的幅频特性：
![ ](https://user-images.githubusercontent.com/29295862/35437703-942511ac-02ce-11e8-94e0-809114c09697.png)

IP核生成完毕后，就可以编写IP核的调用模块了。

- **FIR IP核调用模块：**

```
module FIR_Control(
	input	clk,
	input	rst_n,
	input	signed	[15:0]	s_axis_data_tdata,
	output	reg 	[7:0]	data_out
);

wire 	s_axis_data_tready;
wire	m_axis_data_tvalid;
wire 	[39:0]	m_axis_data_tdata;		//滤波器输出信号

always @(posedge clk or negedge rst_n) begin
	if(!rst_n)	begin
		data_out <= 0;
	end
	else	begin
		data_out <= m_axis_data_tdata[33:26];	//根据仿真结果进行截位
	end
end

//--------------调用FIR核----------------//
FIR						FIR_inst0(
  .aclk					(clk),
  .s_axis_data_tvalid	(1),					//拉高时IP核开始工作
  .s_axis_data_tready	(s_axis_data_tready),	
  .s_axis_data_tdata	(s_axis_data_tdata),	//输入信号
  .m_axis_data_tvalid	(m_axis_data_tvalid),	//拉高时表明数据输出有效
  .m_axis_data_tdata	(m_axis_data_tdata)		//输出信号
);
//---------------------------------------//


endmodule 
```

需要注意的是：	
**m_axis_data_tdata** 信号是滤波器的数据输出信号，我们在使用时一般都要对此数据进行截位操作，如何进行截位需要根据仿真结果来确定。比如，在这个工程中，我需要的滤波器的输出数据是8位，但不能一下子截取高8位，而且**m_axis_data_tdata**是个40位的数据，从仿真波形来看**m_axis_data_tdata**[39:34]都是符号位，因此从33位开始往下截取8位数据（当然也可以从34位开始截，这样的话就多了一位符号位，相应的数据位就变少了一位）。

### 3.去直流处理

经过FIR滤波后的波形其实就是一个叠加了直流分量的调制信号。在本工程中，AM调制是100%调制，也就是说解调时经过FIR后的信号的最小值为0，可以把它看作是无符号的数，直接经DA输出就行了。
如果不是100%调制呢？也就是说解调时经过FIR后的信号的最小值是大于0的，那么这个大于0的量就相当于直流，需要去掉后再经DA输出。
因此，在这个工程中，不需要去直流处理。下面给出顶层文件的代码。

- **顶层模块编写：**

```
module TOP(
	input	clk,
	input	rst_n,
	output	[7:0]	AM_demod
);

//--------------------------------//
reg 	signed	[15:0]	data_tdata;
wire 	signed	[15:0]	AM_mod;
//--------------------------------//

//-----------取绝对值-------------//
always @(posedge clk or negedge rst_n) begin
	if(!rst_n)	begin
		data_tdata <= 0;
	end
	else if(AM_mod[15] == 1)	begin
		data_tdata <= -{AM_mod};		//如果符号位是1，对数据取反
	end
	else if(AM_mod[15] == 0)	begin
		data_tdata <= AM_mod;			//如果符号位是0，数据不变
	end
	else	begin
		data_tdata <= data_tdata;
	end
end
//--------------------------------//

//-----------AM已调信号------------//
modulate		modulate_inst0(
	.clk		(clk),
	.rst_n		(rst_n),
	.AM_mod		(AM_mod)
);
//--------------------------------//

//----------滤波器控制模块---------//
FIR_Control				FIR_Control_inst2(
	.clk				(clk),
	.rst_n				(rst_n),
	.s_axis_data_tdata	(data_tdata),
	.data_out			(AM_demod)
);
//--------------------------------//

endmodule

```

### 4.解调仿真

- **编写TestBeach：**

```
`timescale 1ns/1ps

module tb_AM();

//===================解调部分====================//
//----------接口设置----------//
reg 	sclk;
reg		rst_n;
wire 	[7:0]	AM_demod;
//--------------------------//
initial		sclk = 1;
always	#5	sclk = ~sclk;		//100M时钟

initial	begin
	rst_n = 0;
	#500
	rst_n = 1;
end
//----------解调模块----------//
TOP				TOP_inst(
	.clk		(sclk),
	.rst_n		(rst_n),
	.AM_demod	(AM_demod)
);
//---------------------------//

endmodule

```

- **仿真结果**

  ![ ](https://user-images.githubusercontent.com/29295862/35448887-7a2c7452-02f6-11e8-91ce-0253953ac892.png)

由仿真结果可知，最终输出信号正确还原了已调制信号的包络，表明解调正确。