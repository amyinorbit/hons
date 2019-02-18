---
title: "Week 2 Roundup: Back to Work"
date: 2017-01-23
layout: post
---

Last week was the first week back from the holiday break, so most of my work
was, well, getting back to work. The work left to do on the project spans
multiple modules, each fairly different from the others, and I wanted to
get a good idea of what has to be done, and in what order, before I dived into
code and design documents -- I have gone down a rabbit hole far too many times.

So far, my plan for the semester is:

 1. Define (mostly done), document and implement the packet & frame system:
    binary layout of frames and packets, C encoder and decoder for latter use in
    the flight computer and the ground station receiver. This is fairly
    abstract, and should allow me to make some good progress before I need to
    worry about hardware.
 
 2. Design and implement the main event loop of the flight software, and
    integrate the packet encoder. The main loop is required for pretty much
    everything other system in the bus to function, since it will handle data,
    dispatch and time management.
   
 3. Implement software for the ground station to decode and show the contents of
    packets. It doesn't need to be advanced at this point, as long as it allows
    me to start testing and troubleshooting the complete radio transmission
    chain.
 
 4. Design, document and implement the payload bus software protocol. I know
    that I want to use I2C, and use a remote memory access protocol. This is the
    last required module for the software side of AHABus to be complete.
   
 5. Design, document and prototype the hardware side of the payload bus. The
    rest of the project does not rely on this, since all of the prototyping and
    debugging can be done with development boards and a breadboard, but it would
    be interesting to design a common hardware interface.

I plan on collecting data and doing testing in parallel with each task, to avoid
having very little time left at the end of the project to test every single
system.

The last thing I need to settle is what platform I'll use for the development
of the bus. I have leaned towards the Arduino Uno for a while because of its
simplicity and low power consumption, but available RAM (2kB) might become a
problem since packets will have to be queued while the radio transmitter is
busy. This leaves me two options (and a half):

 * Design the data rate per payload cap so that payloads cannot generate data
   faster than the radio can transmit them. This seems like a sensible approach
   anyway, since that would create a case where some data is never transmitted
   to the ground.
 
 * Use a Raspberry Pi. Even on the Pi Zero, the 1GB of RAM would dwarf the
   issue. To avoid having to deal with the default OS taking resources and
   making real-time operations complicated, the software would have to run
   "bare metal", without an OS. This means some amount of assembly to bootstrap
   C, and not using standard library functions that depend on an OS -- say,
   `malloc()` for example. This shouldn't be too much of a problem, since
   dynamic allocation in the flight software wasn't really my goal.
 
 * [the half solution] use an SD card shield on the Arduino, and write the
   packet queue to it. Let's be plain: this is a bad idea. This adds a fairly
   severe failure point (the SD card falls off its slot) and a slow code path
   (writing and reading from the card) to the most critical system of the bus.
   There will probably be a card anyway so that every bit of transmitted data
   can be archived during the flight, but relying on that for transmission seems
   iffy.
