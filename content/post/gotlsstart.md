+++
date = "2016-03-28T22:22:11+02:00"
tags = ["go", "ssl"]
title = "TLS HTTP SERVER IN GO MIT EIGENEM ZERTIFIKAT"

+++

HTTPS als Server in go zu nutzen ist eigentlich ganz einfach, trotzdem vergesse ich auch gerne welche Schritte dazu notwendig sind. Hier jetzt ein Blog Eintrag als Gedächnisstütze.

Um HTTPS nutzen zu können muss man ein Zertifikat erstellen. Unter Linux ist das schnell erledigt.
Zuerst den private key erzeugen:

```bash
openssl genrsa -out server.key 2048
```

Dann damit das eigentliche Zertifikat:

```bash
openssl req -new -x509 -key server.key -out server.pem -days 3650
```

So, jetzt nur noch einen kleines Program mit dem Webserver:

```go
package main

import (
	"fmt"
	"net/http"
)


func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello world"!)
}

func main() {
	http.HandleFunc("/", handler)
	http.ListenAndServeTLS(":8080", "server.pem", "server.key", nil)
}
```

Unter [https://127.0.0.1:8080/](https://127.0.0.1:8080/) gibts jetzt ein TLS verschlüsseltes "Hello World!"