---
title: "Week 10 Roundup: FreeRTOS/FCORE"
date: 2017-03-19
layout: post
---

The project is finally taking shape his week I've started assembling each of the
building blocks I've designed since September into [FCORE][1] -- Flight Computer
Operations and Resources Environment -- what's the point of doing a space project
if you don't use NASA-level backronyms?

## FCORE's Real Time Architecture

FCORE is based on FreeRTOS, which saves me from implementing my own task
management and scheduling system. FreeRTOS provides two types of "tasks": Tasks
are preemptively schedule (the scheduler can interrupt a task at any time to
allow a higher-priority task to run), and coroutines are cooperatively scheduled
(each coroutine must `yield` execution back to the scheduler).

I like coroutines because I get to decide when to hand back control, and I can
be sure that a task B won't go into action when task A is polling data from a
payload. On the other hand, tasks are much better supported in recent versions
of FreeRTOS, and provide a way to mark certain regions of code as critical.
They also provide better isolation, since each task gets its own stack
(coroutines all share the same stack). Tasks and coroutines can also be mixed,
with tasks taking priority over coroutines (which kind of defeats their 
purpose).

In the end, the architecture I'm going with for the time being is:

 * A System/Watchdog task. It runs every few seconds and sends a health packet
   to the ground station if nothing has been sent in a while, to make sure
   the ground has up-to-date GNSS coordinates. Additionally, it sends a list of
   payloads that have failed and have been pulled out of the data bus.
   [Priority 10]
 
 * A GNSS task that runs every minute (to be determined). It fetches the latest
   [NMEA GGA sentence][ ] output by the GNSS receiver, parses it and stores the
   updated position in memory. [Priority 5]
 
 * One AHABus task per payload. Each task runs at an interval determined by the
   importance the user has given to the payload, fetches data from its
   instrument, puts it in a packet alongside the current position stored in
   memory, and writes the packet to the radio transmission buffer.

UART, I2C and RTTY operations are done inline, without their own tasks.

## Implementation Issues

So far, I've implemented UART communications, RTTY radio transmission and the
GNSS task. UART/RTTY were mostly a matter of porting the code I wrote for AVR
last semester. Noteworthy things and issues:

 * RTX is driven by `UART1`, since RTTY is really just UART 8N2, over radio.
   `UART1` can only transmit and not receive, which isn't a problem since I
   only have bus-to-ground transmissions.
 
 * On the ESP, `UART0` and `UART1` share the same interrupt mask, which is a
   bit tricky to handle. Any bug when setting up one UART usually leads to both
   acting up, which makes for fun debugging hours (days). The differentiation is
   done in the Interrupt Service Routine, by looking at each UART's status flags
   register.
 
 * While AVR's UART chip only holds the byte being sent, the ESP's UARTs have
   their own FIFO buffer in hardware. Being only 127 bytes long, they're not
   enough to ensure stable transmissions (the radio runs at 200 bauds, which
   means any significant amount of data would cause the buffer to overflow).
   Instead, I listen for interrupts that signal that buffers are filled below
   (Tx) or above (Rx) a threshold, and copy the next few bytes in/out of the
   hardware buffer from/to larger software ring buffers.
 
 * The `UART_TXFIFO_EMPTY` and `UART_RXFIFO_FULL` interrupts do not fire only
   when the threshold is crossed, but _any time_ the buffer is filled above or
   below the threshold. This means that the interrupt must be disabled on top
   of clearing the interrupt flag once it has been serviced.

Discovering all that was akin to a four day game of incredibly un-fun
hide-and-seek because of the very lacking english documentation of the ESP, but
it works reliably enough that I could use UART and RTX to implement the GNSS
task. To avoid reinventing the wheel (something I've already been doing a lot
for this project), I used Kosma Moczek's [minmea][2] library to parse GNSS
NMEA output.

The task's synopsis is really straightforward

 1. Clear the UART Rx buffer to start from a clean slate.
 2. Wait three seconds to get GNSS receiver data.
 3. Isolate the first NMEA sentence (`'$'` -> `'\r\n'`).
 4. Check if the sentence is valid, and a GGA sentence.
     * If the sentence is valid GGA, store coordinates.
     * Else, go to 3.
 5. Wait 60 seconds, repeat.

To avoid the possibility of entering an infinite loop, I put a limit on the
maximum number of sentences that can be checked. If more than four fail, the
task ignores NMEA output and goes directly to the wait step.

 [1]: https://github.com/AHABus/fcore
 [2]: https://github.com/cloudyourcar/minmea