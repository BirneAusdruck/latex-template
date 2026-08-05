# LaTeX Dev Container (Template)

Portable LaTeX-Umgebung für VS Code via Docker. Funktioniert auf jedem Gerät, auf dem Docker und VS Code laufen — aktuell getestet/gedacht für Linux, sollte aber unverändert auch unter Windows/macOS mit Docker Desktop funktionieren.

Dieses Repo ist als **Template** gedacht: Auf GitHub unter Repo-Settings → "Template repository" markieren. Für jeden neuen Kontext (z. B. Uni, Übungsaufgaben, private Dokumente) dann über "Use this template" ein neues Repo erzeugen und dort die eigenen `.tex`-Dateien ablegen.

## Empfohlene Struktur (ein Repo pro Kontext, nicht pro Dokument)

```
uni-latex/                    # eigenes Repo, aus diesem Template erzeugt
├── .devcontainer/
├── .gitignore
├── mathe/
│   ├── uebung1.tex
│   └── uebung3.tex
└── info/
    └── skript-notizen.tex

niece-arbeitsblaetter/         # eigenes Repo, aus diesem Template erzeugt
├── .devcontainer/
├── .gitignore
├── brueche.tex
└── prozentrechnung.tex
```

Jeder Kontext = ein Repo = ein VS-Code-Workspace, den du direkt in einem Rutsch im Dev Container öffnest. Kein Vermischen von Dokumenten aus unterschiedlichen Lebensbereichen, aber auch kein `.devcontainer/` pro einzelnem Übungsblatt.

## Voraussetzungen (auf jedem Gerät, das genutzt werden soll)

1. **Docker** (bzw. Docker Engine) installiert und laufend
2. **VS Code**
3. VS-Code-Erweiterung **"Dev Containers"** (`ms-vscode-remote.remote-containers`)

Das sind die einzigen lokalen Abhängigkeiten — der Rest (TeX Live, latexmk, biber, LaTeX Workshop, ...) steckt im Container.

## Nutzung

1. Diesen Ordner (bzw. dein LaTeX-Projekt, in das du `.devcontainer/` kopierst) in VS Code öffnen.
2. VS Code fragt automatisch: **"Reopen in Container"** → klicken.
   (Falls die Meldung nicht kommt: `Strg+Shift+P` → "Dev Containers: Reopen in Container")
3. Beim ersten Start wird das Docker-Image gebaut (dauert ein paar Minuten, danach nur noch Sekunden dank Cache).
4. `main.tex` öffnen und mit `Strg+Alt+B` bauen, oder einfach speichern — Autobuild ist aktiv.
5. PDF öffnet sich automatisch in einem Tab daneben.

## Struktur

```
.
├── .devcontainer/
│   ├── Dockerfile          # TeX-Live-Umgebung + Tools
│   └── devcontainer.json   # VS-Code-Konfiguration, Extensions, Settings
├── .vscode/                # optional, lokale Extra-Settings
├── main.tex                # Beispieldokument zum Testen
└── README.md
```

## Anpassen

- **Mehr/weniger TeX-Pakete**: Liste in `.devcontainer/Dockerfile` erweitern oder auf `texlive-full` umstellen (dann Base-Image deutlich größer, ~5-7 GB mehr).
- **Andere Extensions**: In `devcontainer.json` unter `customizations.vscode.extensions` ergänzen.
- **Mehrere Projekte**: Einfach den `.devcontainer/`-Ordner in jedes LaTeX-Projekt kopieren, oder ein zentrales Template-Repo daraus machen und pro Projekt klonen/forken.

## Später: andere Betriebssysteme

Da alles über Docker + Dev Containers läuft, sollte auf Windows/macOS mit installiertem Docker Desktop exakt derselbe Ablauf funktionieren — ohne Änderungen an Dockerfile oder devcontainer.json. Der einzige Unterschied ist dort ggf. etwas mehr Overhead durch die Docker-Desktop-VM.
