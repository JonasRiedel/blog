+++
date = "2016-10-08T18:26:56+02:00"
title = "FLUGZEUGPOSITIONEN MIT DVB-T USB DONGLE EMPFANGEN"
tags = ["sdr", "dvbt", "adsb"]
+++

Mit einem einfachen DVB-T USB Stick von ebay, z.B. den [hier](http://www.ebay.de/itm/111879357729) kann man quasi SDR machen.
Welcher Stick das genau ist, ist eigentlich nicht so wichtig, hauptsache er basiert auf dem Chipsatz `RTL2832U`.

Unter Windows geht man wie folgt vor:

Stick per USB anschliessen, dann das Tool von der Seite [rtl1090.com](http://rtl1090.com/) runterladen und starten.

"New Install" wählen und den Anweisungen folgen. 

Im Laufe der Installation starte auch das Tool zadig.exe, mit dem der normale Windows Treiber für den USB Stick durch den "WinUSB" Treiber ersetzt werden muss.

![](/post/zadig.png)

So, wenn das durch ist startet das Program `rtl1090.exe`:

![](/post/rtlexe.png)

Dann das Program [Virtual Radar Server](http://www.virtualradarserver.co.uk/) installieren

In den Options folgendes einstellen:

![](/post/virtualradaroptions.png)

Dann im Browser [http://127.0.0.1:4000/VirtualRadar/](http://127.0.0.1:4000/VirtualRadar/) aufrufen.

Da sind die Flugzeuge:

![](/post/virtualradar.png)


