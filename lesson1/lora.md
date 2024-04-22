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
music = {"NOTE_B0":31 , "NOTE_C1":33 , "NOTE_CS1":35 , "NOTE_D1":37 , "NOTE_DS1": 39 , "NOTE_E1":41 , "NOTE_F1":44 , "NOTE_FS1":46 , "NOTE_G1":49 , "NOTE_GS1":52 , "NOTE_A1":55 , "NOTE_AS1":58 , "NOTE_B1":62 , "NOTE_C2":65 , "NOTE_CS2":69 , "NOTE_D2":73 , "NOTE_DS2":78 , "NOTE_E2":82 , "NOTE_F2":87 , "NOTE_FS2":93 , "NOTE_G2":98 , "NOTE_GS2": 104 , "NOTE_A2":110 , "NOTE_AS2":117 , "NOTE_B2":123 , "NOTE_C3":131 , "NOTE_CS3":139 , "NOTE_D3":147 , "NOTE_DS3":156 , "NOTE_E3":165 , "NOTE_F3":175 , "NOTE_FS3":185 , "NOTE_G3":196 , "NOTE_GS3": 208 , "NOTE_A3":220 , "NOTE_AS3": 233 , "NOTE_B3":247 , "NOTE_C4":262 , "NOTE_CS4":277 , "NOTE_D4":294 , "NOTE_DS4":311 , "NOTE_E4":330 , "NOTE_F4":349 , "NOTE_FS4":370 , "NOTE_G4":392 , "NOTE_GS4":415 , "NOTE_A4":440 , "NOTE_AS4": 466 , "NOTE_B4":494 , "NOTE_C5":523 , "NOTE_CS5":554 , "NOTE_D5":587 , "NOTE_DS5":622 , "NOTE_E5":659 , "NOTE_F5":698 , "NOTE_FS5":740 , "NOTE_G5":784 , "NOTE_GS5":831 , "NOTE_A5":880 , "NOTE_AS5":932 , "NOTE_B5":988 , "NOTE_C6":1047 , "NOTE_CS6":1109 , "NOTE_D6":1175 , "NOTE_DS6":1245 , "NOTE_E6":1319 , "NOTE_F6":1397 , "NOTE_FS6":1480 , "NOTE_G6":1568 , "NOTE_GS6":1661 , "NOTE_A6":1760 , "NOTE_AS6":1865 , "NOTE_B6":1976 , "NOTE_C7":2093 , "NOTE_CS7":2217 , "NOTE_D7":2349 , "NOTE_DS7":2489 , "NOTE_E7":2637 , "NOTE_F7":2794 , "NOTE_FS7":2960 , "NOTE_G7":3136 , "NOTE_GS7": 3322 , "NOTE_A7":3520 , "NOTE_AS7":3729 , "NOTE_B7":3951 , "NOTE_C8":4186 , "NOTE_CS8":4435 , "NOTE_D8":4699 , "NOTE_DS8":4978 }


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

