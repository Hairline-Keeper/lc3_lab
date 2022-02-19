环境搭建
=========

实验目的
--------

- 在电脑上安装今后实验所需要的工具
- 初步了解一个简单电路的实现和仿真流程
- 为后续的实验打下基础

实验设备
--------
- Ubuntu操作系统的电脑一台，或装有Ubuntu操作系统的虚拟机

实验任务
--------

- 在系统中安装jdk-13
- 在系统中安装mill-0.9.8
- 在系统中安装verilator-3.916
- 使用verilog编写一个简单的译码器，并通过verilator编译并仿真运行

.. hint:: 
    1.	由于安装和下载程序可能会需要比较久的时间，强烈建议在课前预习时完成安装
    2.	如果没有课前预习，可以导入打包好的VirtualBox镜像，里面已经将需要用到的环境配置好了，但是仍然希望同学在课后自己尝试安装一遍
    3.	在实验指导中给出了在设计本课程时使用到的程序版本，今后如果有大的更新影响到我们的代码或者程序行为的正确性时，请及时更新实验指导


实验内容
--------

安装JDK
*******
安装jdk的原因是mill使用时会用到jdk，因为mill不仅能够编译Scala，也能够用来构建java的工程。在Ubuntu中可以直接通过以下命令安装

.. code-block:: shell

    sudo apt install -y openjdk-13-jdk

这是最直接最简单的方法，但是在真实安装过程中，可能会由于软件源的问题，导致下载的速度非常的慢。
可以通过换源，或者自己直接通过网站下载好之后解压到指定的目录安装，再配置好环境变量后使用，具体的操作可以在网上搜索。
总之，想办法把jdk安装好，在执行 ``java --version`` 后会输出版本信息，就代表安装完成了。

.. code-block:: shell

    > java --version
    openjdk 11.0.13 2021-10-19
    OpenJDK Runtime Environment GraalVM CE 21.3.0 (build 11.0.13+7-jvmci-21.3-b05)
    OpenJDK 64-Bit Server VM GraalVM CE 21.3.0 (build 11.0.13+7-jvmci-21.3-b05, mixed mode, sharing)

安装mill
********

mill是LC3项目使用的build tool，用来将我们写的Chisel代码编译成verilog，在之后的实验中会使用到。
安装mill首先需要安装curl程序，curl是一个利用URL规则在命令行下工作的文件传输工具。
安装完curl，利用该工具安装mill并加执行权限。

.. code-block:: shell

    sudo apt install -y curl
    mill sudo bash -c "curl -L https://github.com/com-lihaoyi/mill/releases/download/0.9.8/0.9.8 > /usr/local/bin/mill && chmod +x /usr/local/bin/mill"

之后运行 ``mill version`` 命令，第一次运行它会下载一些必要的文件，可能需要等待一段时间，在运行完成后你应该看到你安装的mill的版本号，这代表mill安装完成。

.. figure:: _static/mill.png
    :alt: mill
    :align: center

    fig1-1: mill version


安装verilator
*************


verilator和curl一样，直接安装后查看版本。

.. code-block:: shell

    sudo apt install -y verilator
    verilator --version

.. figure:: _static/verilator.png
    :alt: verilator
    :align: center

    fig1-2: verilator version
   

简单电路设计
************
接下来我们可能会需要写一些代码，在每次迭代中，我们首先需要编写或修改Chisel的代码，然后使用mill将其编译生成对应的verilog代码，然后再用verilator将其编译转换成用来仿真的C++代码，最后再执行仿真。
当然这个过程比较复杂，一般都是用make或cmake之类的命令来控制的，这个具体的流程可以留到下一次实验来做，这节课我们可以先写点简单的：用verilog编写一个3-8译码器，然后用verilator将它运行起来

**编写一个3-8译码器**
3-8译码器是一个纯组合逻辑，输入0-7的数字n，返回一个8bits的信号，其中第n位是1，其它位都是0，下表是3-8译码器的真值表


+-----+--------+--------+--------+---------+---------+---------+---------+---------+---------+---------+---------+
| in  | in[2]  | in[1]  | in[2]  | out[7]  | out[6]  | out[5]  | out[4]  | out[3]  | out[2]  | out[3]  | out[2]  |
+=====+========+========+========+=========+=========+=========+=========+=========+=========+=========+=========+
| 0   | 0      | 0      | 0      | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       |
+-----+--------+--------+--------+---------+---------+---------+---------+---------+---------+---------+---------+
| 1   | 0      | 0      | 1      | 0       | 0       | 0       | 0       | 0       | 0       | 0       | 0       |
+-----+--------+--------+--------+---------+---------+---------+---------+---------+---------+---------+---------+
| 2   | 0      | 1      | 0      | 0       | 0       | 0       | 0       | 0       | 1       | 0       | 1       |
+-----+--------+--------+--------+---------+---------+---------+---------+---------+---------+---------+---------+
| 3   | 0      | 1      | 1      | 0       | 0       | 0       | 0       | 1       | 0       | 1       | 0       |
+-----+--------+--------+--------+---------+---------+---------+---------+---------+---------+---------+---------+
| 4   | 1      | 0      | 0      | 0       | 0       | 0       | 1       | 0       | 0       | 0       | 0       |
+-----+--------+--------+--------+---------+---------+---------+---------+---------+---------+---------+---------+
| 5   | 1      | 0      | 1      | 0       | 0       | 1       | 0       | 0       | 0       | 0       | 0       |
+-----+--------+--------+--------+---------+---------+---------+---------+---------+---------+---------+---------+
| 6   | 1      | 1      | 0      | 0       | 1       | 0       | 0       | 0       | 0       | 0       | 0       |
+-----+--------+--------+--------+---------+---------+---------+---------+---------+---------+---------+---------+
| 7   | 1      | 1      | 1      | 1       | 0       | 0       | 0       | 0       | 0       | 0       | 0       |
+-----+--------+--------+--------+---------+---------+---------+---------+---------+---------+---------+---------+

首先我们找个合适的地方，新建一个目录，取名为decoder，然后进入到这个目录下。接下来按照这个真值表，用verilog编写相应的模块，参考代码如下，建议大家最好能够做到不看参考代码自己实现。

.. code-block:: verilog

    // decoder.v
    module decoder(
    input [2:0] in,
    output [7:0] out
    );

    reg [7:0] out_reg;;

    always @(*) begin
    case(in)
    			3'd0: out_reg = 8'b00000001;
    			3'd1: out_reg = 8'b00000010;
    			3'd2: out_reg = 8'b00000100;
    			3'd3: out_reg = 8'b00001000;
    			3'd4: out_reg = 8'b00010000;
    			3'd5: out_reg = 8'b00100000;
    			3'd6: out_reg = 8'b01000000;
    			3'd7: out_reg = 8'b10000000;
    			default: out_reg = 8'b0;
    		endcase
    	end

    	assign out = out_reg;

    endmodule

编写用于控制verilator仿真的main函数。
verilator在仿真前需要自己编写一个main函数，来控制整个仿真过程的一些行为，以下是参考代码

.. code-block:: c++

    // sim_main.cpp
    #include "Vdecoder.h" // 这个头文件会根据你模块的名字不同而改变
    #include <verilated.h>
    #include <iostream>
    #include <bitset> // 用于输出二进制的数据

    using namespace std;

    int main(int argc, char **argv, char **env){
        Verilated::commandArgs(argc, argv);
        Vdecoder* decoder = new Vdecoder;  // 模块的实例

        int code = 0; // 用于decoder模块的输入，从0-7遍历

        while (!Verilated::gotFinish() && code < 8) {
            decoder->in = code;

            decoder->eval(); // 每执行一次eval函数，就对decoder模块执行一次仿真
            cout<<"in: "<<code<<"\t";
            cout<<"out: "<<bitset<8>(decoder->out)<<endl; // 输出deocder模块的out接口的信号

            code++;
        }

        decoder->final();
        delete decoder;
        exit(0);
    }


现在你的decode目录下应该有两个文件了，分别是decoder.v和sim_main.cpp，接下来我们使用verilator将verilog代码转换成仿真需要用到的文件，包括.h和.cpp文件和其他的一些文件，
在decode目录下运行如下命令:

.. code-block:: shell

    verilator -Wall --cc decoder.v --exe sim_main.cpp

在你解决完所有的错误后，你应该能够看到在当前目录下生成了一个新的目录obj_dir，进到这个目录后能够看到这些文件。

.. figure:: _static/decoder.png
    :alt: decoder
    :align: center

    fig1-3: 生成的文件


接下来回到decoder目录，运行

.. code-block:: shell

    make -C ./obj_dir -f ./Vdecoder.mk
    
就会看到在obj_dir目录下新生成了一个名为Vdecoder的可执行文件，运行这个文件，你就能够看到decoder这个模块对应的输入和输出，确保它的功能正常。
如果运行时出现了错误，请耐心的按照错误信息定位检查代码中的问题


.. figure:: _static/decoder_out.png
    :alt: decoder_out
    :align: center

    fig1-4: decoder仿真输出

实验总结
--------

通过本节课，希望大家能够掌握：

- Chisel开发环境的搭建
- 使用verilator对verilog代码进行仿真的流程，以及相应代码的作用
