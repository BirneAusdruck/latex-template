# LaTeX Dev Container (Template)

Portable LaTeX-Umgebung für VS Code via Docker. Funktioniert auf jedem Gerät, auf dem Docker und VS Code laufen — aktuell getestet/gedacht für Linux, sollte aber unverändert auch unter Windows/macOS mit Docker Desktop funktionieren.

Dieses Repo ist als **Template** gedacht: Auf GitHub unter Repo-Settings → "Template repository" markieren. Für jeden neuen Kontext (z. B. private Dokumente, Skripte etc.) dann über die Template-Funktion ein neues Repo erzeugen und dort die eigenen `.tex`-Dateien ablegen.

## Empfohlene Struktur (ein Repo pro Kontext, nicht pro Dokument)

```
private_documents/                   # Variante 1: eigenes Repo, aus diesem Template erzeugt
├── .devcontainer/
├── .gitignore
├── private_doc_1/
│   ├── doc1.tex
│   └── doc2.tex
└── private_doc_2/
    └── doc1.tex

university_latex/                    # Variante 2: eigenes Repo, aus diesem Template erzeugt
├── .devcontainer/
├── .gitignore
├── modul1/
│   ├── exercises/
│   │   └──ex1.tex
│   └── solutions/
│       └──so1.tex
└── modul2/
    ├── exercises/
    │   └──ex1.tex
    └── solutions/
        └──so1.tex

scripts/                              # Variante 3:  eigenes Repo, aus diesem Template erzeugt
├── .devcontainer/
├── .gitignore
├── script1.tex
└── script2.tex
```

Achtung bei Variante 3! Durch die auto Build und PDF creation features werden eine vielzahl von Compilaten pro `.tex` Datei erzeugt.

Jeder Kontext = ein Repo = ein VS-Code-Workspace, den du direkt in einem Rutsch im Dev Container öffnest. Kein Vermischen von Dokumenten aus unterschiedlichen Lebensbereichen, aber auch kein `.devcontainer/` pro einzelnem Übungsblatt.

## Voraussetzungen (auf jedem Gerät, das genutzt werden soll)

1. **Docker** (bzw. Docker Engine) installiert und laufend
2. **VS Code**
3. VS-Code-Erweiterung **"Dev Containers"** (`ms-vscode-remote.remote-containers`)
4. **`claude-base:latest`-Image lokal gebaut** — siehe [Claude Code einbinden](#claude-code-einbinden-standard). Dieses Template baut standardmäßig auf diesem Image auf; ohne es schlägt der Container-Build fehl.

Das sind die einzigen lokalen Abhängigkeiten — der Rest (TeX Live, latexmk, biber, LaTeX Workshop, Claude Code, ...) steckt im Container.

## Nutzung

1. Diesen Ordner (bzw. dein LaTeX-Projekt, in das du `.devcontainer/` kopierst) in VS Code öffnen.
2. Über VS Code das Projekt im Container öffnen
   (entweder per automatischen Aufforderung von VS Code oder über den command `Strg+Shift+P` → **"Dev Containers: Reopen in Container"**).
3. Beim erstmaligen Start wird das Docker-Image gebaut (dauert ein paar Minuten, danach nur noch Sekunden dank Cache). Voraussetzung: `claude-base:latest` wurde bereits lokal gebaut (siehe [Voraussetzungen](#voraussetzungen-auf-jedem-gerät-das-genutzt-werden-soll) und [Claude Code einbinden](#claude-code-einbinden-standard)).
4. `main.tex` öffnen und mit `Strg+Alt+B` bauen, oder einfach speichern — Autobuild ist aktiv.
5. PDF öffnet sich automatisch in einem Tab daneben.

## Standard Template-Struktur

```
# Entspricht der Variante 1
.
├── .devcontainer/
│   ├── Dockerfile          # TeX-Live-Umgebung + Tools
│   └── devcontainer.json   # VS-Code-Konfiguration, Extensions, Settings
├── .vscode/                # optional, lokale Extra-Settings
├── kontext_1               # Ordner unterteilte `.tex` files
│   ├── main.tex            # Beispieldokument zum Testen
├── kontext_2               # Ordner unterteilte `.tex` files
│   ├── main.tex            # Beispieldokument zum Testen
└── README.md
```

## Anpassen

- **Mehr/weniger TeX-Pakete**: Liste in `.devcontainer/Dockerfile` erweitern oder auf `texlive-full` umstellen (dann Base-Image deutlich größer, ~5-7 GB mehr).
- **Andere Extensions**: In `devcontainer.json` unter `customizations.vscode.extensions` ergänzen.
- **Mehrere Projekte**: Einfach den `.devcontainer/`-Ordner in jedes LaTeX-Projekt kopieren, oder ein zentrales Template-Repo daraus machen und pro Projekt klonen/forken.

## Claude Code einbinden (Standard)

Dieses Template ist standardmäßig mit der [Claude Code](https://code.claude.com) VS-Code-Extension + CLI ausgestattet, inklusive persistentem Login über einen Rebuild hinweg. Dafür baut `.devcontainer/Dockerfile` auf einem eigenen, wiederverwendbaren Basis-Image (`claude-base:latest`) auf, statt direkt auf `debian:bookworm-slim`. Das spart bei mehreren aus diesem Template erzeugten Repos wiederholte Installationszeit und -bandbreite, da Node.js + Claude Code CLI nur einmal gebaut werden.

### Einmalige Einrichtung (auf jedem Gerät, das genutzt werden soll)

**1. Eigenes Dockerfile für das Basis-Image** anlegen, außerhalb dieses Repos (z. B. unter `~/docker-bases/claude-base/Dockerfile`):

```dockerfile
# syntax=docker/dockerfile:1
FROM debian:bookworm-slim

# ---- Konfiguration ----------------------------------------------------
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=1000
ARG NODE_MAJOR=20

# ---- Basis-Pakete -------------------------------------------------------
# curl/ca-certificates: für NodeSource-Setup-Skript
# git: üblich für Dev-Container-Workflows
# sudo: falls der User später Root-Rechte im Container braucht
RUN apt-get update && apt-get install -y --no-install-recommends \
        curl \
        ca-certificates \
        git \
        sudo \
    && rm -rf /var/lib/apt/lists/*

# ---- Node.js -------------------------------------------------------------
# Wird noch als root installiert, damit die globale npm-Installation
# im nächsten Schritt für alle User zugänglich ist.
RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_MAJOR}.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/*

# ---- Claude Code CLI -------------------------------------------------------
RUN npm install -g @anthropic-ai/claude-code

# ---- Non-root User anlegen -------------------------------------------------
# Falls User/Group-ID schon existiert (kommt bei manchen Base-Images vor),
# bricht useradd/groupadd sonst ab – daher der Check.
RUN if ! getent group "${USER_GID}" >/dev/null; then \
        groupadd --gid "${USER_GID}" "${USERNAME}"; \
    fi \
    && if ! id -u "${USER_UID}" >/dev/null 2>&1; then \
        useradd --uid "${USER_UID}" --gid "${USER_GID}" -m -s /bin/bash "${USERNAME}"; \
    fi \
    && usermod -aG sudo "${USERNAME}" \
    && echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
    && chmod 0440 /etc/sudoers.d/${USERNAME}

# ---- Claude-Config-Verzeichnis vorbereiten ---------------------------------
# Wird später per Volume gemountet (siehe devcontainer.json) - hier nur
# sicherstellen, dass der Ordner existiert und dem User gehört, damit
# der allererste Mount nicht mit root-Berechtigungen daherkommt.
RUN mkdir -p /home/${USERNAME}/.claude \
    && chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.claude

USER ${USERNAME}
WORKDIR /home/${USERNAME}

# Kein CMD/ENTRYPOINT nötig - dieses Image dient nur als Basis (FROM ...)
# für projektspezifische Dockerfiles, nicht zum direkten Ausführen.
```

**2. Lokal bauen:**

```bash
docker build -t claude-base:latest .
```

Dieser Schritt ist pro Gerät **einmalig** nötig — danach kann jedes aus diesem Template erzeugte Repo darauf zugreifen, ohne den Build zu wiederholen.

### Wie es in diesem Template verwendet wird

`.devcontainer/Dockerfile` startet standardmäßig mit:

```dockerfile
FROM claude-base:latest

# ab hier nur noch die LaTeX-spezifischen Installationsschritte
```

`.devcontainer/devcontainer.json` enthält dazu passend:

```json
"containerEnv": {
  "CLAUDE_CONFIG_DIR": "/home/vscode/.claude"
},
"mounts": [
  "source=claude-code-config-shared,target=/home/vscode/.claude,type=volume"
],
"customizations": {
  "vscode": {
    "extensions": [
      "anthropic.claude-code"
    ]
  }
}
```

Der Volume-Name `claude-code-config-shared` ist bewusst **fest** gewählt (nicht `${devcontainerId}`): Alle aus diesem Template erzeugten Repos teilen sich denselben Login — einmal `claude` im Terminal ausführen und einloggen reicht für alle Projekte.

**Hinweis:** `claude-base:latest` existiert nur lokal auf dem Rechner, auf dem es gebaut wurde. Auf jedem neuen Gerät muss der Build-Schritt oben einmal wiederholt werden, bevor Repos aus diesem Template dort geöffnet werden können.

## Claude Code entfernen / nicht nutzen (optional)

Falls du Claude Code nicht brauchst und keine Abhängigkeit von `claude-base:latest` haben willst, kannst du die Integration in einem aus dem Template erzeugten Repo entfernen. Da `claude-base:latest` den `vscode`-User bereits mitbringt, musst du diesen beim Wechsel auf ein reines Debian-Image selbst nachbilden.

1. **In `.devcontainer/Dockerfile`** die erste Zeile ersetzen:
   ```dockerfile
   FROM debian:bookworm-slim
   ```
2. **Direkt danach den `vscode`-User selbst anlegen**, da dieser sonst (anders als bei `claude-base:latest`) nicht existiert:
   ```dockerfile
   ARG USERNAME=vscode
   ARG USER_UID=1000
   ARG USER_GID=$USER_UID

   RUN groupadd --gid $USER_GID $USERNAME \
       && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME
   ```
   Diese Zeilen vor `USER vscode` bzw. vor der ersten Stelle einfügen, an der der `vscode`-User referenziert wird (z. B. vor den `apt-get install`-Schritten, die als root laufen müssen, und nach denen dann auf `USER vscode` gewechselt wird).
3. **In `.devcontainer/devcontainer.json`** die Einträge `containerEnv`, den `mounts`-Block sowie `"anthropic.claude-code"` aus `customizations.vscode.extensions` löschen.

Damit entfällt die Voraussetzung, vorab `claude-base:latest` zu bauen, und der Container läuft komplett unabhängig davon — inklusive eigenständiger `vscode`-User-Anlage direkt im Projekt-Dockerfile.
