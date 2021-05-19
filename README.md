Some great progress with an Ecovac Deebot N79 main board. I was able to dump almost all of the peripheral registers, and I can also dump SRAM. I know of two exploits I might be able to use to extract the firmware, and I am currently trying to get one to work.

I also found out that the Dibea X500 (and maybe others in this brand) is also built on the same design as the Rv750 and N79... even though it looks very different.

 # Snark_Iun_Robot_Vacuum
UPDATES 04/29/2021 At End:
UPDATES 08/12/2020 At End:
UPDATES 08/29/2020 At End:

The objective is to take a Shark Ion RV750 robot vacuum, and perform some Steve Austin stuff to make it much more...

The Shark ION RV750 is a pretty standard, no-frills, robot vacuum.
The brains of the system comes from a STM32F071VBT6 mcu, and the rest of the components on the board are of the standard variety of easy to find and inexpensive. 
There is also an Azure Wireless CU300 daughter board connected through serial, but through my testing I found that it does not provide any support logic outside of interfacing with the Shark Clean App.

My original idea was to extract the firmware from the STM32, but this proved to be a dead end I think.
What I found was that the mcu was protected with RDP Level 1 ( SWD still works in Boot Loader, but ROM is protected), and normally this would not be an issue with the several methods that exist to bypass this on the Cortex-M0, but it seems somebody had the great idea of re-assigning the SWDIO and SWCLK pins in the main program... they are not connected to anything else.

I then turned to the Shark Clean android app to scrape any usefull information from it. I did find an URL addresses to an AWS server where was a raw xml page with link to development and production firmware’s for the newer Shark Ion robot vacuums.
I did try to reverse a firmware from the RV750N, but it seems the firmware is either highly obfuscated or encrypted.

In a last ditch effort, I decided to pull the 4MB SPI flash IC off the AW CU300 and read the contents...
Well the SPI flash is not encrypted, and there is a decent amount of information. I ventured down a rabbit hole of information within some java script files about encryption/decryption, but it turned out to be the routine for exchanging shared keys for the Ayla IOT server that is used. 

I did find a small bit of information on the firmware decryption... SHA512?! WTF! It seems to spill out almost the entire decryption routine EXCEPT the IV!

So this is where I am at now.
I plan on creating my own custom FW to use the STM32 as a very basic ROS node to communicate with a RPi Zero W (yeah, I know!) as the master.
The hope is to re-create most of the original cleaning path algorithm for the STM32 to execute, and pass sensor messages to the Rpi. The RPi will keep track of the sensor data from the STM32. The Rpi will also receive data from an IMU and a HLS-LFCD2 LiDAR sensor. From this sensor data a map of the environment will be created where the Rpi can determine the pose and location in the environment.

A wish would to also be able to implement a camera and open CV for object detection such as cords and rugs that can entangle the robot.


Oh yeah, and last of all.... I am a noob with much of this, and learning on the fly!

UPDATES 08/12/2020:

I worked with the CU300 WiFi module some more, and found a serial console on one of the ports... only problem is I seem to only have the receive side. What I thought would have been the transmit to the CU300 has a odd pulse sequence at startup that reminds me of an IR remote protocol, maybe Manchester?

I have also made some progress on the flash dump from the SPI flash. 

This blog has helped quite a bit: https://medium.com/@urish/inside-the-bulb-adventures-in-reverse-engineering-smart-bulb-firmware-1b81ce2694a6
It seems that one of the segments of the Marvell firmware is obfuscated through some word reversal/swap routine. It looks simple at face value, but we shall see!

08/14/2020:

For posterity sake, https://hackaday.io/project/167594/logs I may forget about this later. This seems to be using the somewhat same method as the Shark.

UPDATES 08/29/2020:

In an interesting turn of events, I found that my sister has an Eufy Robovac 11. Opening it up "to clean it" I found it is very similar to the Shark RV750. It is missing some features, and a few board components are in different areas, but definitely based of the same base as the RV750... is the RV750 even original?
From what I can find, it seems like the RV750_N is functionally the same as the base RV750, but it uses a STM32F103x MCU. The WiFi module is also integrated into the main board, and I don’t know what SoC it uses.
More sleuthing online has revealed that the ECOVACS Deebot N79S is also based of the same design as the RV750_N.

Working on the firmware side of things, I have only made a little progress on the obfuscated firmware. I may be on track to figuring out the header format. Going back the Shark AWS site I found that there are a lot of *.zip files, and they all contain un-encrypted firmware… it seems like they all might be for later models as there is mention of cameras for SLAM based Nav and Mapping.

Started working on the project again, made some good discoveries, but still can't figure out the obfuscation of the firmware I am interested in. Added an example file that points to my target file.

