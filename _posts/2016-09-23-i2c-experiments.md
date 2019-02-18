---
title: "Experiments: AVR, I2C Communications"
date: 2016-09-23 10:00:00
template: post
---

These past two weeks, I've worked on getting I2C working between the Pi and two
Arduinos. I've also started to look into raw C programming for AVR controllers
(the Arduino Uno is based around an [ATMega328p][1]) to avoid the fairly bloated
Arduino library.

## AVR-C

For our application (balloon flight computer, powered by a battery), we want to
use as little power as possible: less power means a smaller battery, which in
turns means a lighter payload, a smaller balloon and higher altitudes.

The Arduino library is very good for quick prototyping because it abstracts
away a lot of the low-level work required to code a controller in pure C, but it
comes at a cost.

    :::c
    // Setting pin 13 HIGH with Arduino
    #define PIN 13

    digitalWrite(PIN, HIGH); 

Setting an output high or low takes about [50 processor cycles][2], because the
function needs to look up a table to find the port that pin 13 is mapped to.

    :::c
    // Setting pin 13 HIGH in pure C
    // Pin 13 is bit 5, on port B
    #define PIN 5
    
    PORTB |= (1 << PIN);

In pure C, you can't work with just a pin number: the 13 digital pins are spread
on two 8-bit ports, one bit on each port representing the output state of one
pin. On the other hand, changing a pin's state only takes two cycles.

It may seem like nothing, but having code execute faster also means we could use
a slower controller, which will use less power. I've started building myself a
small library to cover the basic functions — LCD screen and UART/Serial at the
moment.

## I2C bus

The big advantage of I2C over the serial bus I was experimenting with
during week 1 is that it only requires two lines: `SCL` (clock line) and `SDA`
(data line). Every device is connected to the same bus, and addressing is done
through the data line rather than through individual Device Select lines.

This time I went wth a python script on the Pi's side because I wanted to try
something quickly.

    :::python
    #! /usr/bin/env python

    import smbus
    import time

    bus = smbus.SMBus(1)

    def writeNumber(address, value):
    	"""writes an 8-bit number on the I2C bus"""
        bus.write_byte(address, value)

    def readNumber(address):
    	"""reads an 8-bit number from the I2C bus"""
        num = bus.read_byte(address)
        return num

    def main():
        while True:
		
    		# Ask for an address and data to send, send it
    		# to the Arduino.
            add = input('target: ')
            var = input('  data: ')
            writeNumber(add, var)
            print('rpi->arduino(0x%02x): %d' % (add, var))
		
    		# Wait 1 second for the arduino to process the data.
            time.sleep(1)
		
    		# Read the number from the same arduino.
            number = readNumber(add)
            print('rpi<-arduino(0x%02x): %d' % (add, number))

    if __name__ == '__main__':
        try:
            main()
        except KeyboardInterrupt:
            print('\r\nexiting...')


The programme is fairly straightforwards: it asks for an I2C address and a
number, sends the number to the device at the address, and then reads an 8-bit
number from that same address.

The code running on Arduino is the most basic: when a byte is received, it
stores it in a global variable, and echoes it back on the bus when the bus
master requests to read from the Arduino's address.

    :::cpp
    #include <Wire.h>
    #define SLAVE_ADDRESS 0x4

    void setup() {
        Wire.begin(SLAVE_ADDRESS);
    
        // Setup the callbacks
        Wire.onReceive(receiveData);
        Wire.onRequest(sendData);
    }

    void loop() {
        delay(100);
    }

    // callback for received data
    void receiveData(int byteCount){
        while(Wire.available()) {
            number = Wire.read();
        }
    }

    // callback for sending data
    void sendData(){
        Wire.write(number);
    }

The system works pretty well, and also underlines the main feature/problem of
I2C: it's a Controller/Controlled system, where controlled devices cannot talk
_at all_ until requested to do so by the controller.

It's good because it means payloads can't pollute the Bus with data unless the
flight computer has addressed them, but it also means the flight computer will
need to poll each device regularly to find out whether they have data to send or
not, instead of each payload sending a signal when data is available.

   [1]: http://www.atmel.com/Images/Atmel-42735-8-bit-AVR-Microcontroller-ATmega328-328P_datasheet.pdf
   [2]: https://www.peterbeard.co/blog/post/why-is-arduino-digitalwrite-so-slow/