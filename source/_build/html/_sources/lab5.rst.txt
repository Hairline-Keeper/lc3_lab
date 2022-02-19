LC3的启动与仿真
=======================

实验目的
--------

- 了解LC3的启动原理与仿真过程

实验设备    
--------
- Ubuntu操作系统的电脑一台，或装有Ubuntu操作系统的虚拟机

实验任务
--------

- 使用chisel实现LC3的启动Boot
- 仿真运行LC3并使用gtkwave查看波形

实验内容
--------

Boot状态和控制设计
********************

**功能**：

- 接收Uart传输的LC3程序数据，每周期接收8bits，至少需要2周期接收完一个16bits指令或数据。
  Uart接收到数据时uartValid会置高，前8bits会寄存一拍，第二拍接收后8bits会与前面拼接成16bits并且置wordValid为高。
- 控制LC3的状态并根据状态控制Memory的工作

**流程**：LC3在的工作过程主要分为三步，在代码设计中分为三个状态：
- initpc: 初始化PC，在LC3汇编中的第一个数据即为PC，需要将该数据存入到一个寄存器中,该数据来源于Uart。
- initmem: 初始化内存，程序内容需要逐个放入Memory中。
- work:执行程序。

    Lab5-task1(src/main/scala/LC3/Boot.scala):编写状态转换以及控制信息。

运行仿真
*********

在image目录下存在默认程序dummy.obj，默认程序会向uart输出Hello!并打印到屏幕上。运行make emu TRACE=-t会执行编译和运行dummy.obj并打印波形文件emu.vcd。
加IMAGE=$your_file选项可以执行指定的程序。
最终波形会生成在build/文件夹下的emu.vcd。
用波形工具gtkwave打开emu.vcd ``gtkwave build/emu.vcd``

在gtk界面中选择查看的信号添加到波形面板

.. figure:: _static/panel.png
    :alt: controller
    :align: center

    fig5-1: 波形面板

可以看见lc3state的状态转换经历3个过程

.. figure:: _static/lc3state.png
    :alt: controller
    :align: center

    fig5-2: lc3state变化
 
观察lc3state状态为work的情况下控制器的state，与状态机转换一致，其中状态18则为每条指令的第一个状态。
 
.. figure:: _static/mstate.png
    :alt: controller
    :align: center

    fig5-3: 状态机

注意，波形数值默认不是十进制数，方便观察建议转换到对应进制。

.. figure:: _static/decimal.png
    :alt: controller
    :align: center

    fig5-3: 十进制显示
|

    Lab5-task2:在波形中观察ALU的4个功能在波形上的表现是正确的。

.. tip::
    1. 找出哪些指令用到ALU的功能。
    2. 查看alu模块端口信号看是否满足预期。

