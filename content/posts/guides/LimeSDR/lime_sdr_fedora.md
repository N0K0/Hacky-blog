---
title: "Setting up an GSM sniffer with LimeSDR mini on Fedora 30"
date: 2019-06-29T18:00:00
toc: true
---

**All the data gathered during the creation of this blog post has been deleted, i do not feel like i have an adequate solution.**

# Intro

So for a while now i've had an idea where i can count the number of patrons in a [pub](https://cyb.no/) i hold near and dear to my heart.
The first round of ideas was to check for Bluetooth devices and filter on whatever seems like an phone.

I dropped that idea when i realized that phones generally do not have Bluetooth activated/visible.
After that plan was scrapped i moved over to the next technology phones tend to have. Namely Wifi.
So i bought myself a [AWUS1900](https://www.alfa.com.tw/service_1_detail/15.htm) from Alfa Network.
The idea ended with me setting up a system for monitoring devices with Kismet and a metric server and more and let it run for a week in the locale of choice.

I ended up with way too much data. Apparently _500 000_ devices had been broadcasting in that one week... For me its sounds impossible, but i don't know..
Further testing with Wifi was abruptly stopped when the dongle died :(
So with my money refunded for the Wifi dongle i figured it would be way more fun testing something new instead of screwing around more with Wifi.

I ended up looking into SDRs! I was looking into LimeSDR and a HackRF and for some magical reason i opted for picking up the LimeSDR.
The reason why is that specwise the LimeSDR is way better at basically everything, but unfortunately less supported by the community it seems.

Anyways now you know how i ended up here, lets look into how we can get all our things to play nice together as well as getting some tracking up and running.


![picture of the setup](/LimeSDR/setup.jpg)

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

{{< highlight sh >}}

sudo dnf install git cmake libsqlite3x-devel gcc-c++ \
    libusb-devel libi2c-devel SoapySDR-devel freeglut-devel
{{< / highlight >}}


And the to make and install LimeSuite
{{< highlight sh >}}
cd /tmp/
git clone https://github.com/myriadrf/LimeSuite.git
cd LimeSuite
mkdir build
cd build
cmake ..
make -j `nproc`
sudo make install
{{< / highlight >}}


If you wish for the LimeSDR to be available for all users and not just root:

Add the following file to `/etc/udev/rules.d/64-limesuite.rules`
{{< highlight sh >}}
SUBSYSTEM=="usb", ATTR{idVendor}=="04b4", ATTR{idProduct}=="8613", SYMLINK+="stream-%k", MODE="666"
SUBSYSTEM=="usb", ATTR{idVendor}=="04b4", ATTR{idProduct}=="00f1", SYMLINK+="stream-%k", MODE="666"
SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="601f", SYMLINK+="stream-%k", MODE="666"
SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", ATTR{idProduct}=="6108", SYMLINK+="stream-%k", MODE="666"
{{< / highlight >}}


And to finish up reload the rules
{{< highlight sh >}}
udevadm control --reload-rules
udevadm trigger
{{< / highlight >}}



When this is setup you should run a quick self test:  
https://wiki.myriadrf.org/Testing_the_LimeSDR#Testing_the_LimeSDR_Mini  
Basically just run `LimeQuickTest --no-gui`


## GNU Radio Companion
*For some reason an application with __Radio Companion__ in the title seems reasonable to have*

For GNU Radio things are easy peasy! Just install the one from the default repository <3

{{< highlight sh >}}
sudo dnf install gnuradio
{{< / highlight >}}


After this install we would still not be able to use our LimeSDR Mini, we still need some add-ons to be able to interact with it.

### GNU Radio Companion Add-ons

We are going to install two add-on first one is so that we can use LimeSDR as a source for GRC, and a add-on which implements the GSM support that we need.
First packages is `gr-limesdr` add-on, the second one is the `gr-gsm`. 
**NOTE:** the gr-gsm version we need is the one from MyriadFM (the creators of the LimeSDR) which supports the Lime. 
The compatibility patch has unfortunately not been upstreamed yet. Check out the following issue for updates:  
https://github.com/ptrkrysik/gr-gsm/issues/393


#### gr-limesdr

Installing packages

{{< highlight sh >}}
dnf install git cmake boost-devel swig
{{< / highlight >}}



And compile and install what we need

{{< highlight sh >}}
cd /tmp/
git clone https://github.com/myriadrf/gr-limesdr
cd gr-limesdr
mkdir build
cd build
cmake ..
make -j 4
sudo make install
{{< / highlight >}}


#### gr-gsm

Installing packages
{{< highlight sh >}}

dnf install git cmake libsqlite3x-devel gcc-c++ libtool \
                boost-devel swig libosmocore-devel cppunit-devel \
                gnuradio-devel gr-osmosdr octave fltk-devel SoapySDR-devel \
                wxGTK3-devel mingw64-wxWidgets3
{{< / highlight >}}


And compile and install what we need

{{< highlight sh >}}
cd /tmp/
git clone https://gitlab.com/myriadrf/gr-gsm.git
cd gr-gsm
mkdir build
cd build
cmake ..
make -j 4
sudo make install
{{< / highlight >}}

## Adding it all together
_GNU Radio ✅ LimeSDR software ✅ gr\_gsm ✅ gr\_limesdr ✅_  
_Ready to go!_

Sweet! Now we got all the packages we need. Just to verify we need to open GRC and look for the following blocks in the right hand pane:

![GRC in action](/LimeSDR/grc.png)

So now we can use our SDR as a source and we got the modules needed to parse the GSM data going over the airwaves.
Next up is actually finding some GSM traffic and fetch some frames!

When you installed the grgsm packet you also installed a couple of GRC programs which are quite useful.
Unfortunately like with everything else we are going to need a tweak.. 
Even tough we downloaded and installed gr_gsm from MyriadRF we are still installing a version of the tools which is based on the upstream gr_gsm. 
That is, its using the wrong type of radio source! But not always... More on that later.

I must say, it felt like i knew what i was doing when i figured out how to fix the program 😎

### Tweaking the GRC programs
So here is a image of the graph that makes the grgsm_livemon

![grgsm_livemon original](/LimeSDR/grgsm_livemon_original.png)

It's not really all that much we need to change to make the program work again. Simply change the source block (and maybe the samplerate)
The resulting graph from the change looks like this:

![grgsm_livemon limesdr](/LimeSDR/grgsm_livemon_limesdr.png)

If you are in a hurry you can take my version of the grc file from here:

[grgsm_livemon_limesdr.grc](/LimeSDR/grgsm_livemon_limesdr.grc) (With GUI)  
[grgsm_livemon_limesdr_headless.grc](/LimeSDR/grgsm_livemon_limesdr_headless.grc) (Without GUI)



# So what is GSM?
*The Global System for Mobile Communications (GSM) is a standard developed by the European Telecommunications Standards Institute (ETSI)
 to describe the protocols for second-generation (2G) digital cellular networks used by mobile devices such as mobile phones and tablets.*

Basically its the way our phones talked to the network between the 1990s ans 2010ish.
Its still used by quite a few phones as an fallback protocol for when 4G and 3G fails (or you are living in a rural area), 
fortunately more and more phones prohibit the usage of GSM due to weaknesses found in the protocol.

GSM has a couple of possible frequency bands which is in use where the two most popular om global basis must be the
GSM900 and GSM1800 bands.

|  Band   | Device -> Tower | Tower -> Device |
|---------|-----------------|-----------------|
| GSM900  | 890 - 915 MHz   | 935 - 960 MHz   |
| GSM1800 | 1710 - 1785 MHz | 1805 - 1880 MHz |

We also got a bunch of variations like [GSM-R](http://en.wikipedia.org/wiki/GSM-R), [E-GSM](https://en.wikipedia.org/wiki/GSM_frequency_bands#E-GSM)

The full list can be found here: [GMS_frequency_bands](https://en.wikipedia.org/wiki/GSM_frequency_bands)

Since i'm from Norway we are mainly using the GSM900 band for our phones and therefor this is the area i will look for devices.

# Finding and capturing our frequencies
So i know i'm in Norway, next step now is finding the frequencies in my area.
There is a couple of different ways to do this. Like using SDRAngel and just poke around the spectrum like so:

![grgsm_livemon limesdr](/LimeSDR/SDRAngel_gsm.png)

Its quite fast since the LimeSDR can look at such an insane width at once. Normally you can't really see more than 2 MHz at once.

If you don't want to install tools like SDRAngel too you already got the tool `grgsm_scanner` installed.
For some reason not explored it supports the Lime already...

It works as follows:

{{< highlight sh >}}
[n@mamluk hugo]$ grgsm_scanner --help
Usage: grgsm_scanner: [options]

Options:
  -h, --help            show this help message and exit
  -b BAND, --band=BAND  Specify the GSM band for the frequency. Available
                        bands are: GSM900, DCS1800, GSM850, PCS1900, GSM450,
                        GSM480, GSM-R
  -s SAMP_RATE, --samp-rate=SAMP_RATE
                        Set sample rate [default=2000000.0] - allowed values
                        even_number*0.2e6
  -p PPM, --ppm=PPM     Set frequency correction in ppm [default=0]
  -g GAIN, --gain=GAIN  Set gain [default=24.0]
  --args=ARGS           Set device arguments [default=]
  --speed=SPEED         Scan speed [default=4]. Value range 0-5.
  -v, --verbose         If set, verbose information output is printed: ccch
                        configuration, cell ARFCN's, neighbour ARFCN's
  -d, --debug           Print additional debug messages
{{< / highlight >}}


And a sample of its output:

{{< highlight sh >}}
[n@mamluk hugo]$ grgsm_scanner -b GSM900

ARFCN:    2, Freq:  935.4M, CID: 55181, LAC:  3804, MCC: 242, MNC:   2, Pwr: -71
ARFCN:    5, Freq:  936.0M, CID:     0, LAC:  3804, MCC: 242, MNC:   2, Pwr: -87
ARFCN:    6, Freq:  936.2M, CID:  3839, LAC:  3804, MCC: 242, MNC:   2, Pwr: -81
ARFCN:   11, Freq:  937.2M, CID: 51133, LAC:  3804, MCC: 242, MNC:   2, Pwr: -46
ARFCN:   14, Freq:  937.8M, CID:     0, LAC:     0, MCC:   0, MNC:   0, Pwr: -89
ARFCN:   17, Freq:  938.4M, CID:  3991, LAC:  3804, MCC: 242, MNC:   2, Pwr: -81
ARFCN:   38, Freq:  942.6M, CID: 51132, LAC:  3804, MCC: 242, MNC:   2, Pwr: -66
ARFCN:   39, Freq:  942.8M, CID: 45011, LAC:  3804, MCC: 242, MNC:   2, Pwr: -79
ARFCN:   41, Freq:  943.2M, CID:     0, LAC:     0, MCC:   0, MNC:   0, Pwr: -85
ARFCN:   43, Freq:  943.6M, CID:     0, LAC:     0, MCC:   0, MNC:   0, Pwr: -83
ARFCN:   45, Freq:  944.0M, CID: 45271, LAC:  3804, MCC: 242, MNC:   2, Pwr: -72
ARFCN:   48, Freq:  944.6M, CID: 51132, LAC:  3804, MCC: 242, MNC:   2, Pwr: -73
ARFCN:   49, Freq:  944.8M, CID:  3332, LAC: 11901, MCC: 242, MNC:   1, Pwr: -82
ARFCN:   52, Freq:  945.4M, CID: 20275, LAC: 11901, MCC: 242, MNC:   1, Pwr: -83
ARFCN:   57, Freq:  946.4M, CID:  3244, LAC: 11901, MCC: 242, MNC:   1, Pwr: -73
ARFCN:   59, Freq:  946.8M, CID:  3332, LAC: 11901, MCC: 242, MNC:   1, Pwr: -71
ARFCN:   61, Freq:  947.2M, CID:  3333, LAC: 11901, MCC: 242, MNC:   1, Pwr: -80
ARFCN:   62, Freq:  947.4M, CID:  3052, LAC: 11901, MCC: 242, MNC:   1, Pwr: -87
ARFCN:   67, Freq:  948.4M, CID: 20087, LAC: 11901, MCC: 242, MNC:   1, Pwr: -73
ARFCN:   71, Freq:  949.2M, CID: 20088, LAC: 11901, MCC: 242, MNC:   1, Pwr: -82
ARFCN:   72, Freq:  949.4M, CID: 20033, LAC: 11901, MCC: 242, MNC:   1, Pwr: -72
ARFCN:  122, Freq:  959.4M, CID:  3243, LAC: 11901, MCC: 242, MNC:   1, Pwr: -51
{{< / highlight >}}


We are getting quite a bit of information here right now, but out focus is the `Freq` and `Pwr` fields-
Since this is just a simple demo i want to single out the one channel with the highest power, namely the one located at `937.2M`.
That is the same as `937.2MHz`.

The last approach is using the GUI version of `grgsm_livemon`. 

In the GUI we can see an graph of where there is signal. Simple adjust the Frequency slider until you can see a stream of data in your terminal of choice!

![grgsm_livemon in action](/LimeSDR/grgsm_livemon_gui.png)

What is happening in the background while grgsm_livemon is running is that its parsing the data stream from the gsm network, 
and slicing it up in UDP packets which it sends out on the localhost port 4729. This is the default ports, 
it can be quite easily changed to whatever is needed. For example when an master-agent setup has been created.

# Parsing out data
I'm still generally new to the GSM protocols. I have been trying to find some neato data structure diagrams like the ones we have for the OSI stack, 
but nothing has really been available..

So instead of rolling my own decoder from scratch i've instead been looking two projects, one which everyone already has been using,
and one which everyone who just wants to capture tracking data fast has been using.

First tool is no other than `Wireshark`. Which ships with a GSMTAP dissector that works perfectly out of the box.
Just start Wireshark and make it listen to for example the Loopback device and you will see data immediately.
 
From where we could make our own tool maybe using something like PyShark and wrap the Wireshark packet engine.
At this point in time i opted for using the tool created by *Oros42* and *petterreinholdtsen*.

You can find it here: https://github.com/Oros42/IMSI-catcher  
Its super easy to use, all you need to do is let `grgsm_livemon` run and start the simple_IMSI-cather.py script.
What is does is parse all the different kinds of packets which livemon passes to it. From there it tries to parse our either the TMSI or IMSI values.
It can from there save the data in a sqlite database for further aggregation of the data.

{{< highlight sh >}}
[n@mamluk IMSI-catcher]$ python simple_IMSI-catcher.py  --help
Usage: simple_IMSI-catcher.py: [options]

Options:
  -h, --help            show this help message and exit
  -a, --alltmsi         Show TMSI who haven't got IMSI (default  : false)
  -i IFACE, --iface=IFACE
                        Interface (default : lo)
  -m IMSI, --imsi=IMSI  IMSI to track (default : None, Example:
                        123456789101112 or "123 45 6789101112")
  -p PORT, --port=PORT  Port (default : 4729)
  -s, --sniff           sniff on interface instead of listening on port
                        (require root/suid access)
  -w SQLITE, --sqlite=SQLITE
                        Save observed IMSI values to specified SQLite file
{{< / highlight >}}


## TMSI? IMSI?
IMSI stands for *International mobile subscriber identity* and TMSI is *Temporary Mobile Subscriber Identity*  
Basically the telecom operators out the same way as the operating system developers that having your device broadcast a 
unique identifier for all to see might infringe on some basic privacy (looking at you MAC addresses).
The telco equivalent of an MAC address is the IMSI number. The IMSI identifies you as an customer of your telco, and as such does not change unless your telco forces it on you.
An solution to this problem is that the local cellular network can assign a (actually two ) TMSI values which the devices can use instead of its IMSI number.
The problem with this is that even tough a device man use the TMSI number it needs to expose its IMSI number wierdly often...

So this is what Oros42's tool does. It simply captures the IMSI, TMSI-1 and TMSI-2 numbers and stores them with an timestamp.


# The result and conclusion

{{< highlight sh >}}

Nb IMSI ; TMSI-1     ; TMSI-2     ; IMSI              ; country      ; brand      ; operator              ; MCC  ; MNC   ; LAC    ; CellId
1       ;            ;            ; 242 14 0XXXXXXXX4 ; Norway       ; ICE        ; ICE Communication Norg; 242  ; 02    ; 3804   ; 51132 
2       ;            ;            ; 242 02 9XXXXXXXX5 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
3       ;            ;            ; 242 14 0XXXXXXXX9 ; Norway       ; ICE        ; ICE Communication Norg; 242  ; 02    ; 3804   ; 51133 
4       ; 0x6f0c93e9 ;            ; 240 08 0XXXXXXXX0 ; Sweden       ; Telenor    ; Telenor Sverige AB    ; 242  ; 02    ; 3804   ; 51133 
5       ;            ;            ; 242 23 2XXXXXXXX0 ; Norway       ; Lycamobile ; Lyca Mobile Ltd       ; 242  ; 02    ; 3804   ; 51133 
6       ;            ;            ; 242 05 9XXXXXXXX4 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
        ; 0x640cd8c7 ;            ; 242 05 9XXXXXXXX4 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
        ; 0xea401e6d ;            ; 242 05 9XXXXXXXX4 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
7       ; 0x220e09ea ; 0x740ca8d9 ; 242 05 9XXXXXXXX0 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
8       ; 0x53045cff ;            ; 242 05 9XXXXXXXX8 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
        ; 0xe340213d ;            ; 242 23 2XXXXXXXX0 ; Norway       ; Lycamobile ; Lyca Mobile Ltd       ; 242  ; 02    ; 3804   ; 51133 
9       ;            ;            ; 242 23 1XXXXXXXX1 ; Norway       ; Lycamobile ; Lyca Mobile Ltd       ; 242  ; 02    ; 3804   ; 51133 
10      ;            ;            ; 242 14 0XXXXXXXX4 ; Norway       ; ICE        ; ICE Communication Norg; 242  ; 02    ; 3804   ; 51133 
11      ;            ;            ; 242 05 9XXXXXXXX9 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
12      ; 0xed384335 ; 0x330dc238 ; 242 23 2XXXXXXXX5 ; Norway       ; Lycamobile ; Lyca Mobile Ltd       ; 242  ; 02    ; 3804   ; 51133 
13      ; 0xfc40db3d ; 0x26060b48 ; 242 05 0XXXXXXXX1 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
14      ; 0xfc40db3d ;            ; 242 02 9XXXXXXXX7 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
15      ;            ;            ; 724 10 0XXXXXXXX4 ; Brazil       ; Vivo       ; Vivo S.A.             ; 242  ; 02    ; 3804   ; 51133 
16      ;            ;            ; 242 23 2XXXXXXXX0 ; Norway       ; Lycamobile ; Lyca Mobile Ltd       ; 242  ; 02    ; 3804   ; 51133 
17      ;            ;            ; 242 02 9XXXXXXXX4 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
18      ;            ;            ; 242 23 2XXXXXXXX1 ; Norway       ; Lycamobile ; Lyca Mobile Ltd       ; 242  ; 02    ; 3804   ; 51133 
19      ; 0xf7409d7d ;            ; 242 05 9XXXXXXXX3 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
20      ;            ;            ; 242 23 1XXXXXXXX6 ; Norway       ; Lycamobile ; Lyca Mobile Ltd       ; 242  ; 02    ; 3804   ; 51133 
21      ; 0xef40e315 ;            ; 242 23 2XXXXXXXX7 ; Norway       ; Lycamobile ; Lyca Mobile Ltd       ; 242  ; 02    ; 3804   ; 51133 
22      ;            ;            ; 242 23 1XXXXXXXX0 ; Norway       ; Lycamobile ; Lyca Mobile Ltd       ; 242  ; 02    ; 3804   ; 51133 
23      ;            ;            ; 242 14 0XXXXXXXX8 ; Norway       ; ICE        ; ICE Communication Norg; 242  ; 02    ; 3804   ; 51133 
24      ;            ;            ; 242 05 9XXXXXXXX7 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
25      ;            ;            ; 242 05 0XXXXXXXX2 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
26      ;            ;            ; 242 05 9XXXXXXXX9 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
27      ;            ;            ; 232 03 1XXXXXXXX0 ; Austria      ; T-Mobile AT; T-Mobile Austria      ; 242  ; 02    ; 3804   ; 51133 
28      ;            ;            ; 242 02 9XXXXXXXX4 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
29      ;            ;            ; 242 05 9XXXXXXXX8 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
        ; 0xf438624d ;            ; 232 03 1XXXXXXXX0 ; Austria      ; T-Mobile AT; T-Mobile Austria      ; 242  ; 02    ; 3804   ; 51133 
        ; 0xd4407c25 ;            ; 232 03 1XXXXXXXX0 ; Austria      ; T-Mobile AT; T-Mobile Austria      ; 242  ; 02    ; 3804   ; 51133 
30      ;            ;            ; 242 02 6XXXXXXXX1 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
31      ;            ;            ; 242 05 9XXXXXXXX8 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
32      ;            ;            ; 242 02 9XXXXXXXX3 ; Norway       ; Telia      ; TeliaSonera Norge AS  ; 242  ; 02    ; 3804   ; 51133 
        ; 0x690d2ac4 ;            ; 242 23 2XXXXXXXX7 ; Norway       ; Lycamobile ; Lyca Mobile Ltd       ; 242  ; 02    ; 3804   ; 51133 
        ; 0xe1381f3d ;            ; 242 23 2XXXXXXXX1 ; Norway       ; Lycamobile ; Lyca Mobile Ltd       ; 242  ; 02    ; 3804   ; 51133 
{{< / highlight >}}


As you can see its rather easily to capture IMSI data from clients. The script which is used can quite easily be extended 
to save more precise timestamps, jump between different channels, and actually anonymize the data!
The capture shown here was over the span of half an hour in a central location in my city.
While looking over these results it dawned on me one of the biggest problems with this kind of solution. 
How to i anonymize the data from everyone including me? I guess i can generate som key in RAM which changes with each boot
of the device i run the software on, but i do not really know if that is enough. I own the device, and as such. How do i make a system does not allow me to find the original?
For example some normal hashing would not suffice in my eyes. Since the search space is so small. Maybe some bcrypt shenanigans with a giant salt would be okay?




# Whats next?

There is a couple things i want to look more into. 
What is possible to do with LTE networks?  
How on earth can i do this while the "victim" remains anonymous?  
How can i make the livemon app parse data from multiple channels at once? Normally an radio might be able to see one channel at the time, but the LimeSDR is absurd and just does its own thing..
The repository got a version of the script one devices per channel, but why use six devices when one can do the same job? :D


# I'm lazy. Can this be scripted?

**YES!**

Since i've been running this setup on a machine i tend to reinstall every couple of weeks i ended up setting up a couple
of Ansible scripts. It worked last time i ran it ¯\\\_(ツ)\_/¯
I pinkyswear that i'll compress the filestructure to only what we need at some point

Fun fact: It's actually the first Ansible roles i've made. Ever.

You can grab it from over here: https://github.com/N0K0/limesdr_IMSI_fedora

Please fix it if it does not work ;D