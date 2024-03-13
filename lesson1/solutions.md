---
layout: default
title: Solutions
nav_order: 4
has_toc: false # on by default
has_children: false
comments: true
usetocbot: true
parent: hidden
---
# {{ page.title }}
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}
---

## Problem 1

```python
#import Pin from machine Module
from machine import Pin
#import time
import time
while True: #Keep running this piece of code for ever
    for i in range(0,4):
        myPin = machine.Pin(i,Pin.OUT)
        myPin.value(1) #You can also use myPin.high()
        time.sleep_ms(100) #You can also use time.sleep(in sec)
        myPin.value(0) #You can also use myPin.low()
        time.sleep_ms(100)
```

## Problem 2
```python
import machine, neopixel
import time
np = neopixel.NeoPixel(machine.Pin(18), 2) #NeoPixel is connected to Pin 18 and there are two numbered zero and one on the board
color1=[(32,178,70),(128,0,128),(152,251,152)]
color2=[(255,0,0),(127,255,212),(255,255,0)]
while True:
    for i in range(0,3):
        np[0] = color1[i]
        np[1] = color2[i]
        np.write()
        time.sleep_ms(1000)
```

## Problem 3
```python
import machine, neopixel
import time
np = neopixel.NeoPixel(machine.Pin(18), 2) #NeoPixel is connected to Pin 18 and there are two numbered zero and one on the board
CODE = {'A':'.-','B':'-...','C':'-.-.','D':'-..','E':'.','F':'..-.','G':'--.',
    'H':'....','I':'..','J':'.---','K':'-.-','L':'.-..','M':'--','N':'-.',
    'O':'---','P':'.--.','Q':'--.-','R':'.-.','S':'...','T':'-','U':'..-',
    'V':'...-','W':'.--','X':'-..-','Y':'-.--','Z':'--..',
    '0':'-----','1':'.----','2':'..---','3':'...--','4':'....-',
    '5':'.....','6':'-....','7':'--...','8':'---..','9':'----.',
    '.':'.-.-.-',
    ',':'--..--',
    '?':'..--..',
    '/':'--..-.',
    '@':'.--.-.',
    ' ':' ',
}
color1=(255,0,0)
color2=(0,0,0)

wpm=10

def flashNeo(t):
    np[0]=color1
    np[1]=color1
    np.write()
    time.sleep(t)
    np[0]=color2
    np[1]=color2
    np.write()
    return


message = "Hello World"

def send():
    tdot= 0.12 #ms
    tdash = tdot * 3
    tspace = tdot * 3
    tword = tdot * 7
    np[0]=color2
    np[1]=color2
    np.write()
    
    for l in message:
        c = CODE.get(l.upper())
        for e in c:
           if e==".":
               flashNeo(tdot)
               time.sleep(tdot)
           if e=="-":
               flashNeo(tdash)
               time.sleep(tdot)
           if e==" ": time.sleep(tword)
        time.sleep(tword)
    np[0]=color2
    np[1]=color2
    np.write()

while True:
    send()
    time.sleep(1)
```
```python
from machine import Pin,I2C
import time
from accelerometer import ADXL345

i2c = I2C(1,sda=Pin(2),scl=Pin(3), freq=10000)
adx = ADXL345(i2c)

while True:
    x=adx.xValue
    y=adx.yValue
    z=adx.zValue
    print('The acceleration info of x, y, z are:%d,%d,%d'%(x,y,z))
    roll,pitch = adx.RP_calculate(x,y,z)
    print('roll=',roll)
    print('pitch=',pitch)
    time.sleep_ms(50)
```