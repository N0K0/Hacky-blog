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


# Whats next?

There is two things i want to look more into.

1) What can be done with LTE? Somebody need to create something to fiddle with LTE on the LimeSDR
2) Make it so that our Simple IMSI script can listen to multiple channels at once. The Lime can easily listen to four or five channels at once.
So it should be possible to make the 

# I'm lazy. Can this be scripted?

**YES!**

Since i've been running this setup on a machine i tend to reinstall every couple of weeks i ended up setting up a couple
of Ansible scripts. It worked last time i ran it ¯\\\_(ツ)\_/¯

Fun fact: It's actually the first Ansible roles i've made. Ever.

You can grab it from over here: https://github.com/N0K0/limesdr_IMSI_fedora

Please fix it if it does not work ;D