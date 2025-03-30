ARG BASE_APP_IMAGE=ghcr.io/games-on-whales/base-app:edge

# hadolint ignore=DL3006
FROM ${BASE_APP_IMAGE}

ARG DEBIAN_FRONTEND=noninteractive

#GitHub REST query versioning
ARG GITHUB_REST_VERSION=2022-11-28


# The following list of packages is informed by these articles;
#   - https://github.com/lutris/docs/blob/master/InstallingDrivers.md
#   - https://github.com/lutris/docs/blob/master/WineDependencies.md
ARG REQUIRED_PACKAGES=" \
     cabextract \
     curl \
     fluid-soundfont-gs \
     gir1.2-gtk-3.0 \
     gir1.2-notify-0.7 \
     gir1.2-webkit2-4.1 \
     libasound2-plugins:i386 \
     libdbus-1-3:i386 \
     libfreetype6:i386 \
     libsdl2-2.0-0:i386 \
     libsqlite3-0:i386 \
     libvulkan1 \
     libvulkan1:i386 \
     mesa-utils \
     mesa-vulkan-drivers \
     mesa-vulkan-drivers:i386 \
     p7zip \
     python3-distro \
     python3-gi-cairo \
     python3-lxml \
     python3-magic \
     python3-pil \
     python3-requests \
     python3-setproctitle \
     python3-yaml \
     unzip \
     vulkan-tools \
     wine-stable \
     wine32 \
     wine64 \
     winetricks \
     x11-xserver-utils \
     zenity \
    "

RUN dpkg --add-architecture i386 && \
    apt-get update && \
    apt-get install -y --no-install-recommends $REQUIRED_PACKAGES

RUN <<_INSTALL_LUTRIS
#!/bin/bash
set -e
source /opt/gow/bash-lib/utils.sh

github_download "lutris/lutris" ".assets[]|select(.name|endswith(\".deb\")).browser_download_url" "lutris.deb"
dpkg -i lutris.deb
rm lutris.deb
_INSTALL_LUTRIS


RUN  <<_UPDATE_LUTRIS
  #!/bin/bash
  set -e

EMULATORS_DIR=/Applications
mkdir -p $EMULATORS_DIR
cd $EMULATORS_DIR


 github_download(){
    curl -H "X-GitHub-Api-Version: "$(GITHUB_REST_VERSION)"" \
        https://api.github.com/repos/$1/releases/latest | \
        jq ".assets[]|select(.name|endswith(\"${3}\")).browser_download_url" | \
        xargs wget -O $2
 }

 echo "**** Downloading latest Lutris deb ****"
 github_download "lutris/lutris" "lutris.deb" "all.deb"

 echo "**** Downloading latest ProtonUP-Qt ****"
 github_download "DavidoTek/ProtonUp-Qt" "protonup.AppImage" "x86_64.AppImage"


 apt-get install -y ./lutris.deb

 # Cleanup
  rm lutris.deb

chmod -v -R 777 $EMULATORS_DIR
chmod -v -R a+x $EMULATORS_DIR


_UPDATE_LUTRIS


RUN mkdir -pm 777 /var/lutris/ && \
    mkdir /opt/gow/startup.d/
COPY --chmod=777 scripts/startup.sh /opt/gow/startup-app.sh
COPY --chmod=777 scripts/startup-10-create-dirs.sh /opt/gow/startup.d/10-create-dirs.sh
COPY configs/lutris-system.yml /opt/gow/lutris-system.yml
COPY configs/lutris-lutris.conf /opt/gow/lutris-lutris.conf

ENV XDG_RUNTIME_DIR=/tmp/.X11-unix

ARG IMAGE_SOURCE
LABEL org.opencontainers.image.source=$IMAGE_SOURCE
