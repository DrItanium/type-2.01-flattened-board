This project is a flattening of my i960 Eurocard Bus design into a mini itx
motherboard. The idea is that by leveraging the mini-itx/atx ecosystem I will
save a bunch of money. In this design, the chipset is an adafruit grand central
m4 with ram being provided by a file on an SD card (dedicated SD channel +
an effective cache implementation works wonders to hide nearly all latency).

The grand central m4 is exposed out the io shield area. You will see the
evolution of many ideas and where it started from as well. 

I have two 4809 microcontrollers on board as well. They are titled ME and PIC. 

ME or Management Engine abstracts the details of how one handles memory
transactions made by the i960. It generates the 20MHz clock and runs in lock
step with the i960Sx as well. The chipset (GCM4) interacts with the ME to
satisfy these requests. The communication protocol between chipset and ME is
designed to allow the chipset to run completely independently of the ME without
running too fast or out of sync. The ME also uses its CCLs to reduce the number
of ICs required. This includes:

- Enabling data lines on request by chipset
- Fail signal detection (externally synchronized)
- Ready signal detection with external override
- Controlling the i960 reset line (Chipset, ME, and PIC all have to be ready
		before i960 is pulled out of reset). Pullups and an external inverter
		are used to make sure that the i960 will stay in reset until everything is
		ready. 

PIC or Programmable Interrupt Controller makes sure that one can invoke any of
the four exposed interrupt lines by the i960Sx in a consistent manner. A serial
communication channel is provided to allow the chipset to configure the PIC.
The general idea is that each CCL of the PIC is devoted to a separate interrupt
line. The first three interrupts (~INT0, INT1, INT2) are meant for dedicated
interrupts within the i960. The third interrupt (~INT3) is the lowest priority
and designed for the chipset. When the chipset needs to interrupt the i960, it
sends an ~INT3. This causes the i960 to query the chipset what kind of
interrupt it was (through MMIO) and then send an IAC message to trigger an
internal interrupt. 

The PIC has peripherals like timers and such that can be connected to the CCLs
to trigger interrupts. The chipset talks with the PIC through the serial
connection to setup these options via the event system. 
