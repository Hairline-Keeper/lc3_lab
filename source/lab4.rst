LC3数据通路的设计与验证
=======================

本次设计的主要任务和解答如下:

1. ALU模块设计和验证
   
2. Regfile模块设计
   
3. Regfile验证
   
4. 编写数据通路中部分电路

TASK1
-------

ALU模块设计代码如下：

.. code-block:: scala

  // 完整代码见 chisel_lc3/solution/ALU.scala

  class ALU extends Module{
    val io = IO(new Bundle{
      val ina = Input(UInt(16.W))
      val inb = Input(UInt(16.W))
      val op  = Input(UInt(2.W))    //ADD,AND,NOT,PASSA
      val out = Output(UInt(16.W))
      val c   = Output(UInt(1.W))
    })
    val result = Wire(UInt(17.W))

    io.out := DontCare
    io.c := DontCare
    result := DontCare

    // lab4-task1
    // 在此编写运算器逻辑
    switch (io.op) {
      is (0.U) {
        result := io.ina +& io.inb
        io.out := result(15,0)
        io.c := result(16)
      }
      is (1.U) { io.out := io.ina & io.inb }
      is (2.U) { io.out := ~io.ina }
      is (3.U) { io.out := io.ina }
    }
  }


ALU完整的验证代码在项目中给出，用于学习模块级仿真验证。

TASK2
-------

Regfile模块设计代码如下：

.. code-block:: scala

  完整代码见 chisel_lc3/solution/Regfile.scala

  class Regfile extends Module{
    val io = IO(new Bundle {
      val wen = Input(Bool())
      val wAddr = Input(UInt(3.W))
      val r1Addr = Input(UInt(3.W))
      val r2Addr = Input(UInt(3.W))
      val wData = Input(UInt(16.W))
      val r1Data = Output(UInt(16.W))
      val r2Data = Output(UInt(16.W))
    })

    io.r1Data := DontCare
    io.r2Data := DontCare

    val regfile = RegInit(VecInit(Seq.fill(8)(0.U(16.W))))

    when(io.wen){
      regfile(io.wAddr) := io.wData
    }
    io.r1Data := regfile(io.r1Addr)
    io.r2Data := regfile(io.r2Addr)
  }


TASK3
-------

Regfile模块验证代码如下：

.. code-block:: scala

  // 由于此任务在实验中给出结果，因此答案也作为注释放在项目中
  // chisel_lc3/src/test/scala/RegfileTest.scala

  package LC3

  import chisel3._
  import chiseltest._
  import org.scalatest.flatspec.AnyFlatSpec
  import scala.util.Random

   class RegfileTest extends AnyFlatSpec
     with ChiselScalatestTester
   {
     behavior of "Regfile"
  
     def TEST_SIZE = 10

     val data1, data2, addr1, addr2 = Array.fill(TEST_SIZE)(0)

     for (i <- 0 until TEST_SIZE) {
       data1(i) = Random.nextInt(0xffff)
       data2(i) = Random.nextInt(0xffff)
       addr1(i) = Random.nextInt(7)
       addr2(i) = Random.nextInt(7)
     }

     // 硬件部分
     it should "test r/w" in {
       test(new Regfile) { c =>
         println(s"*******regfile read/write test********")
         for(i <- 0 until TEST_SIZE) {
           c.io.wData.poke(data1(i).U)
           c.io.wAddr.poke(addr1(i).U)
           c.io.wen.poke(true.B)
           c.clock.step()

           c.io.r1Addr.poke(addr1(i).U)
           c.io.wen.poke(false.B)
           c.io.r1Data.expect(data1(i).U(15,0))
           c.clock.step()

           c.io.wData.poke(data2(i).U)
           c.io.wAddr.poke(addr2(i).U)
           c.io.wen.poke(true.B)
           c.clock.step()
        
           c.io.r2Addr.poke(addr2(i).U)
           c.io.wen.poke(false.B)
           c.io.r2Data.expect(data2(i).U(15,0))
           c.clock.step()
         }
       }
     }
   }

TASK4
-------
完善数据通路主要编写LD信号控制的寄存器值改变逻辑，Gate信号控制数据源的选择逻辑，和Mux信号控制的数据选择逻辑。其余作为框架代码已经在项目中给出。关键代码如下：

.. code-block:: scala

  // 完整代码见 chisel_lc3/solution/DataPath.scala

  // ...

  // 1. Mux控制信号
  val ADDR1MUX = Mux(SIG.ADDR1_MUX, regfile.io.r1Data, PC)

  val ADDR2MUX = MuxLookup(SIG.ADDR2_MUX, 0.U, Seq(
    0.U -> 0.U,
    1.U -> offset6,
    2.U -> offset9,
    3.U -> offset11
  ))

  val addrOut = ADDR1MUX + ADDR2MUX

  val PCMUX = MuxLookup(SIG.PC_MUX, RESET_PC, Seq(
    0.U -> Mux(PC===0.U, RESET_PC,  PC + 1.U),
    1.U -> GATEOUT,
    2.U -> addrOut
  ))


  val DRMUX = MuxLookup(SIG.DR_MUX, IR(11,9), Seq(
    0.U -> IR(11,9),
    1.U -> R7,
    2.U -> SP
  ))

  val SR1MUX = MuxLookup(SIG.SR1_MUX, IR(11,9), Seq(
    0.U -> IR(11,9),
    1.U -> IR(8,6),
    2.U -> SP
  ))

  val SR2MUX = Mux(IR(5), offset5, regfile.io.r2Data)

  val SPMUX = MuxLookup(SIG.SP_MUX, regfile.io.r1Data+1.U, Seq(
    0.U -> (regfile.io.r1Data+1.U),
    1.U -> (regfile.io.r1Data-1.U),
    2.U -> SP, // TODO: Supervisor StackPointer
    3.U -> SP  // TODO: User StackPointer
  ))

  val MARMUX = Mux(SIG.MAR_MUX, addrOut, offset8)
  
  val VectorMUX = MuxLookup(SIG.VECTOR_MUX, 0.U, Seq(  // TODO: Interrupt
    0.U -> 0.U,
    1.U -> 0.U,
    2.U -> 0.U
  ))

  val PSRMUX = 0.U

  // ...

  // 2. Gate 控制信号
  val GateSig = Cat(Seq(
    SIG.GATE_PC,
    SIG.GATE_MDR,
    SIG.GATE_ALU,
    SIG.GATE_MARMUX,
    SIG.GATE_VECTOR,
    SIG.GATE_PC1,
    SIG.GATE_PSR,
    SIG.GATE_SP
  ).reverse)

  GATEOUT := Mux1H(GateSig, Seq(
    PC,
    MDR,
    alu.io.out,
    MARMUX,
    Cat(1.U(8.W), 0.U),
    PC - 1.U,
    Cat(Seq(0.U(13.W),PSRMUX)),
    SPMUX
  ))

  // ...

  // 3. LD 控制信号
  when(SIG.LD_MAR) { MAR := GATEOUT }
  when(SIG.LD_MDR) { MDR := Mux(SIG.MIO_EN, IN_MUX, GATEOUT) }

  when(SIG.LD_IR)  { IR  := MDR }
  when(SIG.LD_BEN) { BEN := IR(11) && N || IR(10) && Z || IR(9) && P }
  when(SIG.LD_PC || time === 0.U)  { PC := PCMUX }

  when(SIG.LD_CC) {
    N := dstData(15)
    Z := !dstData.orR()
    P := !dstData(15) && dstData.orR()
  }

  // ...
