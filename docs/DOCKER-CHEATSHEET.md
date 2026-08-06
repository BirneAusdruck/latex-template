# 🐳 Docker Grundlagen – Befehlsreferenz

> Diese README dient als Vorlage und Nachschlagewerk für alle Docker-Projekte.

---

## Inhaltsverzeichnis

1. [Images](#images)
2. [Dockerfile bauen](#dockerfile-bauen)
3. [Container starten & verwalten](#container-starten--verwalten)
4. [Container-Status & Logs](#container-status--logs)
5. [In laufende Container hinein](#in-laufende-container-hinein)
6. [Volumes](#volumes)
7. [Netzwerke](#netzwerke)
8. [Aufräumen & Löschen](#aufräumen--löschen)
9. [Docker Compose](#docker-compose)
10. [Wichtige Parameter im Überblick](#wichtige-parameter-im-überblick)
11. [Nützliche Hilfsbefehle](#nützliche-hilfsbefehle)

---

## Images

```bash
# Alle lokalen Images anzeigen
docker images

# Image von Docker Hub / einer Registry herunterladen
docker pull ubuntu:22.04

# Image löschen
docker rmi mein-image:latest

# Image umbenennen / neu taggen
docker tag mein-image:latest mein-image:v2

# Details zu einem Image anzeigen (Layer, ENV, Konfiguration)
docker inspect mein-image:latest

# Historie/Layer eines Images anzeigen
docker history mein-image:latest

# Image als .tar-Datei exportieren (z.B. für Transfer ohne Registry)
docker save -o mein-image.tar mein-image:latest

# Image aus .tar-Datei wieder einspielen
docker load -i mein-image.tar
```

---

## Dockerfile bauen

```bash
# Image aus Dockerfile im aktuellen Verzeichnis bauen
docker build -t mein-image:latest .

# Mit anderem Dockerfile-Namen/-Pfad
docker build -f pfad/zum/Dockerfile -t mein-image:latest .

# Ohne Cache neu bauen (erzwingt alle Schritte neu)
docker build --no-cache -t mein-image:latest .

# Build-Argumente übergeben (siehe ARG im Dockerfile)
docker build --build-arg USERNAME=vscode --build-arg USER_UID=1000 -t mein-image:latest .

# Für eine andere Zielarchitektur bauen (z.B. ARM auf Intel-Rechner)
docker build --platform linux/arm64 -t mein-image:latest .
```

> **Tipp:** `-t name:tag` vergibt einen lesbaren Namen für das gebaute Image (siehe [Wichtige Parameter](#wichtige-parameter-im-überblick)). Ohne `-t` ist das Image nur über eine kryptische ID ansprechbar.

---

## Container starten & verwalten

```bash
# Container aus einem Image starten
docker run mein-image:latest

# Im Hintergrund starten (detached)
docker run -d mein-image:latest

# Interaktiv mit Terminal starten (z.B. für Shell-Zugriff)
docker run -it mein-image:latest bash

# Container automatisch löschen, sobald er beendet ist
docker run --rm mein-image:latest

# Port-Weiterleitung: Host-Port 8080 → Container-Port 80
docker run -p 8080:80 mein-image:latest

# Umgebungsvariable setzen
docker run -e ANTHROPIC_API_KEY=xxx mein-image:latest

# Volume mounten (benanntes Volume)
docker run -v mein-volume:/pfad/im/container mein-image:latest

# Host-Ordner in den Container mounten (Bind Mount)
docker run -v /host/pfad:/container/pfad mein-image:latest

# Container mit Namen starten (statt zufälliger ID)
docker run --name mein-container mein-image:latest

# Container starten / stoppen / neu starten (bereits existierender Container)
docker start mein-container
docker stop mein-container
docker restart mein-container

# Container hart abwürgen (statt sauberem Stop)
docker kill mein-container

# Container löschen
docker rm mein-container

# Laufenden Container erzwungen löschen
docker rm -f mein-container
```

---

## Container-Status & Logs

```bash
# Laufende Container anzeigen
docker ps

# Alle Container anzeigen (auch gestoppte)
docker ps -a

# Nur Container-IDs anzeigen (nützlich für Scripting)
docker ps -aq

# Logs eines Containers anzeigen
docker logs mein-container

# Logs live mitverfolgen (wie tail -f)
docker logs -f mein-container

# Ressourcennutzung laufender Container anzeigen (CPU, RAM, Netzwerk)
docker stats

# Details zu einem Container anzeigen (Netzwerk, Mounts, ENV, ...)
docker inspect mein-container
```

---

## In laufende Container hinein

```bash
# Shell in einem bereits laufenden Container öffnen
docker exec -it mein-container bash

# Falls bash nicht vorhanden ist (z.B. Alpine-Images)
docker exec -it mein-container sh

# Einzelnen Befehl im Container ausführen, ohne interaktive Shell
docker exec mein-container ls /workspace

# Datei vom Host in den Container kopieren
docker cp datei.txt mein-container:/pfad/im/container/

# Datei aus dem Container auf den Host kopieren
docker cp mein-container:/pfad/im/container/datei.txt .
```

---

## Volumes

```bash
# Alle Volumes anzeigen
docker volume ls

# Details zu einem Volume anzeigen (u.a. tatsächlicher Speicherort)
docker volume inspect mein-volume

# Neues, leeres Volume anlegen (meist nicht nötig, docker run legt es automatisch an)
docker volume create mein-volume

# Einzelnes Volume löschen (darf nicht mehr gemountet sein)
docker volume rm mein-volume

# Alle nicht mehr verwendeten Volumes löschen
docker volume prune
```

> ⚠️ **`docker volume prune`** löscht alle Volumes, die aktuell in keinem Container gemountet sind — auch versehentlich wichtige, z.B. gespeicherte Logins. Vorher mit `docker volume ls` prüfen, was betroffen wäre.

---

## Netzwerke

```bash
# Alle Netzwerke anzeigen
docker network ls

# Details zu einem Netzwerk anzeigen (verbundene Container, Subnetz)
docker network inspect mein-netzwerk

# Neues Netzwerk anlegen (z.B. damit mehrere Container sich per Namen erreichen)
docker network create mein-netzwerk

# Container nachträglich mit einem Netzwerk verbinden
docker network connect mein-netzwerk mein-container

# Verbindung trennen
docker network disconnect mein-netzwerk mein-container

# Ungenutzte Netzwerke löschen
docker network prune
```

---

## Aufräumen & Löschen

```bash
# Alle gestoppten Container löschen
docker container prune

# Alle nicht von einem Container referenzierten Images löschen
docker image prune

# Auch alle Images löschen, die von keinem Container genutzt werden
# (nicht nur "dangling"/unbenannte Layer)
docker image prune -a

# Speicherplatz-Übersicht (Images, Container, Volumes, Cache)
docker system df

# Alles Ungenutzte auf einmal aufräumen (Container, Netzwerke, Images, Build-Cache)
docker system prune

# Zusätzlich auch ungenutzte Volumes mit aufräumen (Vorsicht!)
docker system prune --volumes
```

> ⚠️ `docker system prune --volumes` ist der radikalste Aufräumbefehl — er kann auch Volumes mit wichtigen, dauerhaften Daten löschen (z.B. Datenbank-Inhalte, gespeicherte Logins). Immer erst `docker volume ls` / `docker ps -a` prüfen.

---

## Docker Compose

Für Setups mit mehreren zusammenhängenden Containern (z.B. App + Datenbank), definiert in einer `docker-compose.yml`.

```bash
# Alle Services im Hintergrund starten
docker compose up -d

# Services stoppen (Container bleiben erhalten)
docker compose stop

# Services stoppen UND Container entfernen
docker compose down

# Zusätzlich auch die zugehörigen Volumes entfernen
docker compose down -v

# Logs aller Services anzeigen
docker compose logs -f

# Images vor dem Start neu bauen
docker compose up -d --build

# Laufende Services anzeigen
docker compose ps

# Befehl in einem bestimmten Service ausführen
docker compose exec service-name bash
```

---

## Wichtige Parameter im Überblick

| Parameter | Bedeutung |
|---|---|
| `-t name:tag` | Vergibt einen lesbaren Namen (Tag) beim Build |
| `-d` | Detached — Container läuft im Hintergrund |
| `-it` | Interaktiv + Terminal (für Shell-Zugriff nötig) |
| `--rm` | Container nach Beenden automatisch löschen |
| `-p host:container` | Port-Weiterleitung |
| `-e KEY=wert` | Umgebungsvariable setzen |
| `-v quelle:ziel` | Volume oder Bind Mount einhängen |
| `--name` | Fester Containername statt zufälliger ID |
| `--build-arg` | Wert für ein `ARG` im Dockerfile übergeben |
| `--no-cache` | Build-Cache beim Bauen ignorieren |
| `-f` | Anderen Dateipfad angeben (Dockerfile) bzw. erzwingen (rm, rmi) |
| `-a` | "Alle" — z.B. bei `ps -a` (auch gestoppte Container) |

---

## Nützliche Hilfsbefehle

```bash
# Docker-Version anzeigen
docker --version
docker version          # ausführlicher, Client + Server

# Systemweite Docker-Infos (Speicherort, Anzahl Container/Images, ...)
docker info

# Prozesse innerhalb eines Containers anzeigen
docker top mein-container

# Änderungen im Dateisystem eines Containers seit dem Start anzeigen
docker diff mein-container

# Aus einem laufenden Container ein neues Image machen (Snapshot)
docker commit mein-container neues-image:latest

# Layer-für-Layer-Größe eines Images einsehen (Debugging von zu großen Images)
docker history --no-trunc mein-image:latest
```

---

## Tipps

- **Named Volumes vs. Bind Mounts:** Named Volumes (`-v mein-volume:/pfad`) werden von Docker selbst verwaltet und sind ideal für persistente Daten wie Logins oder Datenbanken. Bind Mounts (`-v /host/pfad:/container/pfad`) spiegeln einen konkreten Host-Ordner — praktisch für Quellcode während der Entwicklung.
- **`docker build` vs. `docker run`:** `build` erzeugt ein Image (die Vorlage), `run` erzeugt daraus einen laufenden Container (die Instanz). Ein Image kann beliebig oft in mehrere Container gestartet werden.
- **Aufräum-Reihenfolge bei Platzmangel:** Erst `docker system df` zur Übersicht, dann gezielt `docker image prune -a` bzw. `docker volume prune` statt direkt zum radikalen `docker system prune --volumes` zu greifen.

---
