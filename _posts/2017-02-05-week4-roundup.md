---
title: "Week 4 Roundup: Specifications and FreeRTOS"
date: 2017-02-05
layout: post
---

This week I've been working (mostly) on writing the specification for the
AHABus packet radio protocol.

## Protocols, Frames and Packets

I've (finally) created a repository to keep track of all the documentation I'm
writing for the protocols used in AHABus (for the moment packet radio, but also
payload bus communications in the future), at [github.com/ahabus/docs][1].

I've had a few "uh-oh" moments followed by long discussions with Ian about some
intricate issues that where bound to come up when writing a full network stack:

* FEC is complex (surprising no-one). After talking about it with Ian, We've
settled on using [Reed-Solomon 255,233][2] for the time being since it's
pretty well documented and has a few implementation examples available.

* FEC can detect flipped bits, but how do we work with bytes that are
completely lost to the transmission? Frames are fixed in length (256 bytes),
and the FEC code occupies the last 32 of those bytes. But if we loose one of
the first 224 bytes, then the FEC code is offset and the algorithm will fail
even if the checksum is fine.
    
    So far we've got one solution. Since each frame has a sync marker at the
start, we know how many bytes were decoded for a given frame. If it's less
than 256, then we can pad the frame's payload with null bytes, just before
the start of the FEC section. It's a bit of a Hail-Mary, but it's better
than nothing.

* Do I align packets boundaries on frames? That was the plan last week, but it
turns out it might not be that great an idea. Unlike frames, packets do not
have start markers. The only way to know when a packet ends (and when the next
one starts) is to add the current packet's length to its start index.
    
    The other solution, which is what [CCSDS/ESA does][4], is to have a flag in
each frame that indicates whether packets are aligned or not, and potentially a
one-bit flag to indicate when a frame is the start of a packet. This seems like
a good idea, even though it couples the frame and packet layers a bit more than
I'd ideally like.

## Memory Problem Solution?

After discussing the memory footprint problem I'd encountered last week with
Ian, we might have found a solution. [Wemos D1 boards][5] seem like a good
halfway point between Arduino/ATmega chips and full-fledged ARM-based boards
like the Raspberry Pi Zero. The Wemos provides more than 1MB of RAM, more than
will ever be required given the low bandwidth we have available.

The other interesting thing about the Wemos chip is that it comes with a version
of FreeRTOS a -- wait for it -- Real Time kernel. It's fairly lightweight (just
a few C files compiled with the rest of the firmware), but it would give me
a lot of things for free - simple prioritised tasks, memory usage and timing
limits.

 [1]: https://github.com/ahabus/docs
 [2]: https://ukhas.org.uk/code:rs8encode
 [3]: /2017/week3-roundup/
 [4]: http://microelectronics.esa.int/vhdl/pss/PSS-04-106.pdf
 [5]: https://www.wemos.cc/product/d1-mini.html