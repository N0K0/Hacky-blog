---
title: "Setting up an GSM sniffer with LimeSDR mini on Fedora 30"
date: 2019-06-29T18:00:00
toc: true
---

# Intro

So for a while now i've had an idea where i can count the number of patrons in a [pub](https://cyb.no/) i hold near and dear to my heart.
The first round of ideas was to check for Bluetooth devices and filter on whatever seems like an phone.

I dropped that idea when i realized that phones generally do not have Bluetooth activated and visible.
After that plan was scrapped i moved over to the next technology phones tend to have. Namely Wifi.
So i bought myself a [AWUS1900](https://www.alfa.com.tw/service_1_detail/15.htm) from Alfa Network.
The idea ended with me setting up a system for monitoring devices with Kismet and a metric server and more and let it run for a week in the locale of choice.

I ended up with way too much data. Apparently _500 000_ devices has been broadcasting in the one week... For me its sounds impossible but i don't know.
Further testing with Wifi was abruptly stopped when the dongle died :(
So with my money refunded for the Wifi dongle i figured it would be way more fun testing something new instead of screwing around more with Wifi.

I ended up looking into SDRs! I was looking into LimeSDR and a HackRF and for some magical reason i opted for picking up the LimeSDR.
The reason why is that specwise the LimeSDR is way better as basically everything, but unfortunately less supported by the community.

Anyways now you know how i ended up here, lets look into how we can get all our things to play nice together.



# The perquisites

For this setup i need the following:

* [A LimeSDR Mini](https://www.crowdsupply.com/lime-micro/limesdr-mini)
* [Fedora 30](https://getfedora.org/en/workstation/download/)
    * No packages is really needed other than the ones listed during install

*Note: I have not tried setting this up in a virtual machine.*

# Installation

## LimeSDR Suite
*A natural start is fixing with the LimeSDR system itself*


*Lime Suite is a collection of software supporting several hardware platforms including the LimeSDR,
drivers for the LMS7002M transceiver RFIC, and other tools for developing with LMS7-based hardware. 
Installing the Lime Suite enables many SDR applications such as GQRX to work with supported hardware through the bundled SoapySDR support module.*

```
sudo dnf install git cmake libsqlite3x-devel gcc-c++ libusb-devel libi2c-devel SoapySDR-devel freeglut-devel
```

And the to make and install LimeSuite
```
cd /tmp/
git clone https://github.com/myriadrf/LimeSuite.git
cd LimeSuite
mkdir build
cd build
cmake ..
make -j `nproc`
sudo make install
```

If you wish for the LimeSDR to be available for all users and not just root:

Add the following file to `/etc/udev/rules.d/64-limesuite.rules`
```
SUBSYSTEM=="usb", ATTR{idVendor}=="04b4", ATTR{idProduct}=="8613", SYMLINK+="stream-%k", MODE="666"
SUBSYSTEM=="usb", ATTR{idVendor}=="04b4", ATTR{idProduct}=="00f1", SYMLINK+="stream-%k", MODE="666"
SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="601f", SYMLINK+="stream-%k", MODE="666"
SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", ATTR{idProduct}=="6108", SYMLINK+="stream-%k", MODE="666"
```

And to finish up reload the rules
```
udevadm control --reload-rules
udevadm trigger
```


When this is setup you should run a quick self test:  
https://wiki.myriadrf.org/Testing_the_LimeSDR#Testing_the_LimeSDR_Mini  
Basically just run `LimeQuickTest --no-gui`


## GNU Radio Companion
*For some reason an application with __Radio Companion__ in the title seems reasonable to have*

For GNU Radio things are easy peezy! Just install the one from the default repository <3

```
sudo dnf install gnuradio
```

After this install we would still not be able to use our LimeSDR Mini, we still need some add-ons to be able to interact with it.

### GNU Radio Companion Add-ons

We are going to install two add-on first one is so that we can use LimeSDR as a source for GRC, and a add-on which implements the GSM support that we need.
First packages is `gr-limesdr` add-on, the second one is the `gr-gsm`. 
**NOTE:** the gr-gsm version we need is the one from MyriadFM (the creators of the LimeSDR) which supports the Lime. 
The compatibility patch has unfortunately not been upstreamed yet. Check out the following issue for updates:  
https://github.com/ptrkrysik/gr-gsm/issues/393


#### gr-limesdr

Installing packages

```
dnf install git cmake boost-devel swig
```


And compile and install what we need

```
cd /tmp/
git clone https://github.com/myriadrf/gr-limesdr
cd gr-limesdr
mkdir build
cd build
cmake ..
make -j 4
sudo make install
```

#### gr-gsm

Installing packages
```
dnf install git cmake libsqlite3x-devel gcc-c++ libtool boost-devel swig libosmocore-devel cppunit-devel gnuradio-devel gr-osmosdr octave fltk-devel SoapySDR-devel wxGTK3-devel mingw64-wxWidgets3
```

And compile and install what we need
```
cd /tmp/
git clone https://gitlab.com/myriadrf/gr-gsm.git
cd gr-gsm
mkdir build
cd build
cmake ..
make -j 4
sudo make install
```


## Adding it all together
_GNU Radio ✅ LimeSDR software ✅ gr\_gsm ✅ gr\_limesdr ✅_  
_Ready to go!_

Sweet! Now we got all the packages we need. Just to verify we need to open GRC and look for the following blocks in the right hand pane:

![GRC in action](/images/LimeSDR/grc.png)

So now we can use our SDR as a source and we got the modules needed to parse the GSM data going over the airwaves.
Next up is actually finding some GSM traffic and fetch some datagrams!

What you installed the grgsm packet you also installed a couple of GRC programs which are quite useful.
Unfortunately like with everything else we are going to need a tweak.. 
Even tough we downloaded and installed gr_gsm from MyriadRF we are still installing a version of the tools which is based on the upstream gr_gsm. 
That is, its using the wrong type of radio source! But not always... More on that later.

I must say, it felt like i knew what i was doing when i figured out how to fix the program 😎

### Tweaking the GRC programs

**FILL OUT**

So here is a image of the graph that makes the g

# So what really is GSM?
*The Global System for Mobile Communications (GSM) is a standard developed by the European Telecommunications Standards Institute (ETSI)
 to describe the protocols for second-generation (2G) digital cellular networks used by mobile devices such as mobile phones and tablets.*

Basically its the way our phones talked to the network between the 1990s ans 2010ish.
Its still used by quite a few phones as an fallback protocol for when 4G and 3G fails, 
fortunately more and more phones prohibit the usage of GSM due to weaknesses found in the protocol.

GSM has a couple of possible frequency bands which is in use where the two most popular om global basis must be the
GSM900 and GSM1800 bands.

|  Band   | Device -> Tower | Tower -> Device |
|---------|-----------------|-----------------|
| GSM900  | 890 - 915 MHz   | 935 - 960 MHz   |
| GSM1800 | 1710 - 1785 MHz | 1805 - 1880 MHz |

We also got a bunch of variations like [GSM-R](http://en.wikipedia.org/wiki/GSM-R), [E-GSM](https://en.wikipedia.org/wiki/GSM_frequency_bands#E-GSM)

The full list can be found here: [GMS_frequency_bands](https://en.wikipedia.org/wiki/GSM_frequency_bands)

# Whats next?

There is two things i want to look more into.
What is possible to do with LTE without 

## What about LTE
What can be done with LTE?

2) Make it so that our Simple IMSI script can listen to multiple channels at once. The Lime can easily listen to four or five channels at once.
So it should be possible to make the 

# I'm lazy. Can this be scripted?

**YES!**

Since i've been running this setup on a machine i tend to reinstall every couple of weeks i ended up setting up a couple
of Ansible scripts. It worked last time i ran it ¯\\\_(ツ)\_/¯
I pinkyswear that i'll compress the filestructure to only what we need at some point

Fun fact: It's actually the first Ansible roles i've made. Ever.

You can grab it from over here: https://github.com/N0K0/limesdr_IMSI_fedora

Please fix it if it does not work ;D