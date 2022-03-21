组合电路与时序电路设计
===========

本次设计的主要任务和解答如下: 

TASK1: 根据实验一用verilog实现的3-8译码器，使用Chisel重写，并能够通过仿真

TASK2: 实现有限状态机的序列检测模块，要求仿真结果与手册截图相符

TASK1
------

.. code-block:: scala

    package decoder

    import chisel3._
    import chisel3.util._
    // import chisel3.experimental._

    // RawModule与Module不同，它不会生成隐式的时钟，由于我们是组合电路，因此现在还没有涉及到时钟的概念，只用RawModule即可
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
        Driver.execute(args, () => new Decoder)
    }


TASK2
------

.. code-block:: scala

    package detection

    import chisel3._
    import chisel3.util._
    // import chisel3.experimental._

    // RawModule与Module不同，它不会生成隐式的时钟，由于我们是组合电路，因此现在还没有涉及到时钟的概念，只用RawModule即可
    class Detection extends Module {
        val io = IO(new Bundle{
            val in = Input(Bool()) // 输入的0-7信号，二进制表示只需要3bits宽即可
            val out = Output(Bool()) // 输出的译码后信号，8bits宽
        })

        val S0 = 0.U(3.W) // 0
        val S1 = 1.U(3.W) // 1
        val S2 = 2.U(3.W) // 11
        val S3 = 3.U(3.W) // 110

        val stat = RegInit(0.U(3.W))

        switch(stat) {
            is (S0) { when(io.in) {stat := S1} .otherwise {stat := S0} }
            is (S1) { when(io.in) {stat := S2} .otherwise {stat := S0} }
            is (S2) { when(io.in) {stat := S2} .otherwise {stat := S3} }
            is (S3) { when(io.in) {stat := S1} .otherwise {stat := S0} }
        }

        printf(p"in = ${io.in}, out = ${io.out}\n")
        io.out := stat === S3 && io.in
    }

    object testMain extends App {
        Driver.execute(args, () => new Detection)
    }
