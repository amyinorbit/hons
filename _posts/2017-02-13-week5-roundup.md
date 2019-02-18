title: Week 5 Roundup
date: 2017-02-13
template: post.html

This week was sort of a slow week, especially with other coursework to worry
about. I have started writing a packet and frame encoder to test data
processing.

One problem I encountered is the packets' variable length. If I use a `struct`
as in-memory representation of each packet, then I have two solutions for
encoding the variable length payload:

1. Store the packet's payload as a pointer and use dynamic allocation to get
memory for the payload. Not great, because it involves invoking `malloc` in
flight code.
    
        :::c
        struct packet {
            uint16_t    length;
            uint8_t*    payload;
        };
        
        struct packet myPacket = {
            .length = length,
            .payload = malloc(length)
        };
        

2. Use C99's [flexible array member][1] system, and allocate each whole packet
using `malloc`, with just enough size to store its payload. Not great either,
for the exact same reason.

        :::c
        struct packet {
            uint16_t    length;
            uint8_t     payload;
        };
        
        struct packet* myPacket = malloc(sizeof(struct packet) + length);

As it turns out, there's a better solution. While having am in-memory
representation of packets will be needed on the ground side (so that each packet
can be forwarded to the instrument's team), there is no need for that on the
bus. Each instrument's data is packetised, and the packet is then split into
frames.

Instead of going through `packetCoder > buffer > frameCoder > buffer > tx`, a
single function could generate raw binary frame data for each chunk of data
sent by an instrument. The frame stream would still encapsulate a packet, but
said packet would never have a structured in-memory representation. As a side
effect, the pipeline is now simpler: `packetFrameCoder > buffer > tx`.

 [1]: https://en.wikipedia.org/wiki/Flexible_array_member