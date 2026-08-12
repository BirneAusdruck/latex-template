# LaTeX Dev Container (Template)

Portable LaTeX-Umgebung für VS Code via Docker. Funktioniert auf jedem Gerät, auf dem Docker und VS Code laufen, aktuell getestet/gedacht für Linux, sollte aber unverändert auch unter Windows/macOS mit Docker Desktop funktionieren.

Dieses Repo ist als **Template** gedacht: Auf GitHub unter Repo-Settings → "Template repository" markieren. Für jeden neuen Kontext (z. B. private Dokumente, Uni, Skripte etc.) dann über die Template-Funktion ein neues Repo erzeugen und dort die eigenen `.tex`-Dateien ablegen.

## Zwei Konfigurationen zur Auswahl

Das Template bringt **zwei Dev-Container-Konfigurationen** mit, zwischen denen du direkt in VS Code umschalten kannst — kein Kopieren, kein manuelles Bearbeiten nötig:

| Konfiguration | Enthält | Wann sinnvoll |
|---|---|---|
| **mit-claude** | LaTeX-Umgebung + Claude Code CLI/Extension | Wenn du Claude Code direkt im Projekt nutzen willst |
| **ohne-claude** | Nur die reine LaTeX-Umgebung | Wenn du Claude Code nicht brauchst — schlankeres Image, schnellerer Build |

Beide Varianten nutzen **dieselbe, gemeinsame** `.devcontainer/Dockerfile` (reine LaTeX-Basis auf `debian:bookworm-slim`) — der Unterschied liegt ausschließlich in der jeweiligen `devcontainer.json`.

### Konfiguration auswählen

```
Strg+Shift+P → Dev Containers: Switch Container Configuration
```

Dort erscheinen beide Varianten (`mit-claude`, `ohne-claude`) zur Auswahl. VS Code baut direkt mit der gewählten Konfiguration.

Falls du beim allerersten Öffnen des Projekts direkt zur Auswahl kommen willst, statt eine Konfiguration automatisch per Popup vorgeschlagen zu bekommen:
```
Strg+Shift+P → Dev Containers: Reopen in Container
```
— zeigt bei mehreren vorhandenen Konfigurationen ebenfalls eine Auswahl an.

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

Achtung bei Variante 3! Durch die auto Build und PDF creation features werden eine Vielzahl von Compilaten pro `.tex` Datei erzeugt.

Jeder Kontext = ein Repo = ein VS-Code-Projekt, den du direkt in einem im Devcontainer öffnest. Kein Vermischen von Dokumenten aus unterschiedlichen Lebensbereichen, aber auch kein `.devcontainer/` pro einzelnem Übungsblatt.

## Voraussetzungen (auf jedem Gerät, das genutzt werden soll)

1. **Docker** (bzw. Docker Engine) installiert und laufend
2. **VS Code**
3. VS-Code-Erweiterung **"Dev Containers"** (`ms-vscode-remote.remote-containers`)

Das sind die einzigen lokalen Abhängigkeiten. Kein Vorab-Build eines eigenen Basis-Images mehr nötig — beide Varianten bauen direkt aus diesem Repo, ohne externe Abhängigkeit auf dem Host.

## Nutzung

1. Diesen Ordner (bzw. dein LaTeX-Projekt, in das du `.devcontainer/` kopierst) in VS Code öffnen.
2. Gewünschte Konfiguration wählen (siehe [Zwei Konfigurationen zur Auswahl](#zwei-konfigurationen-zur-auswahl)) und im Container öffnen.
3. Beim erstmaligen Start wird das Docker-Image gebaut (dauert ein paar Minuten, danach nur noch Sekunden dank Cache).
4. `main.tex` öffnen und mit `Strg+Alt+B` bauen, oder einfach speichern — Autobuild ist aktiv.
5. PDF öffnet sich automatisch in einem Tab daneben.

## Standard Template-Struktur

```
# Entspricht der Variante 1
.
├── .devcontainer/
│   ├── Dockerfile          # gemeinsame TeX-Live-Umgebung + Tools, für beide Varianten
│   ├── mit-claude/
│   │   └── devcontainer.json
│   └── ohne-claude/
│       └── devcontainer.json
├── .vscode/                # optional, lokale Extra-Settings
├── kontext_1               # Ordner unterteilte `.tex` files
│   ├── main.tex            # Beispieldokument zum Testen
├── kontext_2               # Ordner unterteilte `.tex` files
│   ├── main.tex            # Beispieldokument zum Testen
└── README.md
```

## Anpassen

- **Mehr/weniger TeX-Pakete**: Liste in `.devcontainer/Dockerfile` erweitern oder auf `texlive-full` umstellen (dann Base-Image deutlich größer, ~5-7 GB mehr). Wirkt sich auf **beide** Varianten aus, da sie sich dieselbe Dockerfile teilen.
- **Andere Extensions**: In der jeweiligen `devcontainer.json` (`mit-claude/` bzw. `ohne-claude/`) unter `customizations.vscode.extensions` ergänzen.
- **Mehrere Projekte**: Einfach den `.devcontainer/`-Ordner in jedes LaTeX-Projekt kopieren, oder ein zentrales Template-Repo daraus machen und pro Projekt klonen/forken.

## Claude Code (Variante `mit-claude`)

Diese Variante bindet Claude Code über das **offizielle Dev Container Feature von Anthropic** ein — kein eigenes Basis-Image, kein lokaler Vorab-Build nötig:

```json
"features": {
  "ghcr.io/devcontainers/features/node:1": {},
  "ghcr.io/anthropics/devcontainer-features/claude-code:1": {}
}
```

Das Feature installiert Node.js (Voraussetzung für die Claude Code CLI) sowie Claude Code selbst automatisch beim Bauen des Containers, direkt aus Anthropics zentral gepflegter Quelle. Updates an Claude Code kommen dadurch automatisch mit, ohne dass du selbst etwas pflegen musst.

### Login-Persistenz über Rebuilds hinweg

```json
"containerEnv": {
  "CLAUDE_CONFIG_DIR": "/home/vscode/.claude"
},
"mounts": [
  "source=claude-code-config-shared,target=/home/vscode/.claude,type=volume"
]
```

Der Volume-Name `claude-code-config-shared` ist bewusst **fest** gewählt (nicht `${devcontainerId}`): Alle aus diesem Template erzeugten Repos teilen sich denselben Login — einmal `claude` im Terminal ausführen und einloggen reicht für alle Projekte, auch über Rebuilds hinweg.

Falls du stattdessen pro Projekt einen **eigenen, getrennten** Login willst (kein geteilter Zugang zwischen Repos), `${devcontainerId}` statt des festen Namens verwenden:
```json
"mounts": [
  "source=claude-code-config-${devcontainerId},target=/home/vscode/.claude,type=volume"
]
```

### Erste Anmeldung

Nach dem ersten Start der `mit-claude`-Variante im Container-Terminal:
```bash
claude
```
Folgt dem Anmelde-Prompt. Danach bleibt die Anmeldung dank des Volume-Mounts über jeden weiteren Rebuild hinweg erhalten.

## Claude Code nicht nutzen

Einfach beim Öffnen des Projekts die **`ohne-claude`**-Konfiguration wählen (siehe [Zwei Konfigurationen zur Auswahl](#zwei-konfigurationen-zur-auswahl)). Kein Umschreiben von Dateien nötig, beide Varianten liegen bereits fertig nebeneinander im Repo.
