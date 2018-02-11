---
title: Matlab产生测试激励
date: 2018-02-11 14:44:06
categories:
- Matlab
tags:
- Matlab
- Testbench
---

### 前言

在FPGA开发过程中几乎都要用到仿真的功能，对于一些简单的外部激励（如时钟、复位、简单数据或者信号等）直接在testbench中编写产生就行了，但对于复杂的外部激励数据，很难在testbench中产生，这时就要通过读取外部文件里的数据来实现。通过和matlab的配合使用，基本上可以模拟各种外部激励。
举例来说：输入信号是三个不同频率的正弦波的相加，经过FIR低通滤波器滤除高频分量，输出频率最低的那个正弦信号。这种情况下测试用的输入信号不能通过testbench编写产生。
简单来说有以下两种方法可以模拟输入信号：

1. 在FPGA内部通过DDS产生三个正弦波，然后将三个波形相加作为输入信号。
2. 利用matlab产生输入信号，将数据导出为.txt文件，在仿真时读取文件内的数据作为外部激励。

显然第二种方法更加灵活和便捷。下面，具体介绍一下这种方法的使用。

-------------------------------

### 平台：

- Vivado 16.4
- Matlab R2017b

### Matlab程序编写：

- 代码如下：

```
%=============设置系统参数==============%
f1=1e6;        %设置波形频率
f2=500e3;
f3=800e3;
Fs=20e6;        %设置采样频率
L=1024;         %数据长度
N=14;           %数据位宽
%=============产生输入信号==============%
t=0:1/Fs:(1/Fs)*(L-1);
y1=sin(2*pi*f1*t);
y2=sin(2*pi*f2*t);
y3=sin(2*pi*f3*t);
y4=y1+y2+y3;
y_n=round(y4*(2^(N-3)-1));      %N比特量化;如果有n个信号相加，则设置（N-n）
%=================画图==================%
a=10;           %改变系数可以调整显示周期
stem(t,y_n);
axis([0 L/Fs/a -2^N 2^N]);      %显示
%=============写入外部文件==============%
fid=fopen('E:\Workspace\Vivado_16.4\TEST\TestBench\sin_data.txt','w');    %把数据写入sin_data.txt文件中，如果没有就创建该文件
for k=1:length(y_n)
    B_s=dec2bin(y_n(k)+((y_n(k))<0)*2^N,N);
    for j=1:N
        if B_s(j)=='1'
            tb=1;
        else
            tb=0;
        end
        fprintf(fid,'%d',tb);
    end
    fprintf(fid,'\r\n');
end

fprintf(fid,';');
fclose(fid);
```

此程序中设置了三个频率分别为1M、500k和800k的正弦波，然后将三个波形相加并且量化后作为输出。最后将路径设置为相应文件所在路径即可，需要注意的是如果对应路径下没有相应文件，则会自动新建文件并写入数据。

- 运行程序：
 ![ ](https://user-images.githubusercontent.com/29295862/35777703-4d3d1e6c-09ed-11e8-9c6f-3b6f9abd597b.png)

- 打开对应的文件目录
 ![ ](https://user-images.githubusercontent.com/29295862/35777708-51403d82-09ed-11e8-9cbe-41e84d961120.png)
可以看到二进制数据文件已经生成。

接下来就可以进行在testbench中读取外部数据的操作了。

### Testbench的编写：

- 代码如下：

```
`timescale 1ns/1ps

module TB_readfile();
reg     SCLK;
reg    [13:0]  data_out;

//--------------时钟部分----------------//
initial     SCLK = 0;
always      #10     SCLK = ~SCLK;

//-------------------------------------//
parameter data_num = 32'd1024;
integer   i = 0;
reg [13:0]  data_men[1:data_num];
reg [13:0]  data_reg = 0;
initial begin
    $readmemb("E:/Workspace/Vivado_16.4/2017_8_28_TEST/TestBench/sin_data.txt",data_men);   //注意斜杠的方向，不能反<<<<<<<
end
always @(posedge SCLK) begin
    data_out <= data_men[i];
    i <= i + 8'd1;
end
//------------------------------------//

endmodule

```

因为这里只需要读取外部数据，所以Vivado工程里只需要添加仿真文件就行了。

- 运行仿真：
 ![ ](https://user-images.githubusercontent.com/29295862/35778030-52933256-09f3-11e8-87fc-56e030b5d9ba.png)
可以看到，仿真的波形和matlab中显示的波形一致，说明结果正确。

---

### 结束语

Matlab可以说是个很强大的工具，在设计中合理的使用matlab可以起到事半功倍的效果。当然我也在不断的学习中，如果有疑问或者更好的想法欢迎交流。。。
