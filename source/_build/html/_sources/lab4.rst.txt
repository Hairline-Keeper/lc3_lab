LC3数据通路的设计与验证
=======================

实验目的
--------

- 熟悉LC3微结构，具体请仔细阅读 **《计算机系统概论》 附录C**
- 了解LC3数据通路原理以及ALU和Regfile的设计与验证。

实验设备    
--------
- Ubuntu操作系统的电脑一台，或装有Ubuntu操作系统的虚拟机

实验任务
--------

- 使用chisel实现LC3的ALU和Regfile
- 使用chisel完善LC3的数据通路
- 验证ALU与Regfile

实验内容
--------

ALU设计
****************

**端口**：LC3 ALU模块中需要接收三个输入，即操作数1，操作数2，和操作类型，并且得到一个输出。

============== ======= =========
端口名称        方向    位宽   
============== ======= =========
操作数1         输入   16 bits  
操作数2         输入   16 bits
操作类型        输入   2 bits
输出数据        输出   16 bits 
============== ======= =========

**逻辑功能**：LC3 ALU中支持的运算操作为四类，加法，按位或，按位反以及取操作数1。
在ALU运算时会根据上述的输入类型进行选择做何种运算（2bits的操作类型可表示四种操作），其中操作类型的2bits来自于数据通路的SIG_ALUK。

============== ==============
操作类型        功能 
============== ==============
SIG_ALUK = 0    加法
SIG_ALUK = 1    按位与
SIG_ALUK = 2    对操作数1取反
SIG_ALUK = 3    取操作数1
============== ==============


    **Task1** (src/main/scala/LC3/ALU.scala) 编写ALU逻辑。

ALU验证
****************
完成设计之后，我们需要对ALU的功能进行验证。除了在第三章中使用简单的结果观察方法(容易出错)，在数字电路中的验证方法通常采用一个正确的模型(golden modle)去和我们的设计进行对比。
给正确模型和被测试的电路给同样的输入，判断输出是否正确。

.. figure:: _static/golden.png
    :alt: controller
    :align: center

    fig4-1: 对比验证

在chisel设计中最容易获得的模型便是使用scala编写的。

.. code-block:: scala

    // src/test/scala/ALUTests.scala

    def TEST_SIZE = 10  // 定义测试数量

    val ina, inb, add_out, and_out, not_out, pass_out = Array.fill(TEST_SIZE)(0)  // 定义正确模型的输入和输出

    for (i <- 0 until TEST_SIZE) {
        ina(i) = Random.nextInt(0xffff)   // 输入采用随机数生成，因此也叫随机测试
        inb(i) = Random.nextInt(0xffff)   
        add_out(i) = ina(i) + inb(i)      // 以下4行是正确模型计算的结果。
        and_out(i) = ina(i) & inb(i)    
        not_out(i) = 0xffff-ina(i)
        pass_out(i) = ina(i)
    }

    it should "test add" in {             // 此处为add功能的测试用例，其他测试用例见源码
        test(new ALU) { c =>
            for(i <- 0 until TEST_SIZE) { 
                c.io.ina.poke(ina(i).U)                 // 对ina端口输入数据
                c.io.inb.poke(inb(i).U)                 // 对inb端口输入数据
                c.io.out.expect(add_out(i).U(15,0))     // 检测out端口输出，并比对正确模型结果
            }
        }
    }

输入如下命令对ALU进行验证。

.. code-block:: shell

    mill chisel_lc3.test.testOnly LC3.ALUTest

若验证正确，获得以下结果

.. figure:: _static/alutest.png
    :alt: dirtree
    :align: center

    fig4-1: ALUTest pass


Regfile设计和验证
******************
**端口**：对一个寄存器堆需要做的操作主要有读和写。在LC3的指令中最多需要读取两个数据，写入一个数据。
因此需要两组读端口一组写端口，每组端口需要一个地址和数据。此外还需要一个信号来判断是否为写信号(在datapath中该信号为LD_REG)，只有该信号为真才可以改变寄存器里的数据。


============== ======= =========
端口名称        方向    位宽   
============== ======= =========
读地址1         输入    3 bits  
读数据1         输入    3 bits
读地址2         输入    16 bits
读数据2         输出    16 bits 
写地址          输入    3 bits 
写数据          输入    16 bits
写使能          输入    1 bits
============== ======= =========

**逻辑功能**：根据 LC3 的设计，寄存器堆深度为8，宽度为16。读操作时，改变写地址的数据。读数据时，读数据端口为读地址的寄存器数据。

    **Task2** (src/main/scala/LC3/Regfile.scala) 编写Regfile逻辑。
    
    **Task3** (src/test/scala/RegfileTest.scala) 编写Regfile进行读写测试，即对某个寄存器进行写操作，再读该寄存器进行读操作，对比写入和读出数据是否一样。

完成设计和验证后输入如下命令对Regfile进行验证。

.. code-block:: shell

    mill chisel_lc3.test.testOnly LC3.RegfileTest

完善数据通路
**************
**Task4,Task5,Task6** (src/main/scala/LC3/Datapath.scala)根据控制信号，分别编写由LD信号控制的寄存器值改变逻辑，Gate信号控制数据源的选择逻辑，和Mux信号控制的数据选择逻辑。