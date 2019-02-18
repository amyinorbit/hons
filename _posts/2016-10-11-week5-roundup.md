---
title: "Week 5 Roundup: Methods"
date: 2016-10-11
layout: post
---

This week, I have started looking into what research methods I will use to
evaluate the project and the communications bus (which I'm calling AHABus at
the moment – Abertay High Altitude Bus).

## Practical Work

The first part of the project will be practical -- I want to design and build a
hardware and software bus for High-Altitude Balloon missions. The goal is to
create something that can:

 * Provide a documented hardware data interface to payloads (Payload Data Bus).
 
 * Provide a documented software protocol that payloads must comply with.
 
 * Design the flight software controlling the main computer and the data bus.
 
 * Design the radio communication protocol used by the flight computer to
   forward data from payloads to a ground station.
   
 * Choose and implement a FEC (Forward Error Correction) algorithm to minimise
   data loss over the radio link.
   
 * Build a prototype that can be tested on the ground, and given enough time
   and funding on a test mission.

## Analysis

Once I have a prototype of the AHABus, I'll have to do a critical analysis of
it to answer the research question:

> What are the advantages, issues and obstacles in designing a generalised
> intra-payload and long-range communications platform for High-Altitude Balloon
> scientific missions?

I have broken down the question in two more focused questions I will have to
answer:

 * What is the data bus's performance? (reliability, throughput, power, timing)
 
 * What is the radio-link protocol's performance? (reliability, throughput)

Based on what I want to know, I can device a method to answer these questions:

 * [quantitative] Evaluate the radio-link system through range tests.
 
 * [quantitative] Measure data transmission, loss rates for the radio link with
   and without chosen FEC algorithms.
 
 * [quantitative] Measure the data transmission and loss rate on the data bus.
 
 * [quantitative] Measure the timing performance of the data bus: time between
   when a payload's data is available and when it's been added to the radio
   transmission queue.
 
 * [quantitative] Measure the power consumption of the data bus & computer.
 
 * [qualitative] Evaluate the performance of the overall system through
   hardware-in-the-loop, "day in the life"-style tests: a test of all systems,
   integrated, in a simulated mission timeline.

I've finished the slides of my pre-proposal presentation, which sum up all of
this, plus the aim, objectives, question and literature I have researched for
the project so far: [proposal slides][1]

  [1]: /static/prop-slides.pdf