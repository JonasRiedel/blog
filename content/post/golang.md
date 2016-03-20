+++
css = []
date = "2014-05-24T22:09:01+01:00"
tags = ["go"]
title = "GO - GOLANG"

+++

[Go](http://golang.org/) (oder auch Golang) ist eine "neue" Programmiersprache, die von Google erfunden wurde. Genauer gesagt von Rob Pike und Ken Thomson, die beiden könnte man kennen, da sie auch UTF-8 entwickelt haben.

Wie immer wenn eine neue Programmiersprache entwickelt wird, wurde auch hier versucht, aus den Problemen der aktuellen Sprachen zu lernen.
So wurde Go zur "Programmiersprache des 21. Jahrunderts", mit Garbage Collection, starker Typisierung, Nebenläufigkeit, "Duck typing" und hoher Internet Affinität.

Ein einfaches "Hello World" Programm in Go:

```go
package main
 
import "fmt"
 
func main() {
	fmt.Println("Hello World")
}
```
Hier das ganze nochmal, nur das das Go Programm jetzt ein Webserver ist, der eine Webseite mit "Hello World" zeigt:
```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello World")
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```
Man sieht schon das es sehr einfach ist einen normalen Browser als Frontend für das Go Programm zu benutzen. Bei meinem Programmen lasse ich das Programm auch gerne die Webseiten für das Frontend generieren und gleichzeitig dient es als REST Endpunkt um dynamisch Daten zu liefern.