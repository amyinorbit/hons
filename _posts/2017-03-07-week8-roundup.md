title: Week 8 Roundup
date: 2017-03-07
template: post.html

I spent the first few weeks of the semester laying groundwork. This week, I've
started implementation work on the radio protocol systems (encoder and decoder)
and dissertation writing (mostly structure and literature review at the moment).

The radio protocol is implemented in two "libraries": `RTXEncoder`, to be used
on platforms emitting AHABus frames, and `RTXDecoder`, used to extract packet
data from a frame stream.

The library lives in its own GitHub repository at [ahabus/packet-radio][1].
The goal was to make it reusable, which means any interaction with buffers is
done through user-supplied callbacks. The version that will be built into
FCORE will probably be modified to write directly to the RTTY subsystem -- I
want to avoid jumping through function pointers on embedded systems.

The Reed-Solomon encoding and decoding is provided by a two-file library written
by Phil Karn (KA9Q). The format of AHABus frames proved a problem when it comes
to computing the RS codes: The maximum chunk of data that can be encoded with
RS(223, 255) is 255 bytes, which means one byte had to be excluded from frames
before encoding.

The SSDV protocols achieves this by removing the frame sync marker from the
checksum computation (if the sync marker is corrupted, the frame will not be
detected by the receiver anyway). The problem is, AHABus uses a two-byte sync
marker (`0xAA,0x5A`).

I ended up changing the protocol specifications slightly: frames now only carry
a one-byte start marker (`0x5A`), and must be preceded by at least one sync
byte (`0xAA`), which is not counted as part of the 256 bytes of the frame. A
example stream will look like this:

    ::plain
    0xAA 0xAA [0x5A (rest of frame)] 0xAA [0x5A (rest of frame)] ...

The Reed-Solomon codes can now be computed using bytes 1 to 222 in the frame.

 [1]: https://github.com/AHABus/packet-radio
