# How to: Digital Weight Tracking With Raspberry Pi
## By Ben Eckardt and Xavier Hayden

During Short Term 2017 at Bates College, Ben and Xavier set out to hack a digital scale such that it could send its weight readings to a Raspberry Pi, where the readings would be manipulated and sent to a line graph tracking a persons weight. Here is the final product:

![img_2414](https://cloud.githubusercontent.com/assets/28270449/26220800/1b737cfc-3be2-11e7-989b-d7b99a628fe1.JPG)

Please note that a LCD screen was used in the scale to allow the user to select his or her profile (so the program would know where to send the person's weight) and to show the person's weight. The scale's original screen was taken out of the equation because the wires leading to the chip were redirected.

And here is a few days of weight tracking for Xavier:

![screen shot 2017-05-18 at 3 57 51 pm](https://cloud.githubusercontent.com/assets/28270449/26221010/ec77765a-3be2-11e7-8b76-eefd1890bf73.png)

## List of Parts and Costs

1. Raspberry Pi 3 Model B - 40$
2. *Digital Bathroom Scale - 12$
3. Hx711 Load Cell Amplifier - 3$
4. Sparkfun Load Sensor Combinator - 3$
5. Adafruit i2c 16x2 RGB LCD Pi Plate - 20$
6. Breadboard female to female wires - 4$
7. Solid Core Wire - 3$
8. Shield Headers - 2$
*Price of scale varies, though any digital scale is compatible for this project.

Materials NOT Listed:

1. Basic materials for Raspberry Pi (i.e. power source, monitor hookup, etc.)
2. Solder Equipment.




## Instructions

### 1. Preparing Your Pi and Google Spreadsheet

i. Follow the directions of this [link](https://wp.josh.com/2014/06/04/using-google-spreadsheets-for-logging-sensor-data/) to set up your google spreadsheet. (make sure to copy the url given to you at the end). Then insert a line graph chart.

ii. Go to the following [link](https://github.com/tatobari/hx711py) and create equivalent hx711.py and example.py files by copy and pasting their contents into files you have created on your Pi. For new programmers, type this in your shell:
```shell 
Nano hx711.py
```
then copy and pasting the given hx711.py code inside. Then do the same for example.py



### 2. Physical Set Up Before LCD Screen
  i. Unsolder the scale's sensor wires from the scale's chip.
  
  The next two steps are useful because the scale's wires will most likely be cheap and flimsy. We recommend this to allow for cleaner, easier soldering to the combinator board.
  
  ii. Strip some of the rubber coating from each wire so that you have enough of a metal tip to work with.
  
  iii. for each wire, wrap the exposed metal part around the exposed metal part of the solid core wires to enable a stronger connection, and solder the two together, then protect the soldering by taping the region closed with electrical tape. Here is a picture of the wrapping of the two wires, before soldering and taping:
  
  
  
  iv. Solder the wires to the combinator board. The combinator board has groups of holes for the upper left sensor (UL), upper right sensor (UR), lower left sensor (LL), and lower right sensor (LR). These match up with the sensors as if the scale was right side up. So when the scale is upside down, the upper right sensors wires go to the upper left group, etc... We connected each red wire to its corresponding C hole, each White wire to its corresponding + hole, and each black wire to its corresponding - hole on the the combinator board. We've read that scales can vary here so you might need to mess around with the wiring.

  v. Solder shield headers to load combinator out holes and to hx711 in/out holes like so:
  
  ![img_9547](https://cloud.githubusercontent.com/assets/28270466/26418795/8d628d5a-408b-11e7-9c8b-a7969bb10a79.JPG)\
  
  vi. Using female-female wires, connect the Pi to the HX711 like so: BCM5 to DT, ground to ground, BCM6 to SCK and 5V to VCC. And the combinator board to the hx711 like so: Red to E+, Black to E-, White to A-, and Green to A+.
  
  ### 3. Testing your Scale and Wiring
   
   i. Run hx711.py to see if there are any errors. If so you will probably just have to "Sudo pip install" something.
   
   ii. Run example.py to see if there are any errors. If so you will probably just have to "Sudo pip install" something. If there are no errors you should be met with some numbers that increase up as you push on the sensors/scale. If this is the case, set your reference unit by following the directions in example.py comments. (you will use the same reference unit in weight_tracker.py)
   
  ### 4. Using the LCD
  
   i. Follow the steps on this [link](https://learn.adafruit.com/adafruit-16x2-character-lcd-plus-keypad-for-raspberry-pi/assembly) to assemble the LCD Screen.
   
   ii. Since the LCD Screen is most easily hooked up to the Pi by sitting it on top of the first 13 rows of pins, we need a new way to attach the hx711's VCC to 5V power. Solve this problem by measuring the voltage of the pins on top of the LCD and seeing which ones are around 5 volts. Whichever is like this, we can attach the VCC to this pin through a female-female wire. Here is a picture of the LCD screen on the Pi:
   
   iii. Now create a file called weight_tracker.py and copy this code into it and make the necessary adjustments as denoted where ever you see ###.
   
   ```python

import RPi.GPIO as GPIO
import time
import sys
from hx711 import HX711
import numpy as np
import datetime
import requests
import Adafruit_CharLCD as LCD


def cleanAndExit():
    print "Cleaning..."
    GPIO.cleanup()
    print "Bye!"
    sys.exit()
mass = []
hx = HX711(5, 6)

### Use your same reference unit as found previously
hx.set_reference_unit(-12175.0)

hx.reset()
hx.tare()
 # Initialize the LCD using the pins
lcd = LCD.Adafruit_CharLCDPlate()
lcd.set_color(1.0, 0.0, 0.0)
lcd.clear()

while True:
    try:
        val =hx.get_weight(5)
        val = str(val)
        val = val[0:5]
        val = float(val)
        if val<5:
                if(len(mass)>0):#person just stepped off
                        print [str(datetime.datetime.now()),np.median(mass)]
                        ### Put in the names of the people who's weight you will track, if there are more than two, you will need to assign them to other buttons. 
                        ### next replace the (User 1 URL) with the Url given of your google spreadsheet, keeping the single quote at the front
                        lcd.message('User1     User2')
                        buttons = ( (LCD.LEFT,   'Weight:'+str(np.median(mass))  , (1,0,0),'(User 1 URL)?weight='+ str(np.median(mass)) +'&date='+str(datetime.datetime.now())),
                                    (LCD.RIGHT,  'Weight:'+str(np.median(mass)) , (1,0,1),'(User 2 URL)?weight='+ str(np.median(mass)) +'&date='+str(datetime.datetime.now())) )

                                 # Loop through each button and check if it is pressed.

                        i=0
                        while i <2:
                                if i==1:
                                        if lcd.is_pressed(buttons[i][0]):
                                                # Button is pressed, change the message and backlight and send data to spreadsheet.
                                                lcd.clear()
                                                lcd.message(buttons[i][1])
                                                lcd.set_color(buttons[i][2][0], buttons[i][2][1], buttons[i][2][2])
                                                url=buttons[1][3]
                                                requests.get(url)
                                                time.sleep(5.0)
                                                lcd.clear()
                                                i=3
                                        else: i=0
                                if i==0:
                                        if lcd.is_pressed(buttons[i][0]):
                                                # Button is pressed, change the message and backlight and send data to spreadsheet.
                                                lcd.clear()
                                                lcd.message(buttons[i][1])
                                                lcd.set_color(buttons[i][2][0], buttons[i][2][1], buttons[i][2][2])
                                                url=buttons[0][3]
                                                requests.get(url)
                                                time.sleep(5.0)
                                                lcd.clear()
                                                i=3
                                        else: i=1
                time.sleep(1)
                mass=[]
        else:
                mass.append(val)


        hx.power_down()
        hx.power_up()
        time.sleep(0.5)
    except (KeyboardInterrupt, SystemExit):
        cleanAndExit()
                             
```
   iii. Now you can run weight_tracker.py, step on the scale, step off the scale, choose your name, and your weight will be sent to your spreadsheet!
  



