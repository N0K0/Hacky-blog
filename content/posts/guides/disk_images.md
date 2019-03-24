---
title: "Dealing with disk images"
date: 2019-03-24T00:49:39+01:00
---

Every now and then a want to poke around with an machine without actually having to run the system.
Say the system is infected with some malware or it want to ping back home. 
Generally things you do not want to happen..

## Creating an image

Generally the easiest way to image a disk is simply to run `dd`.  
Its advisable to use an write-blocker if possible to avoid writing anything to the drive.
For example: ```dd if=/dev/sdb/ of=/tmp/disk_image bs=512 status=progress```
Note that `status=progress` was added in GNU Coreutils 8.24.  
This will give us an complete disk image containing everything as one giant binary blob.


## Fixing the partitions

Alright! So we have our binary data. Unfortunatly you can't mount this disk image as is. The start of the image contains
quite a few things that is not excpected from an normal partition.

This is where the wonderful `sleuthkit` comes in!

Its man pages tells us all we need to know
> **mmls** - Display the partition layout of a volume system  (partition tables)

And an small example of the output:
```
mmls -t dos disk.dd
DOS Partition Table
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
00:  Meta    0000000000   0000000000   0000000001   Primary Table (#0)
01:  -----   0000000000   0000000062   0000000063   Unallocated
02:  00:00   0000000063   0002056319   0002056257   Win95 FAT32 (0x0B)
03:  00:01   0002056320   0008209214   0006152895   OpenBSD (0xA6)
04:  00:02   0008209215   0019999727   0011790513   FreeBSD (0xA5)
```
Here we can see where in an image each partition is located.
From here we can either do an other run with `dd` or use an other tool named `mmcat`

### With dd:
Lets say you want the partition for Win95
```
02:  00:00   0000000063   0002056319   0002056257   Win95 FAT32 (0x0B)
```

All you need to do is the following:

```
dd bs=512 skip=0000000063 count=0002056257 if=/tmp/disk_image of=/tmp/win_image
```


### With mmcat:

> **mmcat** - Output the contents of a partition to stdout

```
mmcat /tmp/disk_image 02 > /tmp/win_image
```

## Mounting the partitions

**For ntfs:**
fetch yourself the NTFS-3G package and mount he image as your normally would

```
mkdir /mnt/windows
sudo mount -o ro /tmp/win_image /mnt/windows
```


