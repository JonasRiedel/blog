+++
date = "2014-09-15T22:38:41+01:00"
tags = ["beaglebone", "hardware"]
title = "REALTEK RTL8192CU TREIBER FÜR UBUNTU 13.04 AUF BEAGLEBONE BLACK"

+++


Der Beaglebone Black ist ein kleiner Platinencomputer vergleichbar mit dem Raspberry PI, nur mit etwas mehr Power, dafür weniger Multimedia Unterstützung.
Ich wollte da jetzt meinen WLAN Stick von Logilink installieren, aber natürlich funktioniert das nicht out-of-the-box.
Daher hier eine kleine Anleitung wie es geht...

	ubuntu@beaglebone:~$ uname -a
	Linux beaglebone 3.8.13-bone28 #1 SMP Fri Sep 13 03:12:24 UTC 2013 armv7l armv7l armv7l GNU/Linux

**Hier im weiteren auf die passende Version (hier 3.8.13-bone28) achten !!**

	ubuntu@beaglebone:~$ wget http://www.jonasriedel.de/assets/linux-headers-3.8.13-bone28_1.0raring_armhf.deb

	ubuntu@beaglebone:~$ sudo dpkg -i linux-headers-3.8.13-bone28_1.0raring_armhf.deb

	ubuntu@beaglebone:~$ wget https://realtek-8188cus-wireless-drivers-3444749-ubuntu-1304.googlecode.com/files/rtl8192cu-tjp-dkms_1.6_all.deb

	ubuntu@beaglebone:~$ sudo dpkg -i rtl8192cu-tjp-dkms_1.6_all.deb
	# Hier gibt es einen Fehler, das macht aber nichts, da die Sourcefiles dann schon installiert wurden

	ubuntu@beaglebone:~$ sudo ln -s /usr/src/linux-headers-3.8.13-bone28/arch/arm /usr/src/linux-headers-3.8.13-bone28/arch/armv7l

	ubuntu@beaglebone:~$ sudo vi /usr/src/linux-headers-3.8.13-bone28/arch/armv7l/include/asm/timex.h

Hier folgende Zeile ändern:
	#include <mach/timex.h> 
zu 
	#include </usr/src/linux-headers-3.8.13-bone28/arch/arm/include/asm/timex.h>
	ubuntu@beaglebone:~$ cd /usr/src/rtl8192cu-tjp-1.6

	ubuntu@beaglebone:/usr/src/rtl8192cu-tjp-1.6$ sudo make
	
	ubuntu@beaglebone:/usr/src/rtl8192cu-tjp-1.6$ sudo cp 8192cu.ko /	lib/modules/3.8.13-bone28/kernel/drivers/net/wireless/
	
	ubuntu@beaglebone:/usr/src/rtl8192cu-tjp-1.6$ sudo depmod
	
	ubuntu@beaglebone:/usr/src/rtl8192cu-tjp-1.6$ sudo vi /etc/	modprobe.d/blacklist.conf
	Und das hier unten anfügen:
	# Blacklist native RealTek 8188CUs drivers 
	blacklist rtl8192cu 
	blacklist rtl8192c_common 
	blacklist rtlwifi

So, bei mir wurde jetzt nach einem Reboot das Interface gefunden:

	ubuntu@beaglebone:~$ iwconfig
	wlan0     unassociated  Nickname:""
	          Mode:Auto  Frequency=2.412 GHz  Access Point: Not-Associated
	          Sensitivity:0/0
	          Retry:off   RTS thr:off   Fragment thr:off
	          Power Management:off
	          Link Quality=0/100  Signal level=0 dBm  Noise level=0 dBm
	          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
	          Tx excessive retries:0  Invalid misc:0   Missed beacon:0

Jetzt noch das Interface konfigurieren:

	ubuntu@beaglebone:~$ sudo vi /etc/network/interfaces

	# WiFi Example
	auto wlan0
	iface wlan0 inet dhcp
	wpa-ssid "JRXXXXX"
	wpa-psk  "xxxxxxxx"

	ubuntu@beaglebone:~$ sudo ifup wlan0


	ubuntu@beaglebone:~$ ifconfig -a
	...
	wlan0     Link encap:Ethernet  HWaddr 00:9e:99:70:28:78
	          inet addr:192.168.178.60  Bcast:192.168.178.255  Mask:255.255.255.0
	          inet6 addr: fe80::29e:99ff:fe70:2878/64 Scope:Link
	          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
	          RX packets:55 errors:0 dropped:465 overruns:0 frame:0
	          TX packets:9 errors:0 dropped:6 overruns:0 carrier:0
	          collisions:0 txqueuelen:1000
	          RX bytes:53628 (53.6 KB)  TX bytes:5518 (5.5 KB)
So, alles super, läuft.

PS: Ach ja, der Beaglebone Black hat keinen Hot-Plug USB Anschluß, d.h. Wenn man einen WLAN Stick rein gesteckt hat, muss man nochmal booten damit der erkannt wird.