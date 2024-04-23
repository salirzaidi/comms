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

# LoRa and LoRaWAN
LoRa (Long Range) is radio frequency (RF) modulation technology that is suited for low power communication in Wide Area Networks (WANs). It operates in the license free Industrial, Scientific and Medical (ISM) frequency bands, for example, 915 MHz, 868 MHz, and 433 MHz. Typical coverage area for LoRa based communication is up to 3 miles in urban areas and 10 miles in rural areas. Due to its wide range and low power requirements, it is particularly well-suited for internet of things (IoT) communication where large number of low power devices send/receive small amounts of data forming a low power wide area network (LPWAN). LoRa is ideal for data transmission at a longer range compared to technologies like WiFi, Bluetooth or ZigBee making it a popular choice for sensors and actuators that operate in low power mode. 

LoRA Wide Area Network (LoRaWAN) is a Media Access Control (MAC) layer protocol built on top of LoRa modulation. It is a software layer which defines how devices use the LoRa hardware, for example when they transmit, and the format of messages. A LoRa network consists of nodes, gateways and LoRa network operators. Typically, nodes broadcast data to be picked up by gateways that forward the information to operator servers for processing.A LoRa transmitter broadcasts messages to gateways that forward the messages the LoRaWAN network server/cloud for processing. More details on LoRaWAN, its parameters and working is left for self study. An excellent resource can be found here: [LoRa Book][def]
{: .Objective of the Lab }
In this lab, you will learn some basics about the tecnhology along as well as send your accelerometer data (x,y,z) from previous lab using LoRa transciever (node) interfaced with the MakerPi RP2040 board we have been using. The data will be broadcasted to and will be picked up by one of The Things Network (TTN) gateways deployed across Leeds. You will be using Thonny IDE for coding as before.
# The Things Network (TTN)
The Things Network provides a global, open LoRaWAN network with a set of open tools and to build an IoT application at low cost. We will be using TTN gateway for receiving our accelerometer data from a LoRa IoT node.
<details>
<summary>Task 1--Getting started with TTN</summary>
To begin, navigate to [The Things Network Homepage](https://www.thethingsnetwork.org/). You will see the following page:
![ScreenshotofIDE1](./assets/SC1 (ttn webpage).jpg)

Click login as shown and choose **Experiment and explore with The Things Network** option. ![ScreenshotofIDE2](./assets/SC2 (ttn).jpg) On the next page, use login detals as follows:

Email: leeds_labs

Password: lora1406@

Then click **Login with The Things ID**. After loggin in, click on console from the user drop down menu on the right as shown ![ScreenshotofIDE3](./assets/SC2 (ttn console).jpg). To configure regional settings, use United Kingdom from the **Device or gateway lcation** dropdown and select **Europe 1** from existing clusters. ![ScreenshotofIDE4](./assets/SC3 (ttn settings).jpg. Choose **Create Application** on the next page. ![ScreenshotofIDE5](./assets/SC4 (create app).jpg)

Enter a unique Application ID on the following screen along with an **Accelerometer Test** as application name and description in respective text boxes as shown. Click create application button. Try out a few if the one you entered already exists untill you have a unique ID to create an application.
</details>

# Setting up the board

 Follow the following steps:
1. Create a new folder called Lab 3.
2. Navigate to this folder from Thonny IDE.
3. Create two new files in this folder, one called **main.py** and other called **LoRaNet.py**.
4. In **LoRaNet.py** file copy the code which is provide below.

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



[def]: ../global_assets/lorabook.pdf