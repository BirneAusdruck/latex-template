FROM debian:bookworm-slim

ENV DEBIAN_FRONTEND=noninteractive

# TeX Live Pakete – scheme-medium-artig, statt texlive-full (~5-7 GB kleiner)
# Bei Bedarf einfach weitere Pakete ergänzen oder texlive-full nehmen.
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
    curl \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Non-root User anlegen, damit erzeugte Dateien (aux, pdf, ...) nicht root gehören
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME

USER $USERNAME
WORKDIR /workspace
