FROM codercom/code-server:latest

USER root

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
      bash-completion \
      ca-certificates \
      curl \
      fonts-dejavu \
      fonts-liberation \
      fonts-noto \
      fonts-noto-cjk \
      fonts-noto-color-emoji \
      fonts-noto-extra \
      fonts-noto-mono \
      git \
      gnupg \
      hunspell \
      hunspell-fr \
      latexmk \
      make \
      openssh-client \
      perl \
      python3 \
      texlive-bibtex-extra \
      texlive-extra-utils \
      texlive-fonts-extra \
      texlive-fonts-recommended \
      texlive-lang-english \
      texlive-lang-french \
      texlive-latex-base \
      texlive-latex-extra \
      texlive-latex-recommended \
      texlive-luatex \
      texlive-pictures \
      texlive-plain-generic \
      texlive-publishers \
      texlive-science \
      texlive-xetex \
      biber \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /opt/code-server/extensions \
    && chown -R coder:coder /opt/code-server

USER coder

RUN code-server --extensions-dir /opt/code-server/extensions --install-extension james-yu.latex-workshop \
    && code-server --extensions-dir /opt/code-server/extensions --install-extension streetsidesoftware.code-spell-checker \
    && code-server --extensions-dir /opt/code-server/extensions --install-extension streetsidesoftware.code-spell-checker-french

WORKDIR /home/coder
