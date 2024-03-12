---
layout: default
title: LK's eVTOL
nav_order: 6
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

# LK's eVTOL and its Tracking
In this lab the aim is to build a navigation and tracking system for LK's eVTOL using Accelerometer. We will learn, how to encode data into binary form so that we are ready to wirelessly transmit this information over Wide Area Networks.

## Story Line and Action Plan
LK has an assistant called Moneyworth, he likes to track LKs mobile during his special Ops. In today's lab we are going to build Accelerometer based system to measure X-Y-Z displacement and acceleration. Our key task is to encode this information into binary form so that this could be transmitted over the wide area wireless network in next lab.

![LK Mobil](./assets/vehicle.png)


# So what is Accelerometer?
Simply an accelerometer measures change in velocity with respect to an observer who is free fall. What is simply means is that when an accelerometer is static it read 0 m/s^2 acceleration istead of 9.8 m/s^2 . What we are going to use today is 3-axis digital accelerometer. 
[Link to Accelerometer Specs](https://www.seeedstudio.com/Grove-3-Axis-Digital-Accelerometer-16g.html)

ADXL345 is a small, thin, ultra low power 3-axis accelerometer, providing high resolution (13-bit) measurement at up to ±16 g. It measures the static acceleration of gravity in tilt-sensing applications, as well as dynamic acceleration resulting from motion or shock. The ADXL345 is interfaced with the MCU using I2C Protocol.


# I2C Protocol

The Inter-Integrated Circuit (I2C) Protocol is intended for connecting multiple **integrated chips** (ICs) with a **controller**. The protocol is intended for short-distance communication and requires only two wires for establishing connection.
![I2C Sparkfun](https://cdn.sparkfun.com/assets/learn_tutorials/8/2/I2C-Block-Diagram.jpg)
Source: Sparkfun























```python
from machine import Pin,I2C
import math
import time

device = const(0x53)
regAddress = const(0x32)
TO_READ = 6
buff = bytearray(6)

class ADXL345:
    def __init__(self,i2c,addr=device):
        self.addr = addr
        self.i2c = i2c
        b = bytearray(1)
        b[0] = 0
        self.i2c.writeto_mem(self.addr,0x2d,b)
        b[0] = 16
        self.i2c.writeto_mem(self.addr,0x2d,b)
        b[0] = 8
        self.i2c.writeto_mem(self.addr,0x2d,b)

    @property
    def xValue(self):
        buff = self.i2c.readfrom_mem(self.addr,regAddress,TO_READ)
        x = (int(buff[1]) << 8) | buff[0]
        if x > 32767:
            x -= 65536
        return x
   
    @property
    def yValue(self):
        buff = self.i2c.readfrom_mem(self.addr,regAddress,TO_READ)
        y = (int(buff[3]) << 8) | buff[2]
        if y > 32767:
            y -= 65536
        return y
     
    @property   
    def zValue(self): 
        buff = self.i2c.readfrom_mem(self.addr,regAddress,TO_READ)
        z = (int(buff[5]) << 8) | buff[4]
        if z > 32767:
            z -= 65536
        return z
           
    def RP_calculate(self,x,y,z):
        roll = math.atan2(y , z) * 57.3
        pitch = math.atan2((- x) , math.sqrt(y * y + z * z)) * 57.3
        return roll,pitch

```