组合电路设计
===========

实验目的
--------

- 了解基本的Chisel和verilog语法要的工具
- 掌握完整的从Chisel到仿真的开发流程现和仿真流程
- 学会编写Makefile将编译过程自动化，实现一键仿真

实验设备
--------
- Ubuntu操作系统的电脑一台，或装有Ubuntu操作系统的虚拟机

实验任务
--------

- 使用Chisel重新实现我们在实验一中用verilog编写的3-8译码器
- 编写build.sc文件，指定Scala和Chisel的版本
- 编写Makefile文件，指定编译仿真的规则

实验内容
--------

使用Chisel实现3-8译码器
***********************
在上一节实验课中我们用verilog实现了一个3-8译码器，现在我们要用Chisel重新实现它。
我们希望你能够在自己的decoder目录下再新建一个decoder/src目录（注意，整个实验指导中所说的decoder目录都是外面这个decder目录，而不是里面嵌套的这个目录，整个目录的结构在图x中可以看到），
这是由mill的项目格式规定的，我们将Chisel的代码放在这个目录下。由于这个译码电路实在是过于简单，因此你可能看不出Chisel相比于verilog有什么优势，
但是随着你设计的电路越来越复杂，你会发现Chisel能极大的提高你的效率。参考代码如下，希望你能读懂它

.. code-block:: scala

    ./decoder/src/main.scala
    package decoder
    import chisel3._
    import chisel3.util._
    import chisel3.stage._
    // RawModule与Module不同，它不会生成隐式的时钟，由于我们是组合电路，因此>现在还没有涉及到时钟的概念，只用RawModule即可
    class Decoder extends RawModule {
        val io = IO(new Bundle{
            val in = Input(UInt(3.W)) // 输入的0-7信号，二进制表示只需要3bits宽即可
            val out = Output(UInt(8.W)) // 输出的译码后信号，8bits宽
        })
        // 在Chisel中，可以对同一个变量多次赋值，以最后一次赋的值为最终结果
        // 因此如果下面的switch语句都没有执行，这一行能给io.out一个默认值
        io.out := "b00000000".U
        switch (io.in) {
            is (0.U) { io.out := "b00000001".U }
            is (1.U) { io.out := "b00000010".U }
            is (2.U) { io.out := "b00000100".U }
            is (3.U) { io.out := "b00001000".U }
            is (4.U) { io.out := "b00010000".U }
            is (5.U) { io.out := "b00100000".U }
            is (6.U) { io.out := "b01000000".U }
            is (7.U) { io.out := "b10000000".U }
        }
    }
    object testMain extends App {
        // 实例化Decoder模块
        (new ChiselStage).execute(args, Seq(
            ChiselGeneratorAnnotation(() => new Decoder))
        )
    }

编写build.sc文件
****************

我们在decoder目录下新建一个build.sc文件，写入如下代码

.. code-block:: scala

    import mill._, scalalib._
    import os.Path
    /**
     * Scala 2.12 module that is source-compatible with 2.11.
     * This is due to Chisel's use of structural types. See
     * https://github.com/freechipsproject/chisel3/issues/606
     */
    trait HasXsource211 extends ScalaModule {
      override def scalacOptions = T {
        super.scalacOptions() ++ Seq(
          "-deprecation",
          "-unchecked",
          "-Xsource:2.11"
        )
      }
    }
    
    object decoder extends ScalaModule with HasXsource211 {
        def scalaVersion = "2.12.10"
        override def millSourcePath = os.pwd
        def ivyDeps = Agg(
            ivy"edu.berkeley.cs::chisel3:3.5.0-RC1"
        )
    }

其中HasXsource211这个trait是为了避免一些兼容性问题，在这个实验中即使不加也不会有问题，但是在今后的实验中缺少这个可能会导致一些错误，因此建议还是加上
除去上面的trait，就只剩下deocder这一个单例对象了，其中的代码很简单，指定了使用2.12.10版本的Scala，指定了3.3.2版本的Chisel
在编写完build.sc文件之后，实际上你已经可以开始使用mill将Chisel转换成verilog文件了，运行如下命令

.. code-block:: shell

    mill decoder.run decoder.main.testMain

你会看到一些警告，可以不用理会，在运行结束后，如果没有错误的话，你会在decode目录下看到生成的Decoder.v文件，如下图所示

.. figure:: _static/decoder_run.png
    :alt: decoder_run
    :align: center

    fig2-1: 运行mill后看到生成的Decoder.v文件

你可以打开Decoer.v文件，看看它和你自己写的verilog有什么区别，你也可以尝试用verilator仿真运行它.


编写Makefile文件
****************

如果你在之前的实验中编写的代码出现了一些错误，导致你每次都要重复的输入这些命令，那么你应该已经开始厌烦了，
因此我们需要编写一个Makefile，通过make命令来自动管理这些代码和命令，这样我们在之后的开发过程中就能省下大量的精力，Makefile的内容如下

.. code-block:: shell

    # Makefile
    TOP = Decoder
    BUILD_DIR = ./build
    TOP_V = $(BUILD_DIR)/$(TOP).v
    SCALA_FILE = $(shell find ./decoder/src -name '*.scala')

    .DEFAULT_GOAL = verilog

    $(TOP_V): $(SCALA_FILE)
    	@mkdir -p $(@D)
    	mill decoder.run decoder.main.testMain -td $(@D) --output-file $(@F)

    verilog: $(TOP_V)

    SIM_TOP = Decoder

    EMU_MK := $(BUILD_DIR)/V$(SIM_TOP).mk
    EMU := $(BUILD_DIR)/emu
    CXX_FILE := ./sim_main.cpp

    $(EMU_MK): $(TOP_V) | $(EMU_DEPS)
    	@mkdir -p $(@D)
    	verilator -Wall --cc --exe \
    		-o $(abspath $(EMU)) -Mdir $(@D) $^ $(CXX_FILE)

    $(EMU): $(EMU_MK)
    	$(MAKE) -C $(dir $(EMU_MK)) -f $(abspath $(EMU_MK))

    emu: $(EMU)

    clean:
    	rm -rf build

简单的解释一下，在这个Makefile文件中我们定义了三个主要的target，分别是verilog，emu和clean，
中clean就是一条rm指令，把编译生成的build文件夹删掉，而verilog指令会将Chisel文件编译成verilog，
emu与verilog的区别在于它不光会将Chisel文件编译成verilog代码，还会将verilog代码转换成仿真使用的C++代码，
并将最终的可执行文件存放在build目录下。具体的Makefile语法可以上网搜索相关的手册
另外，记得把上一节课的sim_main.cpp复制到当前的decoder目录下
总之，如果之前的操作都正确的话，现在你可以直接运行``make emu``来一键生成仿真程序了，此时你完整的项目目录应该如下图所示：

.. figure:: _static/dirtree.png
    :alt: dirtree
    :align: center

    fig2-2: 完整的项目结构


实验总结
--------
通过本节课，希望大家能够掌握：
- 使用Chisel编写简单模块
- 编写build.sc来控制Scala和Chisel的版本，以及今后其他的一些依赖环境
- Makefile的语法，能够通过make指令使编译更简便，加快开发速度
