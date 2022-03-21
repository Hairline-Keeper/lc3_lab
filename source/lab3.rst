LC3控制器的设计
=======================

本次设计的主要任务和解答如下:

TASK1: 根据状态转移图编写状态寄存器的更新状态。

TASK2(选做): 在给出add指令的验证后，可自己设计激励验证其他指令。

TASK1
------

根据状态转移图编写状态寄存器的更新状态答案代码为：

.. code-block:: scala

  // 完整代码见 chisel_lc3/solution/Controller.scala

  when(io.work && !io.end){
    switch (state) {            // 控制状态机
      // 此处为示例: 当前状态为0, 下一状态根据ben信号转移，若为真，则下一状态为22，否则为18 
      is (0.U) { state := Mux(ben, 22.U, 18.U) }

      // lab3-task1 
      // 请在下方填写 1~59的状态转移, 其中17 19 46 53 55~58 没有对应状态
      is (1.U) { state := 18.U }
      is (2.U) { state := 25.U }
      is (3.U) { state := 23.U }
      is (4.U) { state := Mux(ir(0), 21.U, 20.U) }
      is (5.U) { state := 18.U }
      is (6.U) { state := 25.U }
      is (7.U) { state := 23.U }
      is (8.U) { state := Mux(psr, 44.U, 36.U)}
      is (9.U) { state := 18.U }
      is (10.U) { state := 24.U }
      is (11.U) { state := 29.U }
      is (12.U) { state := 18.U }
      is (13.U) { state := Mux(psr, 45.U, 37.U) }
      is (14.U) { state := 18.U }
      is (15.U) { state := 28.U }
      is (16.U) { state := Mux(r, 18.U, 16.U) }
      // no 17 state
      is (18.U) { state := Mux(int, 49.U, 33.U) }
      // no 19 state
      is (20.U) { state := 18.U }
      is (21.U) { state := 18.U }
      is (22.U) { state := 18.U }
      is (23.U) { state := 16.U }
      is (24.U) { state := Mux(r, 26.U, 24.U) }
      is (25.U) { state := Mux(r, 27.U, 25.U) }
      is (26.U) { state := 25.U }
      is (27.U) { state := 18.U }
      is (28.U) { state := Mux(r, 30.U, 28.U) }
      is (29.U) { state := Mux(r, 31.U, 29.U) }
      is (30.U) { state := 18.U }
      is (31.U) { state := 23.U }
      is (32.U) { state := ir }
      is (33.U) { state := Mux(r, 35.U, 33.U) }
      is (34.U) { state := Mux(psr, 59.U, 51.U) }
      is (35.U) { state := 32.U }
      is (36.U) { state := Mux(r, 38.U, 36.U)}
      is (37.U) { state := 41.U }
      is (38.U) { state := 39.U }
      is (39.U) { state := 40.U }
      is (40.U) { state := Mux(r, 42.U, 40.U) }
      is (41.U) { state := Mux(r, 43.U, 41.U) }
      is (42.U) { state := 34.U }
      is (43.U) { state := 47.U }
      is (44.U) { state := 45.U }
      is (45.U) { state := 37.U }
      // no 46 state
      is (47.U) { state := 48.U }
      is (48.U) { state := Mux(r, 50.U, 48.U) }
      is (49.U) { state := Mux(psr, 45.U, 37.U) }
      is (50.U) { state := 52.U }
      is (51.U) { state := 18.U }
      is (52.U) { state := Mux(r, 54.U, 52.U) }
      // no 53 state
      is (54.U) { state := 18.U }
      // no 55-58 state
      is (59.U) { state := 18.U }
      // no 60-64 state
    }
  }

TASK2
------

学生自由发挥，如and指令验证示例代码：


.. code-block:: scala

  // src/test/scala/ControllerTest.scala

  class ControllerTest extends AnyFlatSpecwith ChiselScalatestTester  // chiseltest 类和依赖
  {
      behavior of "Controller"
  
      it should "test state machine" in {   //测试用例
          test(new Controller) { c =>

              // 初始状态
              c.io.work.poke(true.B) 
              c.io.end.poke(false.B)
              c.clock.step()
              println(s"io.state=${c.io.state.peek}")    // 初始为18

              // add指令状态转移
              c.io.in.int.poke(false.B)                  // 转移条件：int为0
              c.clock.step()                             // 此处为让时钟过过一拍，让寄存器才能写入新值
              println(s"io.state=${c.io.state.peek}")    // 转移为33

              c.io.in.r.poke(true.B)                     // 转移条件：r为1
              c.clock.step()
              println(s"io.state=${c.io.state.peek}")    // 转移为35

              c.clock.step()
              println(s"io.state=${c.io.state.peek}")    // 转移为32

              c.io.in.ir.poke(5.U)                       // 转移条件：ir为5
              c.clock.step()
              println(s"io.state=${c.io.state.peek}")    // 转移为1

              c.clock.step()
              println(s"io.state=${c.io.state.peek}")    // 指令结束，转移为18

          }
      }
  }

