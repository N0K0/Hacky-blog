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
1. The locking bolt identifies the bike somehow. Probably in some matter ensuring Authenticity
2. The locking bolt uses some sort of RF based on the lack of pogopins or receptors 

![picture of the bolt](/bysykkel/bolt.jpg)

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
It took me a total of two minutes to find what i needed. Not only did i find writable dongles, 
i also found a writer for a total of 5 Dollars..

https://www.aliexpress.com/item/32978744044.html  

![picture of the setup](/bysykkel/reader.jpg)

This reader/writer could not have been more straight forward if i tried. It's basically a happy meal toy. 
It even got beeping sounds and blinking lights :)
On top of all that the dongle could not have been a better fit for the lock is i tried.
![picture of the dongle](/bysykkel/dongle_crop.jpg)

{{< figure src="/bysykkel/delivered.jpg" alt="Bike delivered screen" width="300px" >}}

And of course here is an garbage video showing the cloner in action.
{{< youtube xEZnKWeSClI >}}

The end result is now that the Bysykkel system thinks the bike is delivered, and i can now run off with it to where ever i feel like.

## Notifying Bysykkel about this weakness

Originally contacted Bysykkel at 3rd of August, at the 20th of august i got the following response. (Translation is left as an exercise for the reader)

### Initial Mail
{{< blockquote >}}
Hei,

Jeg har drevet med litt research relatert til lavfrekens identifikasjonsteknologi og låsesystemer og i den prosessen slo det meg at bysykkel må jo bruke noe for å identifisere sykkelen i stativet.
Svakheten oppdaget er hvilken teknologi man har valgt å bruke til sagt identifikasjon. 
Fra hva jeg ser har man valgt å bruke en RFID tag av typen EM410x.

Dette er en "dum" chip som alt den kan gjøre er å identifisere seg selv med ett ID nummer, ingen form for verifisering at man har den orginale brikken eller kopibeskyttelse.
For å demonstrere har jeg lagt ved en video viser hvor enkelt en RFID verdt fem dollar kan kopiere brikken dere bruker til identifikasjon.

https://www.aliexpress.com/item/32978744044.html

Alt man trenger er å skrive låsen til en liten nok chip og sette den inn i låsen. Se vedlagt bilde.
Hvis mulig anbefaler jeg å bytte ut leserene i stativene med noe som feks støtter DesFire ev2 med AES.

Dette vil gjøre det umulig å kopiere låsen uten å kjenne til en hemmelig kode kun dere har tilgang til. 
Man kan da også ha en hemmelig kode per sykkel som er tilnærmet umulig å knekke.
Vet dette er potensielt umulig, lurer bare på om dere er kjent med risikoen knyttet til teknologien man har valgt å bruke.
Bare å komme med spørsmål hvis der lurer på noe rundt teknologien eller lignende!

Per nå holdes dette konfidensielt, men har ett håp om å kunne publisere researchen min på ett eller annet tidspunkt. Om en tre månders tid, vil selvfølgelig utsette hvis man går inn for å løse de tekniske utfordringene.
{{< /blockquote >}}


### The first response from Urban Sharing AS
{{< blockquote >}}
Vi i Urban Sharing AS leverer teknologien Oslo Bysykkel bruker. 

Vi beklager at vi svarer så seint!

Det stemmer at RFID-tagene som identifiserer syklene i Oslo bysykkel ikke har noen form for challenge-response-protokoll 
eller asymmetrisk kryptografi, og at det i disse dager er trivielt å få tak i utstyr som kan kopiere dem (eller generere nye tags med samme ID som de eksisterende).

Valget av RFID-type var i hovedsak et kost-nytte-spørsmål. Da systemet som er i bruk i Oslo dag ble utviklet og produsert for snart 20 år siden, var andre typer enn passive, ID-baserte RFID-tags vesentlig dyrere og mindre tilgjengelig enn i dag. 
Dette må også ses i sammenheng med den potensielle risikoen; hva er angrepsvektoren for å utnytte svakheten i RFID-tagene? 
Dersom du kopierer RFID-tagen til en sykkel du har tatt ut og bruker kopien til å lure stasjonen til å tro at du har parkert den mens du egentlig stikker av med den, vil vi oppdage at det mangler en sykkel i denne stasjonen og i siste instans fakturere deg for det.
Å levere en fysisk tjeneste ute i bybildet vil alltid innebære en viss risiko. Tjenester som deles med andre som er synlige i bybildet vil alltid være utsatt for menneskers etiske og moralske kompass. 
Hvis formålet er å bryte med det som kan anses å være etisk riktig vil det alltids finnes mange ulike veier til det målet. (Obligatorisk xkcd: https://xkcd.com/538/)
Du må selvsagt gjerne publisere funnene dine, og still gjerne spørsmål dersom det er noe vi kan bistå med eller det er noe annet du lurer på. Bruk helst security@urbansharing.com for sikkerhetsrelaterte saker.
{{< /blockquote >}}

So the response is quite straight forward, not really all that satisfying. The main takeaway is that they know that the technology of choice is about 20 years old, and insecure.
But for them its really a question of cost-benefit. Which is not completely unreasonable except for the fact that a DesFire chip does not incur anything even close to an significant cost compared to the rest of the bike.
Sure there will be an bigger cost upfront for development of the software, but even that would be insignificant compared to the cost of a couple of bikes.

The thing that is really unsettling about their response is that they a stating that if someone where to trick a lock to belive a bike was delivered they will simply send the bill to the last user.
There is a big problem with this response is that the end user got a receipt stating the bike is delivered and locked. And have been numerous instances where i have found bikes not properly seated in the rack and can simply be removed without unlocking.
Can you still then bill the last user that has proof of the bike being delivered? And what if some third party find a way to shim a lock open without obvious proof of tampering? 


### Me following up on the issues i see with their answer
{{< blockquote >}}
Hei! Takk for en skikkelig respons!

> Valget av RFID-type var i hovedsak et kost-nytte-spørsmål. Da systemet som er i bruk i Oslo dag ble utviklet og produsert for snart 20 år siden, var andre typer enn passive, ID-baserte RFID-tags vesentlig dyrere og mindre tilgjengelig enn i dag

Dette er da ikke ett grunnlag for å velge usikker teknologi når man rullet bysykkel ut i Oslo rundt 2015? Hvis dere har noen gamle systemer i andre byer er jo ikke det en grunn for å rulle ut det samme i Oslo. Usikker på hvilken andre steder eller hvor lenge Urban Sharing har levert tjenesten sin skal bemerkes.
Dersom du kopierer RFID-tagen til en sykkel du har tatt ut og bruker kopien til å lure stasjonen til å tro at du har parkert den mens du egentlig stikker av med den, vil vi oppdage at det mangler en sykkel i denne stasjonen og i siste instans fakturere deg for det.

Helt med på tankegangen. Skal ikke påberope med å kunne juss, men samtidig har sluttbruker blitt tildelt en kvittering/bevis på at sykkelen var parkert. For å putte det litt på spissen kan ikke Urban Sharing fakturere forrige bruker hvis noen var til å fysisk stjele en sykkel fra ett stativ. Selvfølgelig holder ikke dette hvis man mangler typ 15 sykler og sluttbruker står fast på at alle var tatt ut og levert i løpet av 15 minutter. Jeg tenker på scenarioet hvor noen ondsinnede dukker opp med en varebil på ett litt avsidesliggende stativ og bare napper med seg alt.
De trenger bare å finne en måte å lage en anonym bruker. Anonyment telefonnummer er lett, betalingskort/Gavekort er verre, men langt ifra umulig.

Bare noen andre tanker også når jeg skrev dette.
Hvordan vet stativene deres om en plass er opptatt eller ikke?
Kunne man potensielt få appen til å tro ett stativ er fullt når det egentlig er tomt? Alternativt kan man tilfeldig spamme systemet med sykkel UIDer bare for bøll.

-Nikolas
{{< /blockquote >}}


### The final response from them
{{< blockquote >}}
Hei igjen!
> Dette er da ikke ett grunnlag for å velge usikker teknologi når man rullet bysykkel ut i Oslo rundt 2015? 
Den fysiske infrastrukturen ble i sin tid levert av selskapet Sharebike. 
Da konstellasjonen som ble opprettet for å by på kommunens anbud for bysykkelordning i 2015 ble opprettet, var premisset å videreføre den eksisterende infrastrukturen.
I andre byer leverer vi for øvrig et helt annet type system, hvor all logikken er flyttet fra stasjonene til syklene.

> Jeg tenker på scenarioet hvor noen ondsinnede dukker opp med en varebil på ett litt avsidesliggende stativ og bare napper med seg alt. 

Igjen, dersom målet ditt er å stjele bysykler, finnes det mange veier til det målet. Vi leverer delt infrastruktur, og vi er i store trekk avhengige av at folk tar vare på den.

> De trenger bare å finne en måte å lage en anonym bruker. Anonyment telefonnummer er lett, betalingskort/Gavekort er verre, men langt ifra umulig.

Om du først kan lage anonyme telefonnumre og bruke ikke-sporbare betalingskort trenger du ikke å forfalske RFID-tags for å stjele sykler. (-:

> Hvordan vet stativene deres om en plass er opptatt eller ikke?

Det er flere sensorer i stasjonene.

> Kunne man potensielt få appen til å tro ett stativ er fullt når det egentlig er tomt?

I teorien kan du det.

> Alternativt kan man tilfeldig spamme systemet med sykkel UIDer bare for bøll.

Systemet vet hvilke IDer som er gyldige.

Vennlig hilsen
-Berge
{{< /blockquote >}}

Well Berge did not really seem to feel like answering anymore questions. Just giving non-answers for the rest of the questions posed. So i simply let him be.

### And finally a signoff
{{< blockquote >}}
Tusen takk for svar! Takk for at dere er såpass åpne om det hele :)
Lurer ikke egentlig på noe mer!

Sent from ProtonMail mobile
{{< /blockquote >}}

## Rounding off

Urban Infrastructure know that the underlying technology for identifying the bikes are not able to guarantee any sort of authenticity. 
I'm not sure if they simply inherited the racks from ShareBike, or if they bought it from them. I never got around to actually ask them.....
Personally i think its sort of brain dead that we live in a time where we go for the tags that is mainly used for books, packages and long range scannables for something that should work on a basis of authenticity.
Even if someone were to not actually steal the bikes this would open up for the possibility of going over the normal rental time of one hour. Which could lead to worsen the shortage of bikes outside of Oslo central.

As for the possibility of DoSing the racks, i have not researched how the racks reacts to invalid UIDs, multiple UIDs in the same lock or the same UID on multiple locks.
Would it be possible to fill a rack with spoofed bikes so that users will seek an empty rack looking for bikes?
Or maybe the inverted case that users will take a detour since they believed that the rack closest to their destination is full.