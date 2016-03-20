+++
date = "2015-09-15T21:37:23+01:00"
highlight = true
tags = ["go"]
title = "CROSSCOMPILING MIT GO"

+++

Mit go 1.5 ist das cross compilieren nochmal deutlich einfacher geworden, zumindest wenn kein cgo im Spiel ist. Aber auch sonst ist es nicht kompliziert.

## Bootstrapping
Wenn man doch mal die go Toolchain bootstrappen muss geht das unter Windows wie folgt:

## Schritt-für-Schritt-Anleitung
Gcc für Windows runterladen z.B. MinGW: hier https://sourceforge.net/projects/mingw/files/latest/download

Gcc über GUI installieren

Path für c:\MinGW\bin setzen

Nach c:\Go\src wechseln

Folgendes ausführen (Erzeugt Bootstrap für Linux 32 bit):

```bash
	env CGO_ENABLED=1 GOROOT_BOOTSTRAP=c:\\Go GOOS=linux GOARCH=386 make.bat --no-clean
```
Folgendes ausfürhren (Erzeugt Bootstrap für Linux 64 bit):

```bash
	env CGO_ENABLED=1 GOROOT_BOOTSTRAP=c:\\Go GOOS=linux GOARCH=amd64 make.bat --no-clean
```
Jetzt kann man wie folgt cross compilen:

**Ziel: Linux 32bit**

	GOOS=linux GOARCH=386 go build

Ergebnis:

	$ file goCsv2Xlsx
	goCsv2Xlsx32: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), statically linked, not stripped

**Ziel: Linux64bit**

	GOOS=linux GOARCH=amd64 go build

Ergebnis:

	$ file goCsv2Xlsx
	goCsv2Xlsx: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped

Generell werden folgende Zielarchitekturen unterstützt:

|$GOOS|$GOARCH|
--- | --- 
|darwin|386|
|darwin|amd64|
|darwin|arm|
|darwin|arm64|
|dragonfly|amd64|
|freebsd|386|
|freebsd|amd64|
|freebsd|arm|
|linux|386|
|linux|amd64|
|linux|arm|
|linux|arm64|
|linux|ppc64|
|linux|ppc64le|
|netbsd|386|
|netbsd|amd64|
|netbsd|arm|
|openbsd|386|
|openbsd|amd64|
|openbsd|arm|
|plan9|386|
|plan9|amd64|
|solaris|amd64|
|windows|386|
|windows|amd64|

