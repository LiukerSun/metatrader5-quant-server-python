# Stage 1: Base image with apt packages
FROM ghcr.io/linuxserver/baseimage-kasmvnc:debianbookworm AS base

ENV TITLE=MetaTrader
ENV WINEARCH=win64
ENV WINEPREFIX="/config/.wine"
ENV DISPLAY=:0

# Ensure the directory exists with correct permissions
RUN mkdir -p /config/.wine && \
    chown -R abc:abc /config/.wine && \
    chmod -R 755 /config/.wine

# Update package lists and upgrade packages
RUN sed -i '/bullseye-backports/d' /etc/apt/sources.list && \
    apt-get update --allow-releaseinfo-change && \
    apt-get upgrade -y
# Install required packages
RUN apt-get install -y \
    dos2unix \
    python3-pip \
    wget \
    python3-pyxdg \
    netcat-openbsd \
    && pip3 install --upgrade pip --break-system-packages

# Add WineHQ repository key and APT source
RUN wget -q https://dl.winehq.org/wine-builds/winehq.key > /dev/null 2>&1\
    && apt-key add winehq.key \
    && add-apt-repository 'deb https://dl.winehq.org/wine-builds/debian/ bookworm main' \
    && rm winehq.key

    # Add i386 architecture and update package lists
RUN dpkg --add-architecture i386 \
    && apt-get update

# Install WineHQ stable 9.0 (Wine 10.x/11.x causes "debugger found" error with MT5)
RUN apt-get install --install-recommends -y \
    winehq-stable=9.0.0.0~bookworm-1 \
    wine-stable=9.0.0.0~bookworm-1 \
    wine-stable-amd64=9.0.0.0~bookworm-1 \
    wine-stable-i386=9.0.0.0~bookworm-1 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Stage 2: Final image
FROM base

# Copy the scripts directory and convert start.sh to Unix format
COPY app /app
COPY scripts /scripts
RUN dos2unix /scripts/*.sh && \
    chmod +x /scripts/*.sh

COPY /root /
RUN touch /var/log/mt5_setup.log && \
    chown abc:abc /var/log/mt5_setup.log && \
    chmod 644 /var/log/mt5_setup.log

EXPOSE 3000 5000 5001 8001 18812
VOLUME /config
