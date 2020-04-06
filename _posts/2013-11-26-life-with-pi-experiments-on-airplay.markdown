---
layout: post
title: "Life with Pi - Experiments on AirPlay"
date: 2013-11-26 00:03
comments: true
categories: [airplay, raspberry, pi, hardware]
---
[Raspberry Pi](http://www.raspberrypi.org), this credit card sized, single-board, 700 MHz computer has become my recent, and not to mention, an interesting fascination. Being an ardent user of Mac and iOS devices, I was very keen on getting [AirPlay](http://www.apple.com/airplay/) running on this $25 device. This would not only be an value for money, but also an interesting hack.

When the search for getting AirPlay working on a Raspberry Pi began, the obvious choice was getting XBMC working on the Pi, which I would get to a little later. First I bumped on [shairplay](https://github.com/juhovh/shairplay) which is actually a very good implementation where the developer reverse engineered the Airport Express and got Airplay to tick. After downloading and installing a lot of [dependencies](http://computers.tutsplus.com/tutorials/using-a-raspberry-pi-as-an-airplay-receiver--mac-54316), shairplay started working. 
<!-- more -->
According to the few sources, shairplay should work with iOS and Android device. Unfortunately iOS is the only device I could test it with. Also, initially the sound was not audible while the music was being played through Shairport. This small hack helped to get the audio running:
```
sudo amixer cset numid=3 n
```
where n is 0=auto, 1=headphones, 2=hdmi	
Moreover, this does not provide display mirroring option from an Apple Device.

Second option to try was to run XBMC and use the inbuilt Airplay option. This comes completely out of the box, all that is should be done is to install RaspBMC and configure Airplay on it. Pretty cool, still no display mirroring.

Finally after a few more hours of search, I bumped on [rPlay](http://vmlite.com/rplay/) which ultimately turns into a winner. This piece of software not only provides streaming options, but also makes your Raspberry Pi and external display over the network. The downside of this cool software is that it is still not available for download. I had tried it out by requesting a beta tester license on their forum. I hope that VMLite soon makes it public and continues to support this awesome software rPlay. More information on installing and using rPlay can be found [here](http://adventuresandwhathaveyou.wordpress.com/2013/09/02/airplay-mirroring-on-raspberry-pi-with-rplay/).
