+++
date = "2016-03-13T18:08:33+01:00"
title = "MIT SYSTEMD DOCKER CONTAINER STARTEN"
tags = ["systemd", "docker"]
+++

Auf meinem [Raspberry Pi Docker Cluster](/post/first) habe ich den Service-Discocvery Dienst Consul als Container installiert. Damit das Teil jetzt auch nach einem reboot startet, wird jetzt ein systemd Service eingerichtet der Consul automatisch startet.

Dazu legen wir als root die Datei `/etc/systemd/system/consul.service` an mit folgendem Inhalt:

```bash
[Unit]
Description=Consul
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/docker start consul
ExecStop=/usr/bin/docker stop consul

[Install]
WantedBy=multi-user.target
```

Hier beschreiben wir den Dienst "Consul" und sagen das er nach dem Docker-Service gestartet werden soll. Das wars schon. Natürlich muss die Datei auf alle Cluster Pi's kopiert werden.
Dann noch den Service aktivieren

	systemctl enable  /etc/systemd/system/consul.service

und erstmal den Dienst von Hand starten

	systemctl start consul.service

So, jetzt sollte der Consul Container laufen und auch beim nächsten Systemstart automatisch geladen werden.
War ja einfach.....