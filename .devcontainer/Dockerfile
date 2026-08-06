FROM claude-base:latest

ENV DEBIAN_FRONTEND=noninteractive

# claude-base:latest bringt bereits Node.js, die Claude Code CLI sowie
# den vscode-User (UID/GID 1000) mit - hier nur noch die LaTeX-spezifischen
# Installationsschritte. Für apt-get zurück auf root wechseln.
USER root

# TeX Live Pakete – scheme-medium-artig, statt texlive-full (~5-7 GB kleiner)
# Bei Bedarf einfach weitere Pakete ergänzen oder texlive-full nehmen.
# git/curl/ca-certificates sind bereits in claude-base:latest enthalten.
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
    && rm -rf /var/lib/apt/lists/*

# Zurück auf den non-root User, damit erzeugte Dateien (aux, pdf, ...)
# nicht root gehören.
USER vscode
WORKDIR /workspace
