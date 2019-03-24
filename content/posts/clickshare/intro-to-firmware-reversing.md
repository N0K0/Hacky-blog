---
title: "An introduction to some firmware reversing: Part 1"
date: 2018-12-16T09:00:00+01:00
---

# Preface

So last month an mail went out regarding a piece of hardware used in some AV setups. The mail asked if someone had taken a close look at the devices in question where there was only one that could say that he had, but only baraly scratched the surface.
My interest was piqued. 

**The overall goals**

* Static Credentials
* Vulnerabilities
* Other fun stuff


**Note:** I'm staying a bit vauge at the moment since i'm not complete with me hijinks. 

# Finding the firmware

For once it was no issue whatsoever actually fetching the firmware i needed. Just popped by their site and went to the support section and downloaded what i needed.


# Unpacking the firmware

We start of with a zip file `R33050020-1-10-1-8-ClickShareCSC-1BaseUnitFirmware.zip`.

```bash
root@kali:/mnt/hgfs/Workspace/blog# file R33050020-1-10-1-8-ClickShareCSC-1BaseUnitFirmware.zip 
R33050020-1-10-1-8-ClickShareCSC-1BaseUnitFirmware.zip: Zip archive data, at least v2.0 to extract
```

And unpack:
```bash
7z x R33050020-1-10-1-8-ClickShareCSC-1BaseUnitFirmware.zip
[...]

root@kali:/mnt/hgfs/Workspace/blog# ls -la
total 204570
drwxrwxrwx 1 root root      4096 Dec 16 14:14  .
drwxrwxrwx 1 root root     12288 Dec 16 13:56  ..
-rwxrwxrwx 1 root root 203560960 Jun 12  2018  clickshare_baseunit_01.10.01.0008_all.signed_release.ipk
-rwxrwxrwx 1 root root   3543996 Dec 19  2014 'ClickShare mise '$'\302\205'' jour logicielle.pdf'
-rwxrwxrwx 1 root root   2356549 Dec 19  2014 'ClickShare Software Update Instructions.pdf'
-rwxrwxrwx 1 root root      1148 Jun 12  2018  ReleaseNotes.txt

```

The PDFs are instructions for how to update the firmware, nothing too important other than this tidbit:

`Enter the user name “admin” and the password “admin”` 

Good to know!

Lets look at the `ipk` file. Never heard about it before.. Lets check what it might be with binwalk.

```bash
root@kali:/mnt/hgfs/Workspace/blog# binwalk clickshare_baseunit_01.10.01.0008_all.signed_release.ipk 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             POSIX tar archive (GNU), owner user name: ".ipk"

```

And unpacking:

```bash
7z x clickshare?baseunit?01.10.01.0008?all.signed?release.ipk
[...]

root@kali:/mnt/hgfs/Workspace/blog/ipk# ls -la
total 198783
drwxrwxrwx 1 root root         0 Dec 16 14:27 CACerts
-rwxrwxrwx 1 root root 203549388 Jun 12  2018 firmware.ipk
```

Checking out the `CACerts` i could not really figure out anything too important, an hash in something i think is binary format and a pem file most probaly used during verification of new firmware updates. Lets just have it in hte back of our head if we find some update system.

```bash
root@kali:/mnt/hgfs/Workspace/blog/ipk/firmware# ls -l
total 198780
-rwxrwxrwx 1 root root    150762 Jun 12  2018 control.tar.gz
-rwxrwxrwx 1 root root 203398434 Jun 12  2018 data.tar.gz
-rwxrwxrwx 1 root root         4 Jun 12  2018 debian-binary
```


## Control
More unpacking...

```bash
root@kali:/mnt/hgfs/Workspace/blog/ipk/firmware/control# ls -l
total 703
-rwxrwxrwx 1 root root    242 Jan 29  2018 control
-rwxrwxrwx 1 root root 713945 Jan 29  2018 md5sums
-rwxrwxrwx 1 root root   1192 Jan 29  2018 postinst
-rwxrwxrwx 1 root root   2764 Jan 29  2018 preinst
```

Nothing here is all too exciting here, so we will move over to the `data` file instead

## Data
More unpacking...

```bash
root@kali:/mnt/hgfs/Workspace/blog/ipk/firmware/data# ls -l
total 38
drwxrwxrwx 1 root root 8192 Jan 29  2018 bin
drwxrwxrwx 1 root root    0 Jan 29  2018 boot
drwxrwxrwx 1 root root 4096 Jan 29  2018 clickshare
drwxrwxrwx 1 root root    0 Jan 29  2018 dev
drwxrwxrwx 1 root root 8192 Jan 29  2018 etc
drwxrwxrwx 1 root root    0 Jan 29  2018 home
drwxrwxrwx 1 root root 4096 Jan 29  2018 lib
-rwxrwxrwx 1 root root    3 Jan 29  2018 lib64
-rwxrwxrwx 1 root root   11 Jan 29  2018 linuxrc
drwxrwxrwx 1 root root    0 Jan 29  2018 media
drwxrwxrwx 1 root root    0 Jan 29  2018 mnt
drwxrwxrwx 1 root root    0 Jan 29  2018 opt
drwxrwxrwx 1 root root    0 Jan 29  2018 proc
drwxrwxrwx 1 root root    0 Jan 29  2018 root
-rwxrwxrwx 1 root root    3 Jan 29  2018 run
drwxrwxrwx 1 root root 8192 Jan 29  2018 sbin
drwxrwxrwx 1 root root    0 Jan 29  2018 store
drwxrwxrwx 1 root root    0 Jan 29  2018 sys
drwxrwxrwx 1 root root    0 Jan 29  2018 tmp
drwxrwxrwx 1 root root    0 Jan 29  2018 usr
drwxrwxrwx 1 root root 4096 Jan 29  2018 var
drwxrwxrwx 1 root root    0 Jan 29  2018 www

```

Ding ding ding! Found the filesystem!

# Surveying the land

So at this point in time i want to quickly survey the filesystem for information that can be of use. For this i generally use the filebrowser that comes with `Visual Studio Code` and either the search-engine in VSC or [The Silver Searcher (ag)](https://github.com/ggreer/the_silver_searcher)

Lets try something wierd first, lets see if [LinEnum](https://github.com/rebootuser/LinEnum) can get something cool out of this box
Unfortunalty i can't be bothered to setup everything that i need to do for setting up the `chroot` environment needed.

Lets get an overview of the general system:

`find . -iname "*" -not -path "./lib/*" -not -path "./usr/share*" -not -path "./usr/lib*" -exec file -F \| {} \;`

This give us an giant `|` separated list which we can dig trough.

First and foremost we can easily spot that the `/www/restapi/` folder is used for some rest api written in NodeJS and that `/www/pages` seems to contain an php server.

Coming back to our goals we should look for an `/etc/shadow` and `/etc/passwd` unfortunatly only the passwd existed 

```plain
root:x:0:0:root:/root:/bin/sh
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:100:sync:/bin:/bin/sync
mail:x:8:8:mail:/var/spool/mail:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
operator:x:37:37:Operator:/var:/bin/sh
haldaemon:x:68:68:hald:/:/bin/sh
ftp:x:83:83:ftp:/home/ftp:/bin/sh
nobody:x:99:99:nobody:/home:/bin/sh
sshd:x:103:99:Operator:/var:/bin/sh
avahi:x:1000:1000::/:/bin/false
dbus:x:81:81:DBus messagebus user:/var/run/dbus:/bin/false
```

Nothing too important here when we can't fetch the possible passwords...
Before moving on to other files we should check if there is any references to said files in the system.

```
root@kali:/mnt/hgfs/Workspace/blog/ipk/firmware/data# ag -t -i shadow
etc/init.d/S00systemsetup
6:# Restore /etc/shadow
7:/usr/bin/testimage jg / && chmod 600 /etc/shadow

```

For some wierd reason those two things are written on the same line. Need to checkout `/usr/bin/testimage` later 

Moving over to look into what happens when the system starts up, so we move our focus over to `/etc/init.d` and `/etc/inittab`

```shell
root@kali:/mnt/hgfs/Workspace/blog/ipk/firmware/data/etc/init.d# ls -l
total 41
-rwxrwxrwx 1 root root   15 Jan 29  2018 centralstore
-rwxrwxrwx 1 root root  789 Jan 29  2018 csdiscovery
-rwxrwxrwx 1 root root   14 Jan 29  2018 dhcp-server
-rwxrwxrwx 1 root root   14 Jan 29  2018 dongleagent
-rwxrwxrwx 1 root root   10 Jan 29  2018 hostapd
-rwxrwxrwx 1 root root  497 Jan 29  2018 iptables
-rwxrwxrwx 1 root root   13 Jan 29  2018 ledcontrol
-rwxrwxrwx 1 root root   11 Jan 29  2018 lighttpd
-rwxrwxrwx 1 root root  872 Jan 29  2018 projector-control
-rwxrwxrwx 1 root root  423 Jan 29  2018 rcK
-rwxrwxrwx 1 root root  408 Jan 29  2018 rcS
-rwxrwxrwx 1 root root  846 Jan 29  2018 restapi
-rwxrwxrwx 1 root root  540 Jan 29  2018 S00systemsetup
-rwxrwxrwx 1 root root    8 Jan 29  2018 S01iptables
-rwxrwxrwx 1 root root  496 Jan 29  2018 S01logging
-rwxrwxrwx 1 root root 2316 Jan 29  2018 S01rsyslog
-rwxrwxrwx 1 root root  426 Jan 29  2018 S02acpid
-rwxrwxrwx 1 root root  274 Jan 29  2018 S05avahi-setup.sh
-rwxrwxrwx 1 root root 1479 Jan 29  2018 S10udev
-rwxrwxrwx 1 root root  348 Jan 29  2018 S11udev-postactions
-rwxrwxrwx 1 root root 1365 Jan 29  2018 S20urandom
-rwxrwxrwx 1 root root 1769 Jan 29  2018 S30dbus
-rwxrwxrwx 1 root root  281 Jan 29  2018 S40network
-rwxrwxrwx 1 root root  862 Jan 29  2018 S49ntp
-rwxrwxrwx 1 root root  285 Jan 29  2018 S50avahi-daemon
-rwxrwxrwx 1 root root 1257 Jan 29  2018 S50dhcp-server
-rwxrwxrwx 1 root root 1354 Jan 29  2018 S50dropbear
-rwxrwxrwx 1 root root  931 Jan 29  2018 S50hostapd
-rwxrwxrwx 1 root root  752 Jan 29  2018 S50lighttpd
-rwxrwxrwx 1 root root 1092 Jan 29  2018 S80dhcp-relay
-rwxrwxrwx 1 root root  786 Jan 29  2018 S80watchdogkicker
-rwxrwxrwx 1 root root  848 Jan 29  2018 S81ledcontrol
-rwxrwxrwx 1 root root  595 Jan 29  2018 S85alsa-state
-rwxrwxrwx 1 root root 2430 Jan 29  2018 S85centralstore
-rwxrwxrwx 1 root root 1656 Jan 29  2018 S86dongleagent
-rwxrwxrwx 1 root root   17 Jan 29  2018 S86projector-control
-rwxrwxrwx 1 root root 1088 Jan 29  2018 S99cron
-rwxrwxrwx 1 root root  781 Jan 29  2018 scepd
-rwxrwxrwx 1 root root 1853 Jan 29  2018 splashprogress.sh
-rwxrwxrwx 1 root root   10 Jan 29  2018 syslog
-rwxrwxrwx 1 root root   17 Jan 29  2018 watchdogkicker
```

In this list the one thing that really caught my eye was `S50dropbear` and `S50lighttpd`

Dropbear is a tiny SSH server which is used for its tiny footprint.

In the config we got the following snippet:

{{< highlight sh "linenostart=9,linenos=inline" >}}
[...]
start() {
	DROPBEAR_ARGS="$DROPBEAR_ARGS -R"

	# If /etc/dropbear is a symlink to /var/run/dropbear, and
	#   - the filesystem is RO (i.e. we can not rm the symlink),
	#     create the directory pointed to by the symlink.
	#   - the filesystem is RW (i.e. we can rm the symlink),
	#     replace the symlink with an actual directory
	if [ -L /etc/dropbear \
	     -a "$(readlink /etc/dropbear)" = "/var/run/dropbear" ]
	then
		if rm -f /etc/dropbear >/dev/null 2>&1; then
			mkdir -p /etc/dropbear
		else
			echo "No persistent location to store SSH host keys. New keys will be"
			echo "generated at each boot. Are you sure this is what you want to do?"
			mkdir -p "$(readlink /etc/dropbear)"
		fi
	fi

	printf "Starting dropbear sshd: "
	umask 077

	start-stop-daemon -S -q -p /var/run/dropbear.pid \
		--exec /usr/sbin/dropbear -- $DROPBEAR_ARGS
	[ $? = 0 ] && echo "OK" || echo "FAIL"
}
[...]
{{< /highlight >}}

This is bad news unfortunalty... The `-R` from line 10 means that each box may get an unique signarture on each boot unless the `/etc/dropbear/` is mounted (given that my testing with Dropbear is correct)

As for lighthttp:


{{< highlight sh >}}
PATH=/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/sbin/lighttpd
NAME=S50lighttpd
DESC="Lighttpd Web Server"
OPTS="-f /etc/lighttpd.conf"
PIDFILE="/var/run/lighttpd.pid"

UPLOAD_DIR=/tmp/uploads

case "$1" in
  start)
	echo -n "Starting $DESC: "
	mkdir -p $UPLOAD_DIR
	chmod a+w $UPLOAD_DIR
	start-stop-daemon --start -m -p $PIDFILE -x "$DAEMON" -- $OPTS
	echo "$NAME."
	;;
[...]
{{< /highlight >}}


With a config that is more or less comletely default.
From the config we can see that the document root is over at `server.document-root    = "/www/pages/"`

# Summation

* We know we got a dropbear SSH server, and a lighttpd server
* We got some host keys in `/etc/dropbear`
* We know a default default password `admin:admin`