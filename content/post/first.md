+++
date = "2016-03-05T15:40:59Z"
title = "Raspberry Pi 3 Docker Swarm Cluster"
tags = ["docker","raspberry pi"]
+++



***
Der Raspberry 3 hat ja noch mehr Power als die beiden Vorgänger und eignet sich damit noch besser als Docker Plattform.
Mit mehreren Raspberrys kann man sich für schmales Geld einen schönen Cluster aufbauen.

Ich habe dafür folgendes genommen:

* [3x Raspberry Pi 3 (Pollin)](http://www.pollin.de/shop/dt/OTQxNzkyOTk-/Bausaetze_Module/Entwicklerboards/Raspberry_PI/Raspberry_Pi_3_Modell_B.html)
* [1x 5-fach USB Netzteil (Amazon)](http://www.amazon.de/gp/product/B013OW6YDM/ref=as_li_tl?ie=UTF8&camp=1638&creative=19454&creativeASIN=B013OW6YDM&linkCode=as2&tag=jonasriedelde-21)
* [1x 5-fach Switch (Amazon)](http://www.amazon.de/gp/product/B00N0OHEMA/ref=as_li_tl?ie=UTF8&camp=1638&creative=19454&creativeASIN=B00N0OHEMA&linkCode=as2&tag=jonasriedelde-21)
* [3x Patchkabel 15cm (Amazon)](http://www.amazon.de/gp/product/B00IHIZF8Y/ref=as_li_tl?ie=UTF8&camp=1638&creative=19454&creativeASIN=B00IHIZF8Y&linkCode=as2&tag=jonasriedelde-21)

Dann die "normale" Distribution von der Raspberry Seite herunter laden. Hab die ["Lite" Version von Raspbian](https://www.raspberrypi.org/downloads/raspbian/) genommen, da ich keine grafische Oberfläche brauche und man so mehr Platz hat.

Die auf die SD-Karten installieren und damit booten.

Dann per SSH auf den Dingern einloggen.

So, jetzt die Docker Version für den Raspberry von [Hypriot](http://blog.hypriot.com/downloads/) runterladen, als root installieren, Service für autostart einrichten und dann rebooten:


```bash
wget https://downloads.hypriot.com/docker-hypriot_1.10.2-1_armhf.deb
dpkg -i docker-hypriot_1.10.2-1_armhf.deb
systemctl enable docker.service
reboot
```

Nach dem reboot dann checken ob docker korrekt installiert ist:

```bash
sudo docker version

Client:
 Version:      1.10.2
 API version:  1.22
 Go version:   go1.4.3
 Git commit:   c3959b1
 Built:        Wed Feb 24 09:51:38 2016
 OS/Arch:      linux/arm

Server:
 Version:      1.10.2
 API version:  1.22
 Go version:   go1.4.3
 Git commit:   c3959b1
 Built:        Wed Feb 24 09:51:38 2016
 OS/Arch:      linux/arm

 ```
Dann bocker-machine runterladen (gibts als fertiges Binary bei hypriot)
Umbenennen und z.B. nach /usr/bin kopieren (als root)

```bash
 wget https://downloads.hypriot.com/docker-machine_darwin-amd64_0.4.1
 mv docker-machine_darwin-amd64_0.4.1 docker-machine
 cp docker-machine /usr/bin
```

Damit docker-machine seien Arbeit verrichten kann, muss es einen autologin für root geben. Dazu muss einiges gemacht werden.

Als erstes in der Datei `/etc/ssh/sshd_config` die folgende Zeile anpassen:

```bash
PermitRootLogin yes 
```
Jetzt sollte ein SSH-Login für root möglich sein. Noch einen passenden Key erstellen und auf die Systeme kopieren (auch auf das aktuelle):

```bash
#als root:
#autologin schlüssel anlegen
ssh-keygen
ssh-copy-id root@raspberrypi1
ssh-copy-id root@raspberrypi2
ssh-copy-id root@raspberrypi3
```
Jetzt kann man mit docker-machine von einem Raspberry die alle Raspberrys mit einem Swarm Client versorgen. Das token müsst ihr durch einen entsprechend langen hex-String ersetzen. Es sorgt dafür, das sich die Cluster Member über einen Docker Webservice im Internet finden. Die IP Adressen müsst ihr natürlich entsprechend für euer Netz anpassen. Pi1-Pi3 sind hierbei die Hostnamen der Systeme die Docker gleich neu setzt. Als "driver" für docker-machine wird hier "generic" verwendet, d.h. docker-machine versucht die Provisionierung per SSH vorzunehmen. Dazu eben der autologin für root....

```bash
docker-machine -D create -d generic --swarm  --swarm-discovery token://xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --generic-ip-address 192.168.178.119 pi1
docker-machine -D create -d generic --swarm  --swarm-discovery token://xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --generic-ip-address 192.168.178.118 pi2
docker-machine -D create -d generic --swarm  --swarm-discovery token://xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --generic-ip-address 192.168.178.120 pi3
```

So, wenn jetzt alles funktioniert hat, sollte ein Docker Swarm Agent auf allen Pi's laufen. 

Um das zu checken kann man mal `docker info` als root auf den Kisten aufrufen, sollte irgendwie so ausehen:
```bash
root@pi2:~# docker info
Containers: 9
 Running: 3
 Paused: 0
 Stopped: 6
Images: 3
Server Version: 1.10.2
Storage Driver: overlay
 Backing Filesystem: extfs
Execution Driver: native-0.2
Logging Driver: json-file
Plugins:
 Volume: local
 Network: bridge null host
Kernel Version: 4.1.18-v7+
Operating System: Raspbian GNU/Linux 8 (jessie)
OSType: linux
Architecture: armv7l
CPUs: 4
Total Memory: 925.8 MiB
Name: pi2
ID: T3GD:TEP7:6FZP:YFT3:U7PE:I653:YAKT:MVZU:F5U2:QTX5:ED72:CUSU
WARNING: No memory limit support
WARNING: No swap limit support
WARNING: No oom kill disable support
WARNING: No cpu cfs quota support
WARNING: No cpu cfs period support
Labels:
 provider=generic
 ```

 Soweit so unspektakulär. Um jetzt den Cluster zu *sehen* muss man quasi die passende Umbebungsvariablen setzen damit docker mit dem Cluster spricht. Das kann mit `eval $(docker-machine env --swarm pi3)` erledigt werden (egal welche Host man da angibt). Wenn das erfolgt ist sollte die `docker info` Ausgabe wie folgt aussehen:

 ```bash
root@pi1:~# eval $(docker-machine env --swarm pi3)
root@pi1:~# docker info
Containers: 33
 Running: 8
 Paused: 0
 Stopped: 25
Images: 9
Server Version: swarm/1.1.2
Role: primary
Strategy: spread
Filters: health, port, dependency, affinity, constraint
Nodes: 3
 pi1: 192.168.178.119:2376
  └ Status: Healthy
  └ Containers: 18
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 972.1 MiB
  └ Labels: executiondriver=native-0.2, kernelversion=4.1.18-v7+, operatingsystem=Raspbian GNU/Linux 8 (jessie), provider=generic, storagedriver=overlay
  └ Error: (none)
  └ UpdatedAt: 2016-03-07T21:05:37Z
 pi2: 192.168.178.118:2376
  └ Status: Healthy
  └ Containers: 9
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 972.1 MiB
  └ Labels: executiondriver=native-0.2, kernelversion=4.1.18-v7+, operatingsystem=Raspbian GNU/Linux 8 (jessie), provider=generic, storagedriver=overlay
  └ Error: (none)
  └ UpdatedAt: 2016-03-07T21:05:18Z
 pi3: 192.168.178.120:2376
  └ Status: Healthy
  └ Containers: 6
  └ Reserved CPUs: 0 / 4
  └ Reserved Memory: 0 B / 972.1 MiB
  └ Labels: executiondriver=native-0.2, kernelversion=4.1.18-v7+, operatingsystem=Raspbian GNU/Linux 8 (jessie), provider=generic, storagedriver=overlay
  └ Error: (none)
  └ UpdatedAt: 2016-03-07T21:05:32Z
Plugins:
 Volume:
 Network:
Kernel Version: 4.1.18-v7+
Operating System: linux
Architecture: arm
CPUs: 12
Total Memory: 2.848 GiB
Name: d3de84a6c6b9
```
Da steht schon mal was von `CPUs: 12`, das ist ja schon mal ganz cool, der Cluster läuft also !

Was man jetzt damit anstellen kann folgt in einem anderen Post.

