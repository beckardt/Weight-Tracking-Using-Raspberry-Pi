The goal of our project is to connect a Raspberry Pi to a digital weighing scale such that the Pi will create a line plot of a persons weight over time.
## Our Daily Progress

## Day 1 and 2
I wasn't enrolled yet, but it was all about setting up the Pi.

First step was downloading Raspian lite from this [link](https://www.raspberrypi.org/downloads/raspbian/) onto our laptop, then using etcher flash onto our raspberry pi 3.

Next we set up the basics on our raspberry pi. By using the code "sudo raspi-config" into the console, we changed internal components like allowing ssh and setting up the correct clock, as well as creating a new password. Then to connect to wifi, we followed this path:

```shell
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

Then we added this code where ssid is the name of the wifi and key_mgmt is the password to the wifi:
```shell
update_config=1
network={
        ssid="Bates Open"
        key_mgmt=NONE
}
```

## Day 3 
We connected our new Raspbery Pi 3 to wifi. We became familiar with Cron as a we created a program that would run a python file every 5 minutes.

## Day 4
We Programmed our Raspberry Pi to email us its IP address whenever it booted up. This way, we can SSH into the Raspberry Pi from our laptops without having the Pi directly hooked up to a monitor. This is called a headless setup. The first step was creating a python program on the Raspberry Pi that would first get the IP, then email both me and Xavier what it found. We did this in a file called getIP.py.


```python

import socket
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.connect(("gmail.com", 80))
msg= s.getsockname()[0]
s.close()

import smtplib

server = smtplib.SMTP('smtp.gmail.com',587)
server.starttls()
server.login("scandylyfeofpi@gmail.com", "what'supdoc")

server.sendmail("scandylyfeofpi@gmail.com","xhayden@bates.edu",msg)
server.sendmail("scandylyfeofpi@gmail.com","beckardt@bates.edu",msg)
server.quit()

```

Next, to get the pi to run getIP.py upon rebooting, we entered the following path...
```shell
sudo nano /etc/rc.local
```

Then, inputted the following code. Note that we had the program sleep, because it took the pi a non-negligable amount of time to hook up to the internet. This of course is needed to run getIP.py since it sends an email, so we don't want to have the pi run the program until wifi has been attained. 

```shell
while ! /sbin/ifconfig wlan0 | grep -q 'inet addr:[0-9]'; do
    sleep 3
done
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
  python /home/pi/getIP.py &
fi

exit 0
```

## Day 5
Following the advice of the following [link](http://graham.tech/digital-scale-hack/) we connected our HX711 to our Pi. Specifically, we connected the Raspberry Pi to the HX711 using 3.3v to vcc, Raspberry Pi BCM23 to the SCK pin (The author recommended the CLK pin, but in his picture he used the SCK pin. Our version of the HX711, does not have a CLK pin. If this is a problem, we are going to order the version that does have it), BCM24 to data(DT) and ground to ground. 

## Day 6
To get a quick idea of what the "inside" of the scale looks like: It has 4 sensors on each corner of the scale, each sends three wires to the scales chip. As seen here:

![picture 1](https://cloud.githubusercontent.com/assets/28270449/26076530/09cb4e92-3987-11e7-8119-aaf7cbfb1ac3.jpg)

Note the sets of three wires (black, white, red). The one set of two wires is from the scale's battery so we ignore that set.

Since we dont have a load combinator yet, we are going to try to get readings from just one of the four sensors for now to test python programs and if our hx711 wiring is correct. We got data from the sensor by soldering external wires to the where the sensors wires are soldered to the chip. It was messy because now there were two wires soldered to one spot, but it should work we believe. Then we connected the these wires to the hx711: Black to E-, Red to A+, and White to E+. Now there is a connection that goes from the sensor to the Pi (from the hx711). Now, we needed a library for hx711 and a script to interpret the sensors readings. We found the following [link](https://github.com/tatobari/hx711py) which provided us these things. We then copied hx711.py and example.py and attempted to run them as we put some weight on the sensor. Example.py before we do any editing of our own looks like:

```python
import RPi.GPIO as GPIO
import time
import sys
from hx711 import HX711

def cleanAndExit():
    print "Cleaning..."
    GPIO.cleanup()
    print "Bye!"
    sys.exit()

hx = HX711(5, 6)

# I've found out that, for some reason, the order of the bytes is not always the same between versions of python, numpy and the hx711 itself.
# Still need to figure out why does it change.
# If you're experiencing super random values, change these values to MSB or LSB until to get more stable values.
# There is some code below to debug and log the order of the bits and the bytes.
# The first parameter is the order in which the bytes are used to build the "long" value.
# The second paramter is the order of the bits inside each byte.
# According to the HX711 Datasheet, the second parameter is MSB so you shouldn't need to modify it.
hx.set_reading_format("LSB", "MSB")

# HOW TO CALCULATE THE REFFERENCE UNIT
# To set the reference unit to 1. Put 1kg on your sensor or anything you have and know exactly how much it weights.
# In this case, 92 is 1 gram because, with 1 as a reference unit I got numbers near 0 without any weight
# and I got numbers around 184000 when I added 2kg. So, according to the rule of thirds:
# If 2000 grams is 184000 then 1000 grams is 184000 / 2000 = 92.
#hx.set_reference_unit(113)
hx.set_reference_unit(92)

hx.reset()
hx.tare()

while True:
    try:
        # These three lines are usefull to debug wether to use MSB or LSB in the reading formats
        # for the first parameter of "hx.set_reading_format("LSB", "MSB")".
        # Comment the two lines "val = hx.get_weight(5)" and "print val" and uncomment the three lines to see what it prints.
        #np_arr8_string = hx.get_np_arr8_string()
        #binary_string = hx.get_binary_string()
        #print binary_string + " " + np_arr8_string
        
        # Prints the weight. Comment if you're debbuging the MSB and LSB issue.
        val = hx.get_weight(5)
        print val

        hx.power_down()
        hx.power_up()
        time.sleep(0.5)
    except (KeyboardInterrupt, SystemExit):
        cleanAndExit()
```

Unfortunately, we were met with random, spiking and even negative numbers that did not seem to even notice when we put weight on the sensor. After seeing this, we examined the code and found that the person who created these python scripts was using pins 5 and 6 for SCK and DT. We then tried countless variations of pins and matching code, which all resulted in relatively similar crazy numbers.

## Day 7
After our first day of no luck, we weren't sure if the issue lied in the code or our wiring or both. Our first idea to solve our issues was incorporating a breadboard. This would allow us to not have to have to have two wires soldered to the same spot, very close to the soldering of other wires. Our concern was short circuiting, so we unsoldered the cramped wires on the scales chip and sent them through a breadboard instead, keeping the other wiring constant. This method was bypassing the the scale's chip, so the weight would no longer be presented on the display on the scale. We decided we could live with this if it meant we were able to get the readings from the scale successfully. We would either send the data back from the breadboard or have the Pi speak out the weight in the final product. Unfortunately, even after using the breadboard and eliminating the possibility of short circuits, we still found no signs of recognition of weight from the python script.

Alas, after some serious frustration and confusion, we discovered that what we were attempting to do was not possible. See, the sensors on our scale have three wires coming out of them, but the scales that are being used in the demos we are following have four wires coming out of them that all go to the hx711. This is where the load combinator comes into play, it takes the three wires coming out of each of the sensors, and sends out four wires in total to the hx711. Now, we feel confident that once the load combinator arrives in the mail, we will be able to connect the wires from the sensors to it, then connect the combinator to the hx711, and keep our hx711 connected to our Pi and get some positive results!

## Day 8
The load combinator has arrived and we are excited to see the results it brings. We gathered from looking a few pictures and reading a link or two that we should connect each red wire to +, each black wire to - and each white wire to C. Then we connected A+, A-, B+, and B- to its match from the combinator to the hx711 (for example A+ on load combinator to  A+ on hx711). We then ran this [link's](https://github.com/dcrystalj/hx711py3) code, which we liked because it had a diagram showing its wiring from the hx711 to the Pi. The only difference in their wiring was that they did not need the combinator leading to the hx711 since they used a scale that had just a single sensor. We were surprised and bummed to find no significant difference in the python's scripts current output now that were using the combinator. Here is our wiring to the combinator board:
![img_2363](https://cloud.githubusercontent.com/assets/28270449/26218743/1e73e4bc-3bda-11e7-9713-88e19bbbc4e9.JPG)
![img_2364](https://cloud.githubusercontent.com/assets/28270449/26218907/b3047cb8-3bda-11e7-9be5-1365016160cf.JPG)


## Day 9
We tried switching the placement of red and white wires, so now red is soldered to C and white is soldered to +. It didn't work again... Out of no where, the program worked (using the original hx711.py and example.py from tatobari's github) and we started getting recognition when we would push on the sensors. Unfortunately, it only worked that one time, then stopped working. So we think that our placement of wires and program are correct, but just moving the scale and wires around probably messed with some of the soldering. Therefore, we think if we can come up with a better system for soldering the wires, we will get the results to stay. We decided the best way to do this was to solder the shoddy wires from the scale to better, male-male wires. We will do this by stripping some rubber off the end of the scale wires, then wrapping the exposed wires around the good wires, soldering this connection and then taping the overlap, then the good wires will be easy to solder to the combinator board cleanly. 


## Day 10 and 11
This soldering process took about two full class periods. matplotlib was installed and the method for storing the data was brainstormed. I believe a good idea is to have a document which is appended to every time a weight is recorded. This way previous data can be kept and easily modified, with the new weight. Tried running the program at the end of class 11.... NOTHING!

## Day 12
To our amazement when testing the program first thing upon coming into class, it worked. Nothing was changed in between today and last class except the scale was moved. Then, after working for a few times it stopped working, again. Today we decided we're going to just test everyway we can to see where the problem was occuring. First on the list, was switching out the wires that connect the combinator to the hx711, which fresh wires. This would check and see if any of those wires were shot. No luck after doing this. Next, we switched out the wires that connect the Pi to the hx711. Drumroll please... it worked and has not stopped working since. Hence, the issue all along may have been just that one female-female wire was bad. Regardless, It is probably good that we assumed the issue was the connection between the wires of the scale and the combinator holes because we really secured those wires and we now have faith they will in their sturdiness and won't become disconnected or short out when the scale moves. 

Now that the scale is sending information to the Pi successfully, the fun begins. Our first task is to calibrate the scale so that the numbers the program spits out are weights in pounds. We did this by first setting the reference unit to 1. Then we put an object whose weight we knew onto the scale. Then we divided the output by the known weight to find the reference unit. There was also some guess and check involved as the ratio for different items wasn't exactly the same and it is hard to get just one reading for an item because multiple outputs are shown with some variation. 

Next, we discussed how to turn on program, like when someone steps on the scale, how it knows to read the value. What we decided is that we should have the program always running, always reading values from the scale, only it will only print values above a certain threshold. We then had a really good idea... We want to be able to distinguish between different weighing sessions. So what we decided is that whenever a reading is above the threshold, its value is appended to an array all the way until the person steps off. At this point the array is printed, and then cleared so that the next session has an empty array to append to. Next step was to get just a single value for a session so that we could graph one point for that session. So now, instead of printing the array with the various readings, we print a separate array with the date and time of the session and the median value of that mass array.

## Day 13
We are now ready to start graphing points. We followed the directions of the following [link](https://wp.josh.com/2014/06/04/using-google-spreadsheets-for-logging-sensor-data/) in order to get our data to be automatically sent to google spreadsheets whenever a weighing session is complete. His directions were very good, the only difference is that since we want to send this data to google spreadsheets from inside our python program, we couldn't use curl. Instead we installed "requests", imported "requests" into our program and then all you need to do is "requests.get('url'). Using google spreadsheets is great because not only is it an easy way to append new data to a sheet, but whenever new data is appended, the line graph that we have created is automatically updated.

At this point, all we need to do is be able to distinguish between different people weighing themselves and find a way to show the weight to the person on sight (AKA not have them go to a monitor to see their weight). Hence, we have sent in an order for an LCD screen, which will have a screen to show the weight and have buttons for a person to select their profile (which we are yet to create).

## Day 14
The LCD screen was delivered, we are following the directions of the following [link](https://learn.adafruit.com/adafruit-16x2-character-lcd-plus-keypad-for-raspberry-pi/assembly) to assemble it. We added one line of code to Rc/local to have example.py run on boot up:

```shell
while ! /sbin/ifconfig wlan0 | grep -q 'inet addr:[0-9]'; do
    sleep 3
done
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
  python /home/pi/getIP.py &
  python /home/pi/example.py 
fi

exit 0
```
## Day 15 
The LCD Scale needs to be placed on the first 13 rows of pins of the Pi. This is a problem because the hx711's VCC is connected to 5V power in the first row. So, we measured the voltage of the pins on top of the LCD screen to check for 5V. We found one that read 5.15V. So we tried plugging the VCC pin of the HX711 to this pin on the LCD and running the program and it worked! Hence, we can have the LCD screen hooked up to the Pi as well as the scale set-up. Now that it is physically set up, we got it ready for use on the Pi by following this [link](https://learn.adafruit.com/adafruit-16x2-character-lcd-plus-keypad-for-raspberry-pi/usage). Then we examined some of the example codes to learn the syntax. We entered some LCD code into example.py within the loop where the person has just stepped off the scale. It worked! Easy enough, or so we thought... The LCD code portion would only work the first time a person stepped on the scale. The next time and all times after it wouldn't work. Hence, we assumed there was an error with the "while true" as the program checked to see if buttons were being pressed. We fiddled around with placing breaks at various points to no avail. Since, neither of us were too familar with "while true" loops, we decided to hand write a while loop we were more accustommed to, and it was fixed! Here is our code at the moment:
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

# I've found out that, for some reason, the order of the bytes is not always the same between versions of python, numpy and the h$
# Still need to figure out why does it change.
# If you're experiencing super random values, change these values to MSB or LSB until to get more stable values.
# There is some code below to debug and log the order of the bits and the bytes.
# The first parameter is the order in which the bytes are used to build the "long" value.
# The second paramter is the order of the bits inside each byte.
# According to the HX711 Datasheet, the second parameter is MSB so you shouldn't need to modify it.
hx.set_reading_format("LSB", "MSB")
#hx.set_reading_format("MSB", "MSB")

# HOW TO CALCULATE THE REFFERENCE UNIT
# To set the reference unit to 1. Put 1kg on your sensor or anything you have and know exactly how much it weights.
# In this case, 92 is 1 gram because, with 1 as a reference unit I got numbers near 0 without any weight
# and I got numbers around 184000 when I added 2kg. So, according to the rule of thirds:
# If 2000 grams is 184000 then 1000 grams is 184000 / 2000 = 92.
#hx.set_reference_unit(113)
hx.set_reference_unit(-12500)

hx.reset()
hx.tare()

while True:
    try:
        # These three lines are usefull to debug wether to use MSB or LSB in the reading formats
        # for the first parameter of "hx.set_reading_format("LSB", "MSB")".
        # Comment the two lines "val = hx.get_weight(5)" and "print val" and uncomment the three lines to see what it prints.
        #np_arr8_string = hx.get_np_arr8_string()
        #binary_string = hx.get_binary_string()
        #print binary_string + " " + np_arr8_string

        # Prints the weight. Comment if you're debbuging the MSB and LSB issue.
        val = hx.get_weight(5)
        if val<5:
                if(len(mass)>0):#person just stepped off
                        url = 'https://script.google.com/macros/s/AKfycbxaflfjud9NMQfimC5EKvbgyOLFKDeUaDFoT3zduc9lev2XmgeZ/exec?weight='+ str(np.median(mass)) +'&date='+str(datetime.datetime.now())
                        requests.get(url)
                        print [str(datetime.datetime.now()),np.median(mass)]
                        # Initialize the LCD using the pins
                        lcd = LCD.Adafruit_CharLCDPlate()
                        lcd.set_color(1.0, 0.0, 0.0)
                        lcd.clear()
                        lcd.message('Xavier     Ben')
                        buttons = ( (LCD.LEFT,   'Xavier weighs'+str(np.median(mass))  , (1,0,0)),
                                    (LCD.RIGHT,  'Ben weighs'+str(np.median(mass)) , (1,0,1)) )
                                 # Loop through each button and check if it is pressed.
                        while i <2:
                                 if i=1:
                                        if lcd.is_pressed(buttons[i][0]):
                                                # Button is pressed, change the message and backlight.
                                                lcd.clear()
                                                lcd.message(buttons[i][1])
                                                lcd.set_color(buttons[i][2][0], buttons[i][2][1], buttons[i][2][2])
                                                time.sleep(5.0)
                                                lcd.clear()
                                                i=3
                                        else: i=0
                                if i=0:
                                        if lcd.is_pressed(buttons[i][0]):
                                                # Button is pressed, change the message and backlight.
                                                lcd.clear()
                                                lcd.message(buttons[i][1])
                                                lcd.set_color(buttons[i][2][0], buttons[i][2][1], buttons[i][2][2])
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
It does basically everything we want it to do at this point except 1) I haven't made a google spreadsheet yet to have my weight added to and 2) we want to find a better way to have the user select who they are. At the moment, there are just two names shown (Xavier on the left and Me on the right) and the person just hits right or left. This will only work smoothly for two people. We want to be able to arrange 4 names on the screen: one centered top, one right, one left, and one centered bottom so that the four 4 buttons can be used.

## Day 16
We calibrated the scale better using another scale to compare on sight. Xavier and I made separate google spreadsheets to send data to, here is the final code:
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

# I've found out that, for some reason, the order of the bytes is not always the same between versions of python, numpy and the h$
# Still need to figure out why does it change.
# If you're experiencing super random values, change these values to MSB or LSB until to get more stable values.
# There is some code below to debug and log the order of the bits and the bytes.
# The first parameter is the order in which the bytes are used to build the "long" value.
# The second paramter is the order of the bits inside each byte.
# According to the HX711 Datasheet, the second parameter is MSB so you shouldn't need to modify it.
hx.set_reading_format("LSB", "MSB")
#hx.set_reading_format("MSB", "MSB")

# HOW TO CALCULATE THE REFFERENCE UNIT
# To set the reference unit to 1. Put 1kg on your sensor or anything you have and know exactly how much it weights.
# In this case, 92 is 1 gram because, with 1 as a reference unit I got numbers near 0 without any weight
# and I got numbers around 184000 when I added 2kg. So, according to the rule of thirds:
# If 2000 grams is 184000 then 1000 grams is 184000 / 2000 = 92.
#hx.set_reference_unit(113)
hx.set_reference_unit(-12175.0)

hx.reset()
hx.tare()
 # Initialize the LCD using the pins
lcd = LCD.Adafruit_CharLCDPlate()
lcd.set_color(1.0, 0.0, 0.0)
lcd.clear()

while True:
    try:
        # These three lines are usefull to debug wether to use MSB or LSB in the reading formats
        # for the first parameter of "hx.set_reading_format("LSB", "MSB")".
        # Comment the two lines "val = hx.get_weight(5)" and "print val" and uncomment the three lines to see what it prints.
        #np_arr8_string = hx.get_np_arr8_string()
        #binary_string = hx.get_binary_string()
        #print binary_string + " " + np_arr8_string

        # Prints the weight. Comment if you're debbuging the MSB and LSB issue.
        val =hx.get_weight(5)
        val = str(val)
        val = val[0:5]
        val = float(val)
        if val<5:
                if(len(mass)>0):#person just stepped off
                        print [str(datetime.datetime.now()),np.median(mass)]

                        lcd.message('Xavier     Ben')
                        buttons = ( (LCD.LEFT,   'Weight:'+str(np.median(mass))  , (1,0,0),'https://script.google.com/macros/s/AKfycbzQ_D_fOlq1JDe7hOYTjMG5-WJ1vdbcXao_3grixn_8j0bSr76w/exec?weight='+ str(np.median(mass)) +'&date='+str(datetime.datetime.now())),
                                    (LCD.RIGHT,  'Weight:'+str(np.median(mass)) , (1,0,1),'https://script.google.com/macros/s/AKfycbw0K53MaQljtafrPS9wbCBZ_BsdX4nkEr8m-P7BSnNqNMWxg0E/exec?weight='+ str(np.median(mass)) +'&date='+str(datetime.datetime.now())) )
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
