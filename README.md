# 2012-05-01: DEPRECATED!

Since version 7.1.1, FreeRTOS has native support for Cortex-M4F controllers.

I'll leave my project on github for reference only. Don't use it for production.

have fun,
Thomas Kindler <mail+stm32@t-kindler.de>

----

Hi!

This is the second version of my FreeRTOS port for ARM Cortex M4 cores with FPU support.

It does now support both FPU and non-FPU tasks, and tries to only save the necessary registers. 

To achieve this, the EXC_RETURN value (stored in the LR register during exceptions, esp. the PendSVCHandler) of a task is saved on it's stack. Only if bit 4 of the EXC_RETURN value indicates an extended stack frame, the FPU registers are saved or restored.

See the ARM architecture manual, B1-653 for more details.

If a task uses the FPU, it will automatically set the CONTROL.FPCA bit. No special user interaction or task registration is required.

This port is also fully compatible with the FPU lazy-save feature (which is enabled by default).

Please discuss the code in the freertos.org forum:

  https://sourceforge.net/projects/freertos/forums/forum/382005/topic/4761747

-- Notes --

Before I started, I did some performance measurements. The time for a full FPU  save/restore is quite long: A pair of vpush {s0-31}/vpop {s0-s31} takes around 400ns on my STM32F407 @ 168MHz.

On the other hand, that translates to just ~68 cycles, which is not bad if you consider the overall performance gain of FPU vs. software emulation.

To speed up interrupt processing, the CPU supports a lazy-save feature (enabled by default), which defers state saving until the FPU is actually used. 

While it seems complicated, the CPU will just do the right thing anyways:

  The AAPCS says that s0-s15 are used as scratch registers, so they're automatically (lazy)-saved on exception entry. s16-s31 are saved by the compiler. There is a performance hit of ~200ns for entry/exit if the lazy save is actually triggered. For interrupts without FPU instructions there is no additional time overhead.

The only time when all registers must be saved is when switching a task that uses the FPU (and therefore has the CONTROL.FPCA bit set). This will take about ~400ns longer than without FPU.

Keep in mind, that stack frames are a lot bigger with FPU support! Each exception will stack 26 registers (vs. 8 without FPU). The task switch context contains 34 additional registers.
