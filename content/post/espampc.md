+++
date = "2015-08-16T20:18:19+01:00"
title = "ESP8266 AM PC TESTEN"
tags = ["esp8266"]
+++

Habe eine ESP8266 (WLAN Chip mit serieller Schnittstelle) und wollte den schon mal am PC betreiben um zu sehen ob alles funktioniert. Der ESP benötigt 3.3V der USB2Serial Adapter (ein CP2102) kann aber auf seinem 3.3V Pin nicht genügend Strom liefern, deswegen habe ich ein LD33 benutzt um die 5V vom USB auf die benötigten 3.3V zu wandeln.

So siehts momentan aus

![](/post/ESP8266Variante1a.png)
 



Ein paar AT Befehle später hatte der ESP eine IP Adresse aus meinem WLAN:

	at 
	OK

	//resetten
	AT+RST 
	OK
	bBJG&SbrVHH7dNFJ$dVHJ
	[Vendor:www.ai-thinker.com Version:0.9.2.4]

	ready

	//passenden mode setzen
	AT+CWMODE=3 
	OK

	//wlans auflisten
	AT+CWLAP 
	+CWLAP:(4,"sid",-83,"9c:c7:xx:xx:xx:xx",7)

	//in gewünschtes wlan einloggen
	AT+CWJAP="sid","passwort" 
	OK

	//aktuelles (also verbundenes) wlan abfragen
	AT+CWJAP? 
	+CWJAP:"sid"

	//status abfragen (2=verbunden mit wlan)
	AT+CIPSTATUS 
	STATUS:2
	OK

	//aktuelle IP abfragen
	AT+CIFSR 192.168.4.1
	192.168.178.73
Hmm.....das war ja einfach....als nächstes werde ich mal ein einfaches Programm auf den ESP laden.

Ach ja, eine Liste der AT Befehle gibt‘s [hier](http://www.pighixxx.com/test/wp-content/uploads/2014/12/ESP8266Ref.pdf)