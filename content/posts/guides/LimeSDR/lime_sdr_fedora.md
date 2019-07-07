---
title: "Setting up an GSM sniffer with LimeSDR mini on Fedora 30"
date: 2019-06-29T18:00:00
---

# Intro

So for a while now i've had an idea where i can count the number of patrons in a [pub](https://cyb.no/) i hold near and dear to my heart.
The first round of ideas was to check for Bluetooth devices and filter on whatever seems like an phone.

I dropped that idea when i realized that phones generally do not have Bluetooth activated and visible.
After that plan was scrapped i moved over to the next technology phones tend to have. Namely Wifi.
So i bought myself a [AWUS1900](https://www.alfa.com.tw/service_1_detail/15.htm) from Alfa Network.
The idea ended with me setting up a system for monitoring devices with Kismet and a metric server and more and let it run for a week in the locale of choice.

I ended up with waaay too much data. Apparently _500 000_ devices has been broadcasting in the one week... For me its sounds impossible but i don't know.
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

![GRC in action](/images/grc.png)

So now we can use our SDR as a source and we got the modules 


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

Fun fact: It's actually the first Ansible roles i've made. Ever.

You can grab it from over here: https://github.com/N0K0/limesdr_IMSI_fedora

Please fix it if it does not work ;D