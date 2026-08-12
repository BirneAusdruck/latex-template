# Basis-Image: schlankes Debian 12 ("bookworm"), minimal vorinstalliert
FROM debian:bookworm-slim

# Docker führt RUN-Befehle sonst standardmäßig über /bin/sh aus - bei Debian
# ist das ein Symlink auf "dash", nicht "bash". dash kennt den Befehl "source"
# nicht, den z. B. das Node.js-Feature (für Claude Code) intern braucht.
# Diese Zeile erzwingt bash als Shell für alle folgenden RUN-Anweisungen.
SHELL ["/bin/bash", "-c"]

# Verhindert interaktive Rückfragen von apt-get (z. B. Zeitzone, Tastatur-
# layout) während des Builds - ohne diese Variable würde der Build hängen
# bleiben, da beim Docker-Build niemand da ist, um sowas zu beantworten.
ENV DEBIAN_FRONTEND=noninteractive

# TeX Live Pakete – scheme-medium-artig, statt texlive-full (~5-7 GB kleiner)
# Bei Bedarf einfach weitere Pakete ergänzen oder texlive-full nehmen.
# curl/ca-certificates werden zusätzlich gebraucht, falls die Claude-Code-
# Variante dieses Images verwendet wird (Feature-Installation braucht das).
RUN apt-get update && apt-get install -y --no-install-recommends \
    texlive-latex-base \
    texlive-latex-recommended \
    texlive-latex-extra \
    texlive-fonts-recommended \
    texlive-fonts-extra \
    texlive-lang-german \
    texlive-science \
    texlive-bibtex-extra \
    biber \
    latexmk \
    chktex \
    git \
    openssh-client\
    curl \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Gruppe und Benutzer anlegen, damit erzeugte Dateien (aux, pdf, ...)
# nicht root gehören
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# -s /bin/bash setzt bash explizit als Login-Shell des neuen Nutzers.
# Ohne dieses Flag würde useradd bei Debian standardmäßig /bin/sh (dash)
# setzen - das Node-Feature wechselt intern per "su vscode -c ..." zu
# diesem Nutzer und würde dann wieder auf denselben "source: not found"-
# Fehler laufen, den wir mit SHELL ["/bin/bash", "-c"] weiter oben schon
# für den Docker-Build-Prozess selbst gelöst hatten (su nutzt eine andere
# Shell-Quelle als der Docker-Build).
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m -s /bin/bash $USERNAME

# Ab hier laufen alle Befehle als dieser Nutzer statt als root
USER $USERNAME

# Arbeitsverzeichnis im Container, hier landest du beim Verbinden
WORKDIR /workspace
