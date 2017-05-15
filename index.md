The goal of our project is to connect a Raspberry Pi to a digital weighing scale such that the Pi will create a line plot of a persons weight over time.
## Our Daily Progress

**Day 3**: We connected our new Raspbery Pi 3 to wifi. We became familiar with Cron as a we created a program that would run a python file every 5 minutes.

**Day 4**: We Programmed our Raspberry Pi to email us its IP address whenever it booted up. This way, we can SSH into the Raspberry Pi from our laptops without the use of a desktop. This is called a headless setup. The first step was creating a python program on the Raspberry Pi that would first get the IP, then email both me and Xavier what it found. We did this in a file called getIP.py.


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

**Day 5**: Following the advice of the following [link](http://graham.tech/digital-scale-hack/) we connected our HX711 to our Pi. Specifically, we connected the Raspberry Pi to the HX711 using 3.3v to vcc, Raspberry Pi BCM23 to the SCK pin (The author recommended the CLK pin, but in his picture he used the SCK pin. Our version of the HX711, does not have a CLK pin. If this is a problem, we are going to order the version that does have it), BCM24 to data(DT) and ground to ground. 

**Day 6**: Since we dont have a load combinator yet, we are going to try to get readings from just one of the four sensors for now to test python programs and if our hx711 wiring is correct. We got data from the sensor by soldering external wires to the where the sensors wires are soldered to the chip. It was messy because now there were two wires soldered to one spot, but it should work we believe. Then we connected the these wires to the hx711: Black to E-, Red to A+, and White to E+. Now there is a connection that goes from the sensor to the Pi (from the hx711). Now, we needed a library for hx711 and a script to interpret the sensors readings. We found the following [link](https://github.com/tatobari/hx711py) which provided us these things. We then copied hx711.py and example.py and attempted to run then as we put some weight on the sensor. Unfortunately, we were met with random, spiking and even negative numbers that did not seem to even notice when we put weight on the sensor. After seeing this, we examined the code and found that the person who created these python scripts was using pins 5 and 6 for SCK and DT. We then tried countless variations of pins and matching code, which all resulted in relatively similar crazy numbers.

**Day 7**: After our first day of no luck, we weren't sure if the issue lied in the code or our wiring or both. Our first idea to solve our issues was incorporating a breadboard. This would allow us to not have to have to have two wires soldered to the same spot, very close to the soldering of other wires. Our concern was short circuiting, so we unsoldered the cramped wires on the scales chip and sent them through a breadboard instead, keeping the other wiring constant. This method was bypassing the the scale's chip, so the weight would no longer be presented on the display on the scale. We decided we could live with this if it meant we were able to get the readings from the scale successfully. We would either send the data back from the breadboard or have the Pi speak out the weight in the final product. Unfortunately, even after using the breadboard and eliminating the possibility of short circuits, we still found no signs of recognition of weight from the python script.

Alas, after some serious frustration and confusion, we discovered that what we were attempting to do was not possible. See, the sensors on our scale have three wires coming out of them, but the scales that are being used in the demos we are following have four wires coming out of them that all go to the hx711. This is where the load combinator comes into play, it takes the three wires coming out of each of the sensors, and sends out four wires in total to the hx711. Now, we feel confident that once the load combinator arrives in the mail, we will be able to connect the wires from the sensors to it, then connect the combinator to the hx711, and keep our hx711 connected to our Pi and get some positive results!

**Day 8**: The load combinator has arrived and we are excited to see the results it brings. We gathered from looking a few pictures and reading a link or two that we should connect each red wires to +, each black wire to - and each white wire to C. Then we connected A+, A-, B+, and B- to its match from the combinator to the hx711 (for example A+ -> A+). We then ran this [link's](https://github.com/dcrystalj/hx711py3) code, which we liked because it had a diagram showing its wiring from the hx711 to the Pi. The only difference in their wiring was that they did not need the combinator leading to the hx711 since they used a scale that had just a single sensor. We were surprised and bummed to find no significant difference in the python's scripts current output now that were using the combinator.

**Day 9**: We tried switching the placement of red and white wires, so now red is soldered to C and white is soldered to +. It didn't work again... Out of no where, the program worked and we started getting recognition when we would push on the sensors. Unfortunately, it only worked that one time, then stopped working. So we think that our placement of wires and program are correct, but just moving the scale and wires around probably messed with some of the soldering. Therefore, we think if we can come up with a better system for soldering the wires, we will get the results to stay. We decided the best way to do this was to solder the shoddy wires from the scale to better, male-male wires. We will do this by stripping some rubber off the end of the scale wires, then wrapping the exposed wires around the good wires, soldering this connection and then taping the overlap, then the good wires will be easy to solder to the combinator board cleanly.

**Day 10 and 11**: This soldering process took about two full class periods. matplotlib was installed and the method for storing the data was brainstormed. I believe a good idea is to have a document which is appended to every time a weight is recorded. This way previous data can be kept and easily modified, with the new weight. Tried running the program at the end of class 11.... NOTHING!

**Day 12**: To our amazement when testing the program first thing upon coming into class, it worked. Nothing was changed in between today and last class except the scale was moved. Then, after working for a few times it stopped working, again. Today we decided were going to just test everyway we could to see where the problem was occuring. First on the list, was switching out the wires that connect the combinator to the hx711, which fresh wires. This would check and see if any of those wires were shot. No luck after doing this. Next, we switched out the wires that connect the Pi to the hx711. Drumroll please... it worked and has not stopped working since. Hence, the issue all along may have been just that one female-female wire was bad. Regardless, It is probably good that we assumed the issue was the connection between the wires of the scale and the combinator holes because we really secured those wires and we now have faith they will sturdy and wont become disconnected or short out when the scale moves. 

Now that the scale is sending information to the Pi successfully, the fun begins. Our first task is to calibrate the scale so that the numbers the program spits out are weights in pounds. We did this by first setting the reference unit to 1. Then we put an object whose weight we knew onto the scale. Then we divided the output by the known weight to find the reference unit. There was also some guess and check involved as the ratio for different items wasn't exactly the same and it is hard to get just one reading for an item because multiple outputs are shown with some variation. 

Next, we discussed how to turn on program, like when someone steps on the scale, how it knows to read the value. What we decided is that we should have the program always running, always reading values from the scale, only it will only print values above a certain threshold. We then had a really good idea... We want to be able to distinguish between different weighing sessions. So what we decided is that whenever a reading is above the threshold, its value is appended to an array all the way until the person steps off. At this point the array is printed, and then cleared so that the next session has an empty array to append to. Next step was to get just a single value for a session so that we could graph one point for that session. So now, instead of printing the array with the various readings, we a separate array with the date and time of the session and the median value of that mass array.

**Day 13** We are now ready to start graphing points. We followed the directions of the following [link](https://wp.josh.com/2014/06/04/using-google-spreadsheets-for-logging-sensor-data/) in order to get our data to be automatically sent to google spreadsheets whenever a weighing session is complete. His directions were vary good, the only difference is that since we want to send this data to google spreadsheets from inside our python program, we couldn't use curl. Instead installed "requests", imported "requests" into our program and then all you need to do is "requests.get('url'). Using google spreadsheets is great because not only is it an easy way to append new data to a sheet, but whenever new data is appended, the line graph that we have created is automatically updated.

At this point, all we need to do is be able to distinguish between different people weighing themselves and find a way to show the weight to the person on sight (AKA not have them go to a monitor to see their weight). Hence, we have sent in an order for an LCD screen, which will have a screen to show the weight and have buttons for a person to select their profile (which we are yet to create).

**Day 14**



Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/beckardt/Weight-Tracking-Using-Raspberry-Pi/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
