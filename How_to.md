# How to: Digital Weight Tracking With Raspberry Pi
## By Ben Eckardt and Xavier Hayden

During Short Term 2017 at Bates College, Ben and Xavier set out to hack a digital scale such that it could send its weight readings to a Raspberry Pi, where the readings would be manipulated and sent to a line graph tracking a persons weight. Here is the final product:

![img_2414](https://cloud.githubusercontent.com/assets/28270449/26220800/1b737cfc-3be2-11e7-989b-d7b99a628fe1.JPG)

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


### 2. Physical Set Up
  i. Unsolder the scale's sensor wires from the scale's chip.
  
  The next two steps are useful because the scale's wires will most likely be cheap and flimsy. We recommend this to allow for cleaner, easier soldering to the combinator board.
  
  ii. Strip some of the rubber coating from each wire so that you have enough of a metal tip to work with.
  
  iii. for each wire, wrap the exposed metal part around the exposed metal part of the solid core wires to enable a stronger connection, and solder the two together, then protect the soldering by taping the region closed with electrical tape.
  
  iv. Solder the wires to the combinator board. The combinator board has groups of holes for the upper left sensor (UL), upper right sensor (UR), lower left sensor (LL), and lower right sensor (LR). These match up with the sensors as if the scale was right side up. So when the scale is upside, the upper right sensors wires go to the upper left group, etc...

  v. Solder shield headers to load combinator out holes and to hx711 in/out holes like so:
  
  vi. Using female-female wires, connect the Pi to the HX711 like so: BCM5 to DT, ground to ground, BCM6 to SCK and 5V to VCC. And the combinator board to the hx711 like so: Red to E+, Black to E-, White to A-, and Green to A+.
  



