# Smartmeter Installation

## Einleitung

Die hier beschriebene Applikation ermöglicht Kunden im Versorgungsgebiet der EVN ihren **aktuellen** Stromverbrauch aus dem Smartmeter auszulesen.
Aufbauend auf der hervorragenden Arbeit von [Michael Reitbauer](https://www.michaelreitbauer.at/kaifa-ma309-auslesen-smart-meter-evn/) habe ich dafür eine Installationsanleitung für den Rasperry Pi basierend auf [Docker](https://www.docker.com/) erstellt. Die Docker-Lösung wurde gewählt, da damit eine sichere Migration zwischen verschiedenen Rechnern möglich ist.

Ich empfehle als Betriebssystem [DietPi](https://dietpi.com) da es wesentlich ressourcensparender ist. Siehe: [DietPi vs Raspberry Pi OS Lite](https://dietpi.com/stats.html)

[DietPi](https://dietpi.com) hat auch den Vorteil, installationsfertige Images für eine große Anzahl von [Alternativen zum RasPi4](https://dietpi.com/#downloadinfo) anbieten zu können. So z.B. der *Odroid C4/HC4*, der zu einem Peis von wesentlich unter € 100,-- lieferbar ist.

Sollte jedoch **Smartmeter** auf ein bereits vorhandenes laufendes System aufgesetzt werden, lese zuerst den Absatz [Smartmeter Installation ohne DietPi](#smartmeter-installation-ohne-dietpi) und fahre dann mit Absatz [Docker](#docker) fort.

# DietPi

[DietPi](https://de.wikipedia.org/wiki/DietPi) ist eine auf Debian basierende Linux-Distribution, die für Single-Board Computer, wie Raspberry Pi, ODROID, ROCK Pi, PINE64, NanoPi und ASUS Tinker Board entwickelt wurde. DietPi unterstützt ebenso PCs und Virtuelle Maschinen. Das Projekt besitzt eine aktive Community und erzeugte über 200 Releases. Aktuell gibt es weltweit über ca. 120 000 laufende DietPi-Systeme.

Meine erste Installation erfolgte auf einem Pi4. Da ich eine Reihe von Pi1 noch in der Sammlung hatte, war mein Bestreben **Smartmeter** auch auf einem leistungsschwächeren System auszuprobieren. Dies stellte sich jedoch auch mit DietPi als **nicht praktikabel** heraus. Für die vorliegende Konfiguration auf **Docker** ist mindestens ein **Pi3** nötig.

## Image erstellen

- 64 Bit

  Aktuelle Images zum Download auf <https://dietpi.com/>

  Menü: Downloads / Rasperry Pi

  Rasperry Pi3 und Pi4 / 64bit -> <https://dietpi.com/downloads/images/DietPi_RPi-ARMv8-Bullseye.7z>

- Flashen des Image

  Benutze zum Flashen der SD-Karte das Programm [balenaEtcher](https://www.balena.io/etcher/)

## Basis-Installation

### Manuelle Installation (nicht empfohlen)

- Setze SD-Karte in Raspi ein und schließe das Netzteil an um zu starten

- Folge den Anweisungen am Bildschirm

- Zur manuellen Installation von DietPi befolge bitte die Schritte der Anleitung in <https://dietpi.com/docs/install/>

### Automatische Basis-Installation

- DietPi bietet eine [automatische Basis-Installation](https://dietpi.com/docs/usage/#how-to-do-an-automatic-base-installation-at-first-boot-dietpi-automation) per Script an. Das für *Smartmeter-Docker* bereits angepasste Script ist hier zum [Download](https://github.com/hansratzinger/smartmeter-docker/blob/main/dietpi.txt)

    **Alle für Docker notwendigen Programme werden durch das Script automatisch installiert!**

- Lade das Script in einen Editor und ändere folgende Zeilen:

  - Zeile 35 Setze hier die IP-Adresse deines Raspi ein

      ```
      AUTO_SETUP_NET_STATIC_IP=10.0.0.10
      ```

  - Zeile 37 Setze hier die IP-Adresse deines Internet-Gateways ein

      ```
      AUTO_SETUP_NET_STATIC_GATEWAY=194.162.0.248
      ```

  - Zeile 85 Setze hier den SSH_PUBKEY deines PC ein um ohne Passwort auf den Raspi sicher zugreifen zu können: C:\Users\deinUser\ssh\id_rsa.pub

      ```
      AUTO_SETUP_SSH_PUBKEY=ssh-ed25519 AAAAAAAA111111111111BBBBBBBBBBBB222222222222cccccccccccc333333333333 mySSHke
      ```

  - Zeile 124 Setze hier dein globales Passwort ein

      ```
      AUTO_SETUP_GLOBAL_PASSWORD=dein-globales-Passwort
      ```

      Du kannst mit den übrigen Einstellungen dieser Dateiversion starten. Mittels dietpi-config lassen sich alle Einstellungen auch später ändern.

- Speichere das File dietpi.txt im Verzeichnis /boot/dietpi.txt der **SD-Karte**
  
- Setze SD-Karte in Raspi ein und schließe folgende Komponenten an um zu starten:

  - Netzteil
  - Bildschirm
  - Tastatur
  - Netzwerkkabel

- Warte einige Minuten. Es werden Updates durchgeführt.
- Folge den Anweisungen am Bildschirm
- Das System startet im *root* User
- Teste das Einloggen über SSH:
  - Öffne *Terminal* oder *Power Shell* am PC und gib die von dir im *personal key* festgelegte Passphrase ein.

      ```
      PS C:\Users\deinUser> ssh dietpi@DEINE.IP.ADRESSE
      Enter passphrase for key 'C:\Users\deinUser/.ssh/id_rsa':
      ```

- Nach dem **erfolgreichen** einloggen über SSH kann der Raspi *headless* betrieben werden. Bildschirm und Tastatur werden nicht mehr benötigt und können entfernt werden.

### Tipp: Einschalten von farbiger Anzeige von "ls" im Terminal

```
nano ~/.bashrc
```

Auskommentiere folgende Zeilen (Entferne das #)

```
# You may uncomment the following lines if you want `ls' to be colorized:
export LS_OPTIONS='--color=auto'
eval "$(dircolors)"
alias ls='ls $LS_OPTIONS'
alias ll='ls $LS_OPTIONS -l'
alias l='ls $LS_OPTIONS -lA'
````

### Tipp: Sicherheitsabfrage bei relevanten Systemcommandos

Es empfiehlt sich die darunter befindlichen Zeilen ebenfalls durch entfernen des # zu aktivieren. Dadurch werden die Befehle rm (Löschen), cp (Kopieren) und mv (Umbenennen) durch die Bildung eines Alias mit -i durch eine Rückfrage vor dem Ausführen gesichert.

````
# Some more alias to avoid making mistakes:
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
````

Speichere die Datei mit *Strg+O* und schließe mit *Strg+X* und anschließend Raspi neu booten:

```
dietpi@Smartmeter:~$ sudo reboot
```

Danach wird das Listing von "ls" farbig dargestellt. Verzeichnisse z.B. sind blau. Die Eingabe von "l" liefert den Output von ls -lA. Die Befehle remove (rm), copy (cp) und move (mv) werden erst nach Rückfrage ausgeführt.

**Hinweis:** Die Setzung der Alias muss für jeden User einzeln vorgenommen werden! (dietpi, root)

# Docker  

**Hinweis:** Sollte Smartmeter nicht auf DietPi aufgesetzt werden, bitte zuerst weiter unten das Kapitel: *Smartmeter Installation ohne DietPi* lesen!

Durch die nun vorgenommene Installation von *DietPi* sind die notwendigen Programme für Docker bereits installiert.

- ### Smartmeter-Repository von [Git-Hub](https://github.com/hansratzinger/smartmeter-docker/settings) klonen

  In Dietpi einloggen:

  Du befindest dich jetzt im Verzeichnis /home/dietpi

  ```
  dietpi@Smartmeter:~$ git clone https://github.com/hansratzinger/smartmeter-docker.git

  Klone nach 'smartmeter-docker' ...
  remote: Enumerating objects: 18, done.
  remote: Counting objects: 100% (18/18), done.
  remote: Compressing objects: 100% (14/14), done.
  remote: Total 18 (delta 4), reused 7 (delta 3), pack-reused 0
  Empfange Objekte: 100% (18/18), 29.24 KiB | 379.00 KiB/s, fertig.
  Löse Unterschiede auf: 100% (4/4), fertig.
  ```

  Wechsle nun in das neu angelegte Verzeichnis

  ````
  dietpi@Smartmeter:~$ cd smartmeter-docker
  ````

## Docker-compose

docker-compose ist ein Automatisierungstool von Docker das die Bedienung wesentlich erleichtert. Gesteuert wird es durch die Datei *docker-compose.yaml*.

**Smartmeter** ist in der *docker-compose.yaml* konfiguriert. Mittels *docker-compose* werden die jeweiligen Docker-Container gebaut und gestartet. Danach ist **Smartmeter** in Betrieb. Die gesammelten Daten werden in den Docker-Volumes persistiert (dauerhaft gespeichert).

- ### Anpassung der *docker-compose.yaml*

  Lade *docker-compose.yaml* in Nano und ändere die Zugangsberechtigungen für die Infux-Datenbank.

  ```
  dietpi@Smartmeter:~/smartmeter-docker$ nano docker-compose.yaml
    ...
    influxdb:
      image: influxdb:1.8
      container_name: influxdb
      restart: unless-stopped
      environment:
        INFLUXDB_DB: smartmeter
        INFLUXDB_HTTP_AUTH_ENABLED: "true"
        INFLUXDB_ADMIN_USER: Admin
        INFLUXDB_ADMIN_PASSWORD: Admin-Passwort
        INFLUXDB_USER: User
        INFLUXDB_USER_PASSWORD: User-Passwort
    ...
  ```

  Passe die Zugangsberechtigungen für die MariaDB / MySql Datenbank an. Dieser Server wird von *Smartmeter* nicht benötigt, kann jedoch für Erweiterungen verwendet werden. Wenn du dieses Service nicht benötigst, lösche die entsprechenden Zeilen aus der docker-compose.yaml.

  ````
    mariadb:
    image: mariadb:latest
    container_name: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: Root-Passwort
      MYSQL_DATABASE: DB-Name
      MYSQL_USER: User
      MYSQL_PASSWORD: User-Passwort
    restart: unless-stopped
    tty: true
    ports:
      - "3306:3306"
    volumes:
      - mariadb-data:/mnt/usb/volumes/mariadb-data/_data
    networks:
      - edge
  ````

  Speichere die Datei mit Strg+O und schließe mit Strg+X

- ### Anpassung des Python-Programmes

  Wechsle nun in das Python-Verzeichnis und passe das Programm mit Nano an:

  ````
  dietpi@Smartmeter:~/smartmeter-docker$ cd python

  dietpi@HeSmartmeter:~/smartmeter-docker/python$ nano EvnSmartmeterMQTTKaifaMA309.py
  ````

  Deinen **EVN-Schlüssel** bekommst du auf Anfrage an
  *<smartmeter@netz-noe.at>*
  zB. "36C66639E48A8CA4D6BC8B282A793BBB"

  ````
  evn_schluessel = "hier EVN-Schlüssel eingeben"
  ````

  MQTT Broker IP adresse Eingeben ohne Port! z.B: "10.0.0.10"

  ````
  mqttBroker = "hier IP-Adresse eingeben"
  ````

  Speichere die Datei mit *Strg+O* und schließe mit *Strg+X*
  
- ### Docker-Container generieren und starten

  in Verzeichnis wechseln

  ```
  cd /home/dietpi/smartmeter-docker
  ```

  docker-compose starten

  - Container werden gemäß der docker-compose.yaml neu gebaut (vorhandene werden NICHT gelöscht)
  - Container werden gestartet

  ```
  dietpi@Smartmeter:~/smartmeter$ sudo docker-compose up -d
  ```

  Tipp: ohne -d wird Logging in die Console geschrieben

  Anschließend baut Docker die Container und startet sie.

  Falls die docker-compose.yaml geändert werden muss und die Container neu gebildet werden sollen, muss erst docker gestopt werden:

  ```
  sudo docker-compose down
  ```

## Speicherplatz der Dockervolumes

Alle Dateien von smartmeter-docker befinden sich außerhalb der Docker-Container hier:
 /mnt/dietpi_userdata/docker-data/volumes

Bei Änderungen in der docker-compose.yaml und wiederholtem Neustart von docker-compose werden neue Container geschrieben, aber die Daten bleiben davon unberührt.

## Portainer

Portainer ist ein nützliches Verwaltungstool für Docker. In DietPi ist es bereits durch die Konfigurationsdatei dietpi.txt automatisch installiert.

URL: <http://10.0.0.10:9002> bzw. IP des jeweiligen Rechners

Menü links: Container - zeigt alle Container und ihren jeweiligen Status an

## Influxdb Datenbank

[Wikipedia](https://de.wikipedia.org/wiki/InfluxDB)

Alle 5 Sekunden werden die Daten von Python-Programm ausgelesen und in der Influx-Datenbank gespeichert.

### Influx commandline tool in Docker aufrufen

1. Laufende Container anzeigen mit *docker ps*

```
dietpi@Smartmeter:~/smartmeter$ docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                                                                                                                             NAMES
290c24e84b86   influxdb:1.8             "/entrypoint.sh infl…"   12 minutes ago   Up 12 minutes   0.0.0.0:8082->8082/tcp, :::8082->8082/tcp, 0.0.0.0:8086->8086/tcp, :::8086->8086/tcp, 0.0.0.0:8089->8089/tcp, :::8089->8089/tcp   influxdb
08b73d393d9a   portainer/portainer-ce   "/portainer"             4 weeks ago      Up 2 days       8000/tcp, 9443/tcp, 0.0.0.0:9002->9000/tcp, :::9002->9000/tcp                                                                     portainer
```

2. Den bereits laufenden Dockercontainer unter seinem Namen *influxdb* aufrufen und auf die Shell aufrufen (/bin/sh). Damit gelangt man auf die Befehlsebene des Dockercontainers. Die angezeigte Nummer ist die ID des Containers. Mit ls kann man alle Verzeichnisse innerhalb des Containers anzeigen:

```
dietpi@Smartmeter:~/smartmeter$ docker exec -it influxdb bash

root@5ad42861c09f:/# ls
bin  boot  dev  entrypoint.sh  etc  home  init-influxdb.sh  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@5ad42861c09f:/# 
````

3. Influx starten. Der Influx-Prompt ist >

````
root@5ad42861c09f:/# influx

Connected to http://localhost:8086 version 1.8.10

InfluxDB shell version: 1.8.10

> 
````

4. Anmeldung, User und Passwort wie in docker-compose.yaml angegeben:

```
> auth

username: admin
password:

````

5. Influx-Datenbanken anzeigen:

````
> show databases

name: databases
name
----
smartmeter
_internal
> 
````

6. Datenbank auswählen:

````
> use smartmeter
Using database smartmeter
````

7. Measurements

Measurements sind in Influxdb das Äquivalent zu den Tabellen in einer SQL-Datenbank. Sie sind die erste Selektionsebene.
In der Smartmeter-DB sind die Measurements z.B.:

- Wirkenergie
- MomentanleistungP
- Spannung L1
- Spannung L2
- Spannung L3

usw.

## Node Red - Weiterleitung der Daten

[Wikipedia](https://de.wikipedia.org/wiki/Node-RED)

Mit Node Red können die vom Python-Programm ausgelesenen und per MQTT gesendeten Daten an die Influx-Datenbank weitergeleitet werden.

### Einrichtung von Node-RED

in Browser aufrufen URL: <http://IP-Deines-Raspi:1880>

Port: siehe docker-compose.yaml 1880

### Influx Datenbank verbinden

- Warnungsfenster beachten, auf Fenster 2x klicken überprüfen ob Volumes persistiert wurden!

- deploy (roter Button drücken)
- Burger Menü / Palette verwalten / Installation / Module durchsuchen
- node-red-contrib-influxdb installieren
- rechtes Menü / Node filtern / influx
- influx-in auf Arbeitsfläche ziehen
  - Doppelklick -> Eigenschaften-Fenster öffnet sich
  - Name: Smartmeter (ist nur Info)
  - Server (Bleistift)
    - Eigenschaften-Tab
      - Name: Smartmeter
      - Version: 1.x
      - Host:  influxdb-smartmeter (Container-Name in docker-compose.yaml)
      - User und Passwort wie in docker-compose.yaml angegeben
  - "Hinzufügen-Button" klicken

- Test: unter Eigenschaften / Query eintragen: SHOW SERIES

Tipp: mit STRG + auf die Arbeitsfläche klicken öffnet ein Menü

- inject auswählen und auf Arbeitsfläche ziehen
- debug auswählen und auf Arbeitsfläche ziehen

### Tutorial

Auf dem Blog von Michael Reitbauer findes sich ein sehr informatives Tutorial dazu: [MQTT Nachrichten in Datenbank speichern](https://www.michaelreitbauer.at/mqtt-nachrichten-in-datenbank-speichern/)

## Grafana - Anzeige des Stromverbrauchs

Mit Grafana kann ein Dashboard zur Visualisierung der Stromdaten gestaltet werden.
Dank an Michael Reitbauer für das informative [Grafana-Tutorial](https://www.michaelreitbauer.at/smart-meter-dashboard-in-grafana-influxdb/).

## Smartmeter Installation ohne DietPi

Docker Installation und Version überprüfen:

```
root@Smartmeter:~# docker --version
Docker version 23.0.2, build 569dd73
```

## Docker Installation

Sollte Docker noch nicht installiert sein:

```
apt update && apt upgrade

curl -fsSL https://get.docker.com -o get-docker.sh

sh get-docker.sh
```

[Für mehr Information zur Docker-Installation]{<https://phoenixnap.com/kb/docker-on-raspberry-pi>}

### Portainer Installation

Portainer ist ein nützliches Verwaltungstool für Docker. In DietPi ist es bereits durch die Konfigurationsdatei dietpi.txt automatisch installiert.

[Installation von Portainer für andere Linux-Versionen](https://docs.portainer.io/start/install-ce/server/docker/linux)
