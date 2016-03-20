+++
date = "2016-03-11T20:29:05Z"
highlight = true
tags = ["datadog","raspberry pi"]
title = "Datadog für Raspberry Pi" 
+++

 
***
Datadog ist eine Online-Monitoring Webseite. Dort kann man diverse Metriken visualisieren lassen. Im konstenlosen Basispaket sind bis zu 5 Hosts enthalten. Genau richtig für [mein Raspberry Pi Cluster](/post/first/)

Da Datadog kein Binary für den Raspberry bereitstellt, nimmt man am besten die Python Variante. Dazu muss folgendes gemacht werden:

```bash
sudo apt-get install sysstat
```

Dann muss man in seinem Datadog Account bei "Installation Instructions" die Auswahl "From Source" treffen und den gezeigten Befehl in der Console ausführen.
![](/post/datadog.png)
   
Also dann den entsprechenden Befehl ausführen:

```bash
DD_API_KEY=<API_KEY> sh -c "$(curl -L https://raw.githubusercontent.com/DataDog/dd-agent/master/packaging/datadog-agent/source/setup_agent.sh)"
```

Wenn der Agent korrekt gestartet wurde sollte das irgendwann so aussehen:
![](/post/datadogagent.png)

Wenn man jetzt CRTL-C drückt, bricht man den Agent ab, ist ja irgendwie suboptimal....
Also hier ein kleines inti.d Script was den Agent als Service starten kann:

```bash
#!/bin/sh
# /etc/init.d/datadog

### BEGIN INIT INFO
# Provides:          datadog
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Datadog Agent start script
# Description:       This service is used to provide data to datadog
### END INIT INFO


case "$1" in
    start)
        echo "Starting datadog agent"
        nohup /root/.datadog-agent/bin/agent 2>&1 >/var/log/datadog.log &
        ;;
    stop)
        echo "Stopping datadog agent"
        killall agent
        ;;
    *)
        echo "Usage: /etc/init.d/datadog start|stop"
        exit 1
        ;;
esac

exit 0

```
Eventuell müsst ihr den Pfad `/root/.datadog-agent/bin/agent` anpassen, jenachdem wo ihr das Paket runtergeladen habt.
Das Script in die Datei `/etc/init.d/datadog` kopieren, dann wird der Agent beim booten gestartet. Von Hand geht das dann so:

```bash
service datadog start
```

Jetzt kann man in Datadog seine Metriken visualisieren, mein Dashboard für meine 3 Pi's sieht z.B. so aus:
![](/post/datadogdashboard.png)