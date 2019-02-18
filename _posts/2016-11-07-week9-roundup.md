title: Week 9 Roundup: Receiving Bytes
template: post.html
date: 2016-11-07

This week I've been experimenting with dl-fldigi (the radio decoding software
used by most high-altitude balloon missions). I've established that I'd use
RTTY for binary transmission, and [dl-fldigi][1] gives me RTTY decoding for free.

Dl-fldigi decodes the incoming byte stream as plain ASCII text. The problem is,
the goal is to transmit everything in binary, so I need some way of intercepting
the bytes between when dl-fldigi decodes the signal and when it prints it in
its window.

Lucky for me, Dl-fldigi is open-source and written in C++. Unlucky for me,
getting it to compile with clang/llvm was a nightmare. The codebase is
apparently fairly old and uses old C++ (C++03 if I'm not mistaken). Some of the
files are 1000+ lines long, with conflicting coding styles (tabs _and_ spaces,
braces on newlines or at the end), and the comments are few and far between.

To get it to compile with clang (the default compiler on), I've had to change a
few things:

 * clang/llvm comes with libc++, a more modern implementation of the C++
   standard library. Since the library is meant only for C++11 and up, I had
   to force clang to use libstdc++, the GNU implementation that supports C++03
   (`CXXFLAGS += "-stdlib=libstdc++"`).
   
 * A few syntax ambiguities would simply not compile and had to be doctored.
   There's too much to detail here, but they make up the commits [bfaa866][2]
   and [830b4e1][3] and on my fork of the project.

The next step was to add the code to extract raw bytes data. Since the SSDV
module must do the exact same thing to decode images, I dug through the code
to add my hook in the same place, in [src/cw_rtty/rtty.cxx][4]:

    :::c
	if(nbits == 8) {
        put_rx_ssdv(c, lb);
        printf("byte: 0x%02x\n", c);
    }

For the moment, it only prints values to the console in hexadecimal, but that's
enough to make sure I can extract the raw byte stream. In the future, I'll
probably open a socket and send the stream through it, so I can write the
packet parser as a separate application. It'll allow me to not have to fiddle
(too much) with dl-fldigi's icky codebase, and to bypass the radio when I'm
working on the parser itself (by feeding test packets directly to the socket).

 [1]: https://ukhas.org.uk/projects:dl-fldigi
 [2]: https://github.com/AHABus/dl-fldigi/commit/bfaa866d62b1decf384b67e599666a006c9a3cca
 [3]: https://github.com/AHABus/dl-fldigi/commit/830b4e1fbf516abf87fbf35f53b4ac032b9fc718
 [4]: https://github.com/AHABus/dl-fldigi/blob/1a45edb885796799eac9436a2b76facd8217af3a/src/cw_rtty/rtty.cxx#L514