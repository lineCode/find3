# Passive tracking 

## Introduction 

It is possible to also use FIND3 to setup a system that can do *passive scanning*. In *passive scanning*, you setup multiple computers which capture probe requests from phones and use those to classify their location.

A typical passive scanning system uses a network of Raspberry Pis which sniff the WiFi probe requests from WiFi-enabled devices and sends these parcels to a central server which compiles and forwards the fingerprint to the FIND server which then uses machine learning to classify the location based on the unique WiFi fingerprints.

This system does not require being logged into a particular WiFi - it will track any phone/device with WiFi enabled! (Caveat: for iOS devices it will only track if Wi-Fi is associated with a network - any network, though - because of MAC spoofing it uses for security). This system also does not require installing any apps on a phone. Tracking occurs anytime a WiFi chip makes a probe request (which is every minute or so). For this to work, it requires a one-time setup to populate the system with known fingerprints of known locations before it can pinpoint locations (see #3 below).

Note: It may be illegal to monitor networks for MAC addresses, especially on networks that you do not own. Please check your country's laws (for [US Section 18 U.S. Code § 2511](https://www.law.cornell.edu/uscode/text/18/2511)) - [discussion](https://github.com/schollz/howmanypeoplearearound/issues/4).

## Prerequisites

You will need 1+ scanner computers. Raspberry Pis with built-in Wifi work best:

- [Raspberry Pi Zero W](https://www.amazon.com/gp/product/B071L2ZQZX/ref=as_li_tl?ie=UTF8&tag=scholl-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B071L2ZQZX&linkId=ab2f9d564a4f517c5b004a760d0d6e29)
- [Raspberry Pi 3](https://www.amazon.com/gp/product/B01C6EQNNK/ref=as_li_tl?ie=UTF8&tag=scholl-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B01C6EQNNK&linkId=805012388be781415a6be827b50c76ac)

You will need a monitor-mode enabled wifi USB adapter. There are a number of possible USB WiFi adapters that support monitor mode. Here's a list that are popular:

- [USB Rt3070 $14](https://www.amazon.com/gp/product/B00NAXX40C/ref=as_li_tl?ie=UTF8&tag=scholl-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B00NAXX40C&linkId=b72d3a481799c15e483ea93c551742f4)
- [Panda PAU5 $14](https://www.amazon.com/gp/product/B00EQT0YK2/ref=as_li_tl?ie=UTF8&tag=scholl-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B00EQT0YK2&linkId=e5b954672d93f1e9ce9c9981331515c4)
- [Panda PAU6 $15](https://www.amazon.com/gp/product/B00JDVRCI0/ref=as_li_tl?ie=UTF8&tag=scholl-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B00JDVRCI0&linkId=e73e93e020941cada0e64b92186a2546)
- [Panda PAU9 $36](https://www.amazon.com/gp/product/B01LY35HGO/ref=as_li_tl?ie=UTF8&tag=scholl-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B01LY35HGO&linkId=e63f3beda9855abd59009d6173234918)
- [Alfa AWUSO36NH $33](https://www.amazon.com/gp/product/B0035APGP6/ref=as_li_tl?ie=UTF8&tag=scholl-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B0035APGP6&linkId=b4e25ba82357ca6f1a33cb23941befb3)
- [Alfa AWUS036NHA $40](https://www.amazon.com/gp/product/B004Y6MIXS/ref=as_li_tl?ie=UTF8&tag=scholl-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B004Y6MIXS&linkId=0277ca161967134a7f75dd7b3443bded)
- [Alfa AWUS036NEH $40](https://www.amazon.com/gp/product/B0035OCVO6/ref=as_li_tl?ie=UTF8&tag=scholl-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B0035OCVO6&linkId=bd45697540120291a2f6e169dcf81b96)
- [Sabrent NT-WGHU $15 (b/g) only](https://www.amazon.com/gp/product/B003EVO9U4/ref=as_li_tl?ie=UTF8&tag=scholl-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=B003EVO9U4&linkId=06d4784d38b6bcef5957f3f6e74af8c8)

Namely you want to find a USB adapter with one of the following chipsets: Atheros AR9271, Ralink RT3070, Ralink RT3572, or Ralink RT5572.

The commands I'll describe use `httpie` which can be installed with Python.

```
$ sudo python3 -m pip install httpie
```

## Setup a scanner computer

For each scanner computer you will need to use the scanning software. Follow the instructions in the [Command-line scanner](/doc/cli-scanner.md) document to install the FIND3 command-line scanner.

As before, determine your **family name** (here `FAMILY`), your **device name** (here `DEVICE`) and your WiFi interface (here `wlan0`). Make sure that the WiFi interface that you specify supports promiscuous mode.


### Start scanning passively

Choose a **device name**, like the name of your computer. We will use `DEVICE` for the rest of this document. The device name should be unique to each scanning computer.

Choose a **family name** which is a unique namespace that you can use to store data for all your devices. Each scanning computer should have the *same* family name. We will use `FAMILY` for the rest of this document.

You need to run the scanner commands using `sudo` to have priveleges to modify the WiFi card. However, if you are using Docker you don't need the `sudo` command.

```
$ sudo ./find3-cli-scanner -i wlan0 -device DEVICE -family FAMILY \
    -server https://cloud.internalpositioning.com \
    -scantime 10 -forever -passive
```

This command-line flag `-passive` tells the scanner to capture the packets with `tshark`. This command will start a scanner that submits to the main server (https://cloud.internalpositioning.com). If you set the `-forever` flag it will also continue running forever.

In this command  the WiFi chip set/unset the promiscuous mode after every scan so that it can connect to the internet to upload the packets. This process takes about 10 seconds, so it is useful to set it permanently if you don't need to connect to the internet with the scanning interface (i.e. you have two WiFi interfaces).

If you have two WiFi interfaces, you can set one to be promiscuous permanently.

```
$ sudo ./find3-cli-scanner -i wlan0 -monitor-mode
```

Then, add the `-no-modify` flag to tell the tool not to alter the promsicuousness of the interface

```
$ sudo ./find3-cli-scanner -i wlan0 -device DEVICE -family FAMILY \
    -server https://cloud.internalpositioning.com \
    -scantime 10 -forever -passive -no-modify
```


## Learning

Unlike the active scanning, to do learning on the passive scanning mode you must tell the server which device to learn on. *You should not stop the scanning tool that is running on the scanning computers*. 

First get a list of the devices that have been scanned.

```
$ http GET https://cloud.internalpositioning.com/api/v1/devices/FAMILY
```

Make sure to change `FAMILY` to the name of the family you are using. The device will be named something like `wifi-60:57:18:3d:b8:14`, where the prefix indicates the sensor (`wifi`/`bluetooth`) and the suffix is the MAC address. You can usually look up on a phone or your router information to get MAC addresses of devices. Once you have the device name you can tell the server where that device is, say the "living room".

```
$ http POST https://cloud.internalpositioning.com/passive \
     t:=1 f=FAMILY  d=wifi-60:57:18:3d:b8:14 l="living room"
```

The family (`FAMILY`) and device (for example, `wifi-60:57:18:3d:b8:14`) are specified. The flag `t:=1` is to indicate to the server that you are changing the passive scanning parameters.

Leave the device in the location for about 10 minutes to collect a good amount of fingerprints.

It is *very important* to stop learning before you move the device away from that location. To stop learning, use the same command as above but without the `location` parameter:

```
$ http POST https://cloud.internalpositioning.com/passive \
    t:=1 f=FAMILY  d=wifi-60:57:18:3d:b8:14
```

## Get data

Once you have learned several locations and are tracking with the computers, you can get data from FIND3 by consulting the [API](/doc/api.md) document.


## Issues?

If you have issues, please file one on Github at https://github.com/schollz/find3-android-scanner/issues.

## Source

If you are interested, the app is completely open-source and available at  https://github.com/schollz/find3-android-scanner.