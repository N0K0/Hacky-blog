---
title: "Oslo City Bikes: A short story about cost–benefit and risk management"
date: 2019-10-23T20:00:00+01:00
---

During this summer there was an sale over at Lab501 for the Proxmark, Chameleon Rev E and some random NFC writer. 
Thought it was about time for me to fetch those devices too. Originally i got the Proxmark for looking closer at the Passive COTAG cards from Simens. 
Those cards has not really been researched all that much, and i wanted to make a go at them. 
Unfortunatly the proxmark forum died a couple of times while a was doing my research. Frustrated as i was a deferred working 
on the COTAG system until i get access to the other part of the system. That is, the readers and such.

![picture of the setup](/bysykkel/setup.jpg)

But while working on this the lock on the [Oslo City Bike](https://oslobysykkel.no/en/) system caught my attention.

My working teory was the following:

1) The locking bolt identifies the bike somehow. Probably in some matter ensuring Authenticity
2) The locking bolt uses some sort of RF based on the lack of pogopins or receptors 

Since my Samsung phone was unable to pick up anything i can rule out normal NFC "high frequency" cards.
Which means its probably some kind of low frequency card of other sort of magic? 

So a took my box fresh Proxmark and made a quick scan of the locking bolt on the front of the bike.

```
[usb] pm3 --> lf search
[=] NOTE: some demods output possible binary          
[=] if it finds something that looks like a tag          
[=] False Positives ARE possible          
[=]           
[=] Checking for known tags...
          
[+] EM410x pattern found          

EM TAG ID      : 7F000807A1           

Possible de-scramble patterns
          
Unique TAG ID  : FE0010E085          
HoneyWell IdentKey {          
DEZ 8          : 00526241          
DEZ 10         : 0000526241          
DEZ 5.5        : 00008.01953          
DEZ 3.5A       : 127.01953          
DEZ 3.5B       : 000.01953          
DEZ 3.5C       : 008.01953          
DEZ 14/IK2     : 00545461372833          
DEZ 15/IK3     : 001090922799237          
DEZ 20/ZK      : 15140000010014000805          
}
Other          : 01953_008_00526241          
Pattern Paxton : 2132559265 [0x7F1C45A1]          
Pattern 1      : 264839 [0x40A87]          
Pattern Sebury : 1953 8 526241  [0x7A1 0x8 0x807A1]          
          
[+] Valid EM410x ID found!
```

Bingo!  
So what the bikes are using for their locks is a type of card referred to as the EM410x. For me this is quite good news.
Since its an low frequency card it generally gets quite a low charge from its reader. An result of this is that the cards do not have the capacity to actually do lame things like *cryptography*.
It is simply a piece of machine readable paper with extra steps.

## Writing your own piece of paper

So now i know how to read the paper, i'd like to use the Proxmark to test the locks, but it does not fit.
And i can't really pull out my laptop and try to shove a random things into the bike racks in such a public place...

What i did instead was to start looking for some alternative, as with everything else gadety the quest started over at Alibaba.
It took me a total of two minutes to find what i needed. Not only did i find writable dongles, i also found a writer for a total of 5 Dollars..

https://www.aliexpress.com/item/32978744044.html

![picture of the setup](/bysykkel/reader.jpg)