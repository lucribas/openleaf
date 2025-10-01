# syntax=docker/dockerfile:1.6

########## STAGE 1: builder com ISO e /mnt/iso "montado" ##########
FROM sharelatex/sharelatex:latest AS tlbuilder
ENV DEBIAN_FRONTEND=noninteractive LANG=C.UTF-8

# Dependências para instalar a partir da ISO
RUN apt-get update && apt-get install -y --no-install-recommends \
      perl libarchive-tools ca-certificates \
 && rm -rf /var/lib/apt/lists/*

# Copie a ISO para o contexto de build (mesma pasta do Dockerfile)
ARG TEXLIVE_ISO=texlive2025.iso
COPY ${TEXLIVE_ISO} /opt/iso/texlive2025.iso

# "Monta" a ISO em /mnt/iso (montagem lógica via extração)
RUN set -eux; \
    mkdir -p /mnt/iso; \
    bsdtar -C /mnt/iso -xf /opt/iso/texlive2025.iso; \
    # sanity check: precisa existir o diretório archive dentro da ISO
    test -d /mnt/iso/archive

# Instala TeX Live 2025 offline a partir de /mnt/iso
RUN set -eux; \
    mkdir -p /tmp/itl; \
    # Descobre/obtém o instalador a partir de /mnt/iso
    ITL_DIR="$(bsdtar -tf /opt/iso/texlive2025.iso | awk -F/ '/^install-tl[^/]+\/$/ {print $1; exit}')" || true; \
    if [ -n "$ITL_DIR" ]; then \
      bsdtar -xpf /opt/iso/texlive2025.iso -C /tmp/itl "$ITL_DIR"; \
      INSTALLER="$(find /tmp/itl -maxdepth 3 -type f -name install-tl | head -n1)"; \
    else \
      ITL_TAR="$(bsdtar -tf /opt/iso/texlive2025.iso | awk '/install-tl[^/]*\.tar\.gz$/ {print; exit}')" || true; \
      [ -n "$ITL_TAR" ] || (echo "install-tl não encontrado na ISO" && exit 1); \
      bsdtar -xOf /opt/iso/texlive2025.iso "$ITL_TAR" | bsdtar -xzf - -C /tmp/itl; \
      INSTALLER="$(find /tmp/itl -maxdepth 3 -type f -name install-tl | head -n1)"; \
    fi; \
    [ -n "$INSTALLER" ]; \
    # Profile de instalação
    printf '%s\n' \
      'selected_scheme scheme-full' \
      'TEXDIR /opt/texlive-offline/2025' \
      'TEXMFLOCAL /opt/texlive-offline/texmf-local' \
      'TEXMFSYSVAR /opt/texlive-offline/2025/texmf-var' \
      'TEXMFSYSCONFIG /opt/texlive-offline/2025/texmf-config' \
      'TEXMFVAR ~/.texlive2025/texmf-var' \
      'TEXMFCONFIG ~/.texlive2025/texmf-config' \
      'TEXMFHOME ~/texmf' \
      'tlpdbopt_autobackup 0' \
      'tlpdbopt_install_docfiles 0' \
      'tlpdbopt_install_srcfiles 0' \
      'option_doc 0' \
      'option_src 0' \
      > /tmp/texlive.profile; \
    # Agora o repositório é o /mnt/iso "montado"
    perl "$INSTALLER" -repository /mnt/iso/ -profile /tmp/texlive.profile; \
    /opt/texlive-offline/2025/bin/x86_64-linux/tlmgr path add || true; \
    /opt/texlive-offline/2025/bin/x86_64-linux/mktexlsr || true; \
    # limpa o que não é mais necessário neste estágio
    rm -rf /tmp/itl

########## STAGE 2: imagem final enxuta (sem ISO) ##########
FROM sharelatex/sharelatex:latest
ENV DEBIAN_FRONTEND=noninteractive LANG=C.UTF-8

# Copia apenas o TeX Live já instalado
COPY --from=tlbuilder /opt/texlive-offline /opt/texlive-offline

# Coloca o TL à frente no PATH
ENV PATH=/opt/texlive-offline/2025/bin/x86_64-linux:$PATH

# Desabilita repositório do tlmgr (sem ISO no runtime)
RUN tlmgr option repository "disabled" || true \
 && tlmgr option autobackup 0 || true \
 && tlmgr option docfiles 0 || true \
 && tlmgr option srcfiles 0 || true \
 && tlmgr path add || true \
 && mktexlsr || true

EXPOSE 80
