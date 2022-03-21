LC3的启动与仿真
=======================

本次设计的主要任务和解答如下:

TASK1: 根据手册的描述，完成Boot模块设计代码

TASK2: 完善Boot代码后对整个LC3系统进行仿真，需要能够生成波形，并在GTKwave中查看分析

TASK1
------

.. code-block:: scala

    package LC3

    import chisel3._
    import chisel3.util._

    class Boot extends Module with HasUartCst{
    val io = IO(new Bundle() {
        val uartRx = Flipped(DecoupledIO(UInt(8.W)))

        val work    = Output(Bool())
        val initPC  = ValidIO(UInt(16.W))
        val initMem = Flipped(new MemIO)
    })

    val uartValid = io.uartRx.valid
    val uartBits = io.uartRx.bits
    io.uartRx.ready := true.B

    val timeCount = RegInit(0.U(log2Up(16*((frequency/baudRate) - 1)).W))
    timeCount := Mux(uartValid, 0.U, timeCount + 1.U)

    val inTransStart = RegInit(false.B)
    inTransStart := inTransStart | uartValid
    
    val timeOut = inTransStart && (timeCount > (((frequency/baudRate) - 1)*16).U)

    val memAddr = Reg(UInt(16.W))
    val second    = RegInit(init = false.B)     // receive uart data twice and connect the data to 16bits
    val firstData = RegEnable(uartBits, uartValid)
    val fullData  = Cat(firstData, uartBits)    // (secondData, firstData)
    val fullValid  = second && uartValid

    when(uartValid){ second := !second }
    when(fullValid){ printf("fullValid: %x\n", fullData) }

    val initpc :: initmem :: work :: Nil = Enum(3)
    val lc3State = RegInit(initpc)
    when(reset.asBool()){ lc3State := initpc }

    when(lc3State === initpc && fullValid){
        memAddr := fullData
        lc3State := initmem
    }
        
    when(lc3State === initmem && fullValid){
        memAddr := memAddr + 1.U
    }

    when(lc3State === initmem && timeOut){
        lc3State := work
        printf("Mem init finished, LC3 start work\n")
    }

    // control memory
    io.initPC.valid := (lc3State === initpc) && fullValid
    io.initPC.bits := fullData

    io.initMem <> DontCare
    io.initMem.wen := (lc3State === initmem) && fullValid
    io.initMem.wdata := fullData
    io.initMem.waddr := memAddr

    io.work := (lc3State === work)
    }
