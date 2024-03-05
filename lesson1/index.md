---
layout: default
title: A Hands-on-Introduction to MicroPython
nav_order: 2
has_toc: false # on by default
has_children: false
comments: true
usetocbot: true
---
# {{ page.title }}
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}
---

This chapter will provide a brief overview of MicroPython.

# MicroPython: The Embedded Code Knight Rises
MicroPython is essentially Python on Keto diet. It is a lean and efficient implementation of Python, geared towards programming the embedded systems. MicroPython wields a selective arsenal, a subset of Python libraries which are required to perform necessary tasks on constrained devices.


{: .note }
The credit for making your life easier goes to the Damien George, who developed MicroPython in 2013. The idea was to bring the high level programming language to embedded systems developers. 

# The Story of LK
![Leeds Coding Kinght (LK)](./assets/lk.png){width=100,height=100}


In the bustling metropolis of Leeds City, there exists a remarkable superhero known as LK, whose extraordinary abilities lie not in super strength or flight, but in the realm of coding and technological ingenuity. LK, whose identity remains shrouded in mystery, harnesses the power of programming to craft ingenious gadgets and tools to combat crime and protect the citizens of Leeds. Armed with a brilliant mind and an arsenal of high-tech gadgets, LK navigates the city's streets with unparalleled precision, using coding as their superpower to outwit even the most cunning adversaries. Whether it's hacking into security systems, constructing intricate devices on the fly, or analyzing data to anticipate criminal activity, LK's coding prowess knows no bounds. With each keystroke, they weave a digital tapestry of justice, ensuring that Leeds City remains safe from harm.

Throughout the rest of the labs, we will be trying to trace his steps and replicate some gadgets from his arsenal.

## LK's early days
So the very first program every programmer needs to learn to write is for printing "Hello world!". This seems to be an obsession.  Why is this the case? Well there is a Wikipedia article describing this (See here) but then again LK was unconventional. His first programming stint included creating a light show on a embedded board. Luckily, you have got the same board. He started with flashing LEDs connected with GPIO0-GPI04 in a sequence. Each LED toggled from high to low after 100 ms. Your mission, if you choose to accept is replicating his steps. Before you embark on this task go through the MicroPython Beginner's [Guide](/mpython.md).
