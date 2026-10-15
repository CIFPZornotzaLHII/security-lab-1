# =============================================================================
#  🛡️ Imagen de la máquina atacante (Kali Linux + herramientas del laboratorio)
# -----------------------------------------------------------------------------
#  La imagen oficial kalilinux/kali-rolling viene "pelada" para ser ligera.
#  Aquí preinstalamos lo que usamos en el lab para no repetir `apt install`
#  en cada clase. docker-compose la construye automáticamente (build: ./kali).
#
#  Construir a mano (opcional):   docker build -t kali-lab ./kali
# =============================================================================
FROM kalilinux/kali-rolling

# Evita preguntas interactivas durante la instalación
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    # --- red y escaneo ---
    nmap net-tools iproute2 iputils-ping dnsutils \
    # --- clientes de servicios ---
    openssh-client ftp curl wget netcat-traditional \
    # --- ataques de credenciales / análisis TLS ---
    hydra sslscan openssl \
    # --- utilidades ---
    sudo plocate less vim nano python3 python3-venv ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Carpeta de trabajo por defecto (se comparte con el anfitrión vía volumen)
WORKDIR /home

# Mantener el contenedor "vivo" cuando se arranca con tty
CMD ["bash"]
