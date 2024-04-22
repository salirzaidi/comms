---
layout: default
title: Introduction to LoRa
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

# LoRa
If you are superhero fan then it is no surprise that LK has several theme songs and tunes done as part of his brand identity. Besides this he can switch from light to acoustics, which propogate better under water. In this chapter, we will learn the art of creating tunes using PWM signals.

{: .note }
Your board comes with the Buzzer is connected to GPIO22. To play a tune, you need to generate Pulse Width Modulation Signal. 

# Pulses Width Modulation (PWM)
PWM signals are digital signals which alternate between **high** and **low** state. The duration for high state is also known as "on time" and the duration for the low state is called "off time". In order to describe the length of on time relative to off time, we use a metric called duty cycle. Duty cycle is described as percentage. It is percentage of time signal is in on state over a window of interval. For a periodic signal, its period of signal during which it assumes high state. 

![LoRa Book](../global_assets/lorabook.pdf)

To create a tune, we just need to use PWM signal with a certain duty cycle to derive the output pin connected to Buzzer. The following snippet of the code will get you started:

```python
from utime import sleep_ms
from sys import exit

class LoRaNet:
    def __init__(self,uart,appKey):
        self.uart = uart
        self.appKey = appKey
        self.band ='EU868'
        self.channels='0-2'
        self.join_EUI = None   # These are populated by this script
        self.device_EUI = None
        self.rxData = None
        self.data=None
        self.done=False
        self.status = 'not connected'
    
    def receive_uart(self):
        '''Polls the uart until all data is dequeued'''
        self.rxData=bytes()
        while self.uart.any()>0:
            self.rxData += self.uart.read(1)
            sleep_ms(2)
        return self.rxData.decode('utf-8')
    
    def send_AT(self,command):
        '''Wraps the "command" string with AT+ and \r\n'''
        buffer = 'AT' + command + '\r\n'
        self.uart.write(buffer)
        sleep_ms(300)
    
    def test_uart_connection(self):
        '''Checks for good UART connection by querying the LoRa-E5 module with a test command'''
        self.send_AT('') # empty at command will query status
        self.data = self.receive_uart()
        if self.data == '+AT: OK\r\n' : print('LoRa radio is ready\n')
        else:
            print('LoRa-E5 detected\n')
            exit()
        
    def get_eui_from_radio(self):
        '''Reads both the DeviceEUI and JoinEUI from the device'''
        self.send_AT('+ID=DevEui')
        self.data = self.receive_uart()
        self.device_EUI = self.data.split()[2]
        self.send_AT('+ID=AppEui')
        self.data = self.receive_uart()
        self.join_EUI = self.data.split()[2]
        print(f'JoinEUI: {self.join_EUI}\n DevEUI: {self.device_EUI}')
        
    def set_app_key(self,appKey):
            if self.appKey is None:
                print('\nGenerate an AppKey on cloud.thethings.network and enter it at the top of this script to proceed')
                exit()
            self.send_AT('+KEY=APPKEY,"' + self.appKey + '"')
            self.receive_uart()
            print(f' AppKey: {self.appKey}\n')
    
    
    def configure_regional_settings(self, DR='0'):
        ''' Configure band and channel settings'''
    
        self.send_AT('+DR=' + self.band)
        self.send_AT('+DR=' + DR)
        self.send_AT('+CH=NUM,' + self.channels)
        self.send_AT('+MODE=LWOTAA')
        self.receive_uart() # flush
    
        self.send_AT('+DR')
        self.data = self.receive_uart()
        print(self.data)
    
    
    def join_the_things_network(self):
        '''Connect to The Things Network. Exit on failure'''
        self.send_AT('+JOIN')
        self.data = self.receive_uart()
    
        print(self.data)

        self.status = 'not connected'
        while self.status == 'not connected':
            self.data = self.receive_uart()
            if len(self.data) > 0: print(self.data)
            if 'joined' in self.data.split():
                self.status = 'connected'
            if 'failed' in self.data.split():
                print('Join Failed')
                exit()
        
            sleep_ms(1000)
            
    def send_message(self,message):
        '''Send a string message'''
        self.send_AT('+MSG="' + message + '"')

        self.done = False
        while not self.done:
            self.data = self.receive_uart()
            if 'Done' in self.data or 'ERROR' in self.data:
                self.done = True
            if len(self.data) > 0: print(self.data)
            sleep_ms(1000)
            
    def send_hex(self,message):
        self.send_AT('+MSGHEX="' + message + '"')
        self.done = False
        while not self.done:
            self.data = self.receive_uart()
            if 'Done' in self.data or 'ERROR' in self.data:
                self.done = True
            if len(self.data) > 0: print(self.data)
            sleep_ms(1000)
```


<details>
<summary>Task 1</summary>
Try converting above code into a function which takes note and the duration as input and plays tune for that duration
</details>

# Songs of LK
Armed with arsenal of the gadget capable of making sounds, you are now required to produce songs for LKs master communicator gadget. To assist you with this task, I have created an array which holds frequency of the PWM signal for each note.

```python

from machine import Pin,I2C,UART
from utime import sleep_ms
from accelerometer import ADXL345
from LoRaNet import LoRaNet
import ubinascii

i2c = I2C(1,sda=Pin(2),scl=Pin(3), freq=10000)
uart1 = UART(1, baudrate=9600, tx=Pin(4), rx=Pin(5))
# Put your key here (string). This should match the AppKey generated by your application.
#For example: app_key = 'E08B834FB0866939FC94CDCC15D0A0BE'
app_key = "9EED71EE577D49D45CB2FDB19242F302"  #donot forget to add quotation marks after pasting appKey here
adx = ADXL345(i2c)
loranet = LoRaNet(uart1,app_key)

def i2b(number):
    c = (number >> 8) & 0xff
    f = number & 0xff
    return c,f

def getFmtData():
    x=adx.xValue
    y=adx.yValue
    z=adx.zValue
    print('The acceleration info of x, y, z are:%d,%d,%d'%(x,y,z))
    data_sent = [0,0,0,0,0,0,0,0,0]
    if(x<0):
        data_sent[0]=1
    if(y<0):
        data_sent[1]=1
    if(z<0):
        data_sent[2]=1
    x=abs(x)
    y=abs(y)
    z=abs(z)
    c,f=i2b(x)
    data_sent[3] = c
    data_sent[4] = f
    c,f=i2b(y)
    data_sent[5] = c
    data_sent[6] = f
    c,f=i2b(z)
    data_sent[7] = c
    data_sent[8] = f
    return bytes(data_sent).hex(),x,y,z


loranet.test_uart_connection()
loranet.get_eui_from_radio()

#Step 2
loranet.set_app_key(app_key)
loranet.configure_regional_settings()
loranet.join_the_things_network()

timeConstant=60000 #minute in ms
#Step 3
while True:
    hex_data,x,y,z=getFmtData()
    print(hex_data)
    loranet.send_hex(hex_data)
    #loranet.send_hex(getFmtData())
    
    sleep_ms(timeConstant*5) #10 minute


```

You can save this in a separate file and call it **notes.py**. You can then import the musical notes into your main program as follow:

```python
import notes

print(notes.music)
```

You can upload multiple files to the board by selecting files while pressng CTRL button on your keyboard. A song is combination of notes and durations for each note. See for instance a [Game of throne](https://github.com/hibit-dev/buzzer/blob/master/src/movies/game_of_thrones/game_of_thrones.ino) song and notes associated. This is obviously written in C for Arduino devices. We can create same song in MicroPython as a list e.g.

```python
song = ["NOTE_BO","NOTE_C1"]
```

<details>
<summary>Task 2</summary>
Convert the arduino code into MicroPython code to Play Game of Thrones Tune. Then modify the tune to compose custom melody for LK.
</details>

