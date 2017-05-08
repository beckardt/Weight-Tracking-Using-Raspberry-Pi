The goal of our project is to connect a Raspberry Pi to a digital weighing scale such that the Pi will create a line plot of a persons weight over time.
## Our Daily Progress

**Day 3**: We connected our new Raspbery Pi 3 to wifi. We became familiar with Cron as a we created a program that would run a python file every 5 minutes.

**Day 4**: We Programmed our Raspberry Pi to email us its IP address whenever it booted up. This way, we can SSH into the Raspberry Pi from our laptops without the use of a desktop. This is called a headless setup. The first step was creating a python program on the Raspberry Pi that would first get the IP, then email both me and Henry what it found. We did this in a file called getIP.py.


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
