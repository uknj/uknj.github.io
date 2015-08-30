---
layout: post
title: Ardupilots and Ultrasounds!
---

As part of my degree, I was asked to design and build an autonomously controlled ground vehicle with a team. The big caveat was that it was all to be done within a budget of 250 GBP.

Note: This included construction of chassis and electronics etc etc.

The design method we decided to use for the navigation was to use an [Ardupilot](http://rover.ardupilot.com/). In order to avoid obstacles, we settled upon using ultrasonic sensors. This was because they were suitable for the required location (outdoors with little to no chance of interference) and that the Ardupilot firmware could navigate based on such devices [see here](http://rover.ardupilot.com/wiki/common-optional-hardware/common-rangefinder-landingpage/).

However upon further research it was discovered that the Ardupilot only directly supported only a few ultrasonic rangefinders. The cheapest of these was 30USD (almost 20 GBP) and would take a large chunk out of our already tight budget. (The ardupilot and GPS cost roughly in the location of 50GBP combined). While researching ultrasonic sensors I came accross the [HC-SR04](http://www.icstation.com/arduino-ultrasonic-module-sr04-distance-transducer-sensor-p-1389.html#.VJRZ-14MAY) sensor, these cost $2 each and were much cheaper. Since they were so cheap, I ordered some to figure out if they could be used in conjunction with the APM.

I discovered through trial, error and research that the APM only accepted analogue voltage, PWM would not work. The HC-SR04 however worked by measuring the time the pulse took to return [see documentation](http://www.micropik.com/PDF/HCSR04.pdf). Running on an Arduino and using the equation given in the documentation example, the distance to the object could be returned. However this was still a far cry from the voltage required by the APM to operate. It might have been possible to modify the firmware on the APM to work directly with the ultrasonic sensor, however this would have been much too time consuming for our needs.

We therefore decided to output the voltage as PWM values from an Arduino, convert it to steady voltage and input it to the APM.

In order to convert the PWM to voltage, we used a DAC (Digital to analogue converter). The one that proved most suitable for the job was the [LTC2644](http://www.linear.com/product/LTC2644). Subsequently, the circuit was buildt for the LTC2644 as shown in the [documentation](http://cds.linear.com/docs/en/datasheet/2644f.pdf).IN^A and IN^B were the PWM outputs from the Arduino respectively, and V^OUTA and V^OUTB were the pins that will go into the Ardupilot. NB: I could only get one of the ultrasound sensors to actually communicate with the APM, however both do output steady voltage and should in theory work.

I then connected the the V^OUT into the APM as shown [here](http://copter.ardupilot.com/wiki/common-optional-hardware/common-rangefinder-landingpage/sonar/#connecting_the_sonar_sensor_on_apm_2x), where the Voltage and Ground were the standard voltage ground connections on the Arduino and Pin 3 was the V^OUT. After setting up the parameters correctly, the ultrasound sensor worked fine.

The code uses an equation (the map function) that converts the distance into PWM values. This mapping will vary based on the distance of the sensor we are emulating. This line of code (`map(ultraleft.Ranging(CM), 0, 431, 0, 70)`) uses the [map function](https://www.arduino.cc/en/Reference/Map) is based on the 10m sensor and according to its [datasheet](http://maxbotix.com/documents/XL-MaxSonar-EZ_Datasheet.pdf), "For the 10 meter sensors (MB1260, MB1261, MB1360, MB1361), this pin outputs analog voltage with a scaling factor of (Vcc/1024) per 2 cm.  A supply of 5V yields ~4.9mV/2cm., and 3.3V yields ~3.2mV/2cm."  After some maths the above map function can be derived. Somewhat arbitrarily I used a maximum of 431 for the value that the HC-SR04 sensor will return as I found that when approaching the maxmimum value of 5m the sensor would appear to give somewhat erratic values.

I do believe that the function can still be optimized, however for a ground based rover it is accurate enough. <s>I will at some point upload the table of values/settings used for the APMw, will update when it is done.</s> EDIT 30/08/2015: This is now done, please check the relevant git commit.

If you have any improvements to the code, please do make a pull request.


Happy Hacking,
Uknj
