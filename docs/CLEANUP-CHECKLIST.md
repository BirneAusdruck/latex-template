# Cleanup-Checkliste (fremde/geliehene Geräte)

Nur relevant, wenn du dieses Template auf einem **Gerät nutzt, das dir nicht gehört** oder das andere Personen mitnutzen (z. B. Uni-Rechner, geliehener Laptop). Auf deinem eigenen Gerät ist das **nicht nötig** — Container, Images und Volumes bewusst liegen lassen, das macht den nächsten Start schnell und den Claude-Login persistent.

## Warum das wichtig ist

Der Claude-Code-Login liegt in einem Docker-**Volume** (`claude-code-config-shared`), nicht im Projektordner selbst. Er verschwindet also **nicht automatisch**, wenn du das Repo löschst oder VS Code schließt — er bleibt auf dem Gerät liegen, bis er explizit entfernt wird.

## Schritt 1: In Claude Code ausloggen

Im Container-Terminal, während `claude` läuft:
```
/logout
```
Oder von außen, ohne interaktive Sitzung:
```bash
claude auth logout
```
Das invalidiert den Token **serverseitig** — wichtigster Schritt, nicht nur lokale Dateien löschen.

**Achtung:** Falls statt der normalen Anmeldung ein `ANTHROPIC_API_KEY` als Umgebungsvariable gesetzt war, wirkt sich `claude auth logout` **nicht** darauf aus. In dem Fall ist Schritt 4 (Volume löschen) der eigentlich entscheidende Schritt.

## Schritt 2: Container stoppen und löschen

```bash
docker ps -a
docker rm -f <container-id>
```

## Schritt 3: Gebautes Image löschen

```bash
docker images
docker rmi <image-id>
```

## Schritt 4: Claude-Config-Volume löschen

```bash
docker volume ls
docker volume rm claude-code-config-shared
```

Falls dabei zusätzlich verwaiste Volumes mit einem langen Hash-Suffix auftauchen (Muster `claude-code-config-<hash>`, z. B. Reste aus früheren Testläufen mit `${devcontainerId}` statt festem Namen) — vor dem Löschen kurz Inhalt prüfen:
```bash
docker run --rm -v <volume-name>:/data alpine ls -la /data
```
Falls dort `.credentials.json` o. ä. liegt: löschen.
```bash
docker volume rm <volume-name>
```

## Schritt 5: Kontrolle

```bash
docker ps -a
docker images
docker volume ls
```
Nichts Projektbezogenes sollte hier mehr auftauchen (das `vscode`-Volume ist die Ausnahme, siehe unten).

## Optional: Kompletter Rundum-Schlag

Nur wenn auf dem Gerät sonst **kein anderes Docker-Projekt** läuft, das erhalten bleiben soll:
```bash
docker system prune -a --volumes
```
**Vorsicht:** Löscht wirklich alle gestoppten Container, ungenutzten Images und Volumes auf dem gesamten Gerät — nicht nur dieses Projekt.

## Hinweis zum `vscode`-Volume

Neben den Claude-Volumes taucht meist auch ein Volume namens `vscode` auf. Das ist **kein** projekt- oder Claude-spezifisches Volume, sondern internes Arbeitsverzeichnis von VS Code Dev Containers selbst (Server-Komponenten, Cache) — repo-übergreifend geteilt für jeden Dev Container auf dem Gerät. Enthält keine Zugangsdaten, ist also nicht sicherheitsrelevant. Löschen ist unbedenklich, kostet aber beim nächsten Dev-Container-Start auf diesem Gerät (egal welches Projekt) etwas Zeit, da VS Code die Server-Komponenten neu installieren muss.
