title: Week 12 Roundup: Preliminary Packet Considerations 
template: post.html
date: 2016-11-30

This week I've started looking a bit closer about the packet & frame design
process that I'll have to do before I can implement the radio communications
part of AHABus.

The main constraint for the whole radio system is going to come from the lowest
layer (RTTY link, which is more-or-less analogous to the OSI's Link Layer). When
I experimented with plain text over RTTY, the highest bandwidth I've achieved
was 300 bauds though in a lot of cases, the highest _stable_ bandwidth was
50 bauds. Since we need a start bit and two stop bits for each octet, we're
using 11 bits per byte. That gives us a bandwidth between `50/11 = 4.5 Bps` and
`300/11 = 27.3 Bps`. This is anything but stellar (ironic for a space
application), and it's going to limit heavily what we can transmit.

I've got one card left in my sleeve to improve that: error correction code. The
idea is that each frame is part data, part checksum, and the checksum is
complete enough that it can be used to reconstruct up to a certain number of 
corrupt bits. If it works well, I could accept faster, more lossy speeds, right?

Except I can't. Well, I could, but I don't think it's a good assumption to make.
FEC is a backup, emergency system. The primary system should not be designed
any more poorly because we know there's a backup – it's a bit like saying "my
rocket can be unreliable, there's an escape system for the astronauts to use":
if I can avoid using the backup system, then I should. The amount of bit flips
that FEC can correct is limited by the percentage of the transmission allocated
to the checksum. If a high number of bits has to be corrected in nominal use,
there won't be any left when the transmission temporarily degrades because of
unpredictable factors. So the design will have to accommodate for worst-case,
and if better antennae make 300 bauds possible I'll raise the data size limit.

## Simple Packet Architecture

So far I've taken a bottoms-up approach to the design (still very much
preliminary) of the radio module's layers. The bottom layer's constrained by
what UK legal transmitters can do, so it'll be RTTY over the 70cm band at 10mW,
with a bandwidth of 50 bauds.

### Frames
   
The next layer does not have to know about the format of the data it carries.
All we care about here is reliably transmitting a stream of bytes: that's where
we'll implement forward error correction. That's our frames, and we can already
come up with a small set of requirements:

 * Provide markers for the start of a new frame, possibly its end.
 * Guarantee the order of the data stream, possibly through a sequence number
 * Provide FEC: some percentage of each frame will be a checksum.
 * (Maybe) provide a version number to allow the protocol to evolve

That's the bare minimum for a packet networking system. There's no
sender/recipient information for example, but since the link layer does not
allow multiplexed communications over the same band, there's no point: if two
payloads are talking over the same frequency in the same area, data **will** be
mangled.

That gives something like that for each frame:

    :::plain
    struct radio_frame {
        u8[]        start_marker
        u8          protocol_version
        u16         sequence_number
        b8[]        data
        b8[]        fec_code
    }

Since we have FEC at this level, frames will have to be fixed-length: we need
to be able to find the checksum with a simple offset from the start of the frame
to check that the rest isn't corrupt. I'd also prefer small frames: if frames
are long, there's a bigger chance of dropping multiple bits in the middle of a
single frame, which will cause problems when computing the FEC.

### Packets

So we've got frames, now we can transmit science stuff. I thought for a while
about having only one layer (RTTY, then directly packets), but splitting the
requirements in two levels makes it much easier to handle. Frames described
above deal with the low-level ugliness of making the transmission reliable.
Packets know about the format of the data they contain and deal with
transmitting science!

Requirements for the packets look like this:

 * Provide a start of packet marker to identify them in the byte stream.
 * Allow variable length: we don't want to use a 65,536 byte packet to transmit
   five characters.
 * Provide a protocol version number, for the same reasons exposed above.
 * (Optional) Provide a flight ID. This might go away since we can't be
   receiving data from multiple flights on the same frequency anyway.
 * Provide an ID for the instrument that generated the data: this is how we
   multiplex the data for the whole flight. On the ground, the packet decoder
   can then forward the encapsulated data to the team in charge of the
   instrument.
 * Provide a sequence number proper to the packet's originating instrument.
 * Provide ancillary (contextual) data. This is what's required to store
   non-scientific but still necessary data like GPS location and altitude.

The good thing about that system is that each burst of data generated by an
instrument gets its own packet, tagged with the instrument's ID. The flight
computer now becomes a plain, dumb encoder: ask for data, receive it if there's
any, wrap it in a packet and add it to the transmission queue. to avoid
transmitting floating point numbers for GPS coordinates (never a fun thing to
do), we can use a fixed point system (send them as 16-bit signed integers,
divid by X on reception to get the decimal version).

That gives us something like that:

    ::plain
    struct radio_packet {
        u8[]        start_marker
        u8          protocol_version
        u8          instrument_id
        u16         sequence_number
        u16         length
        u16         latitude
        u16         longitude
        u16         altitude
        u8[length]  data
    }

All of this is still very preliminary. There's going to be tonnes of tests to
run with different error correction algorithms, and probably some packing/size
optimisation to be done, given the very low bandwidth. I'm thinking reducing
the bit size of `protocol_version` and `sequence_number` for example.

The summary of the Frame/Packet format is available [as a pdf][1].

 [1]: /static/files/fcore-packets.pdf