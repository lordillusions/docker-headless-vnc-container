# This Dockerfile is used to build an headles vnc image based on Ubuntu 
FROM ubuntu:16.04 

MAINTAINER Tobias Schneck "lordillusions@gmail.com" 
ENV REFRESHED_AT 2017-01-04 

LABEL io.k8s.description="Headless VNC Container with Xfce window manager, firefox and chromium" \
      io.k8s.display-name="Headless VNC Container based on Ubuntu" \
      io.openshift.expose-services="5901:xvnc" \
      #io.openshift.expose-services="6901:http,5901:xvnc" \
      io.openshift.tags="vnc, ubuntu, xfce" \
      io.openshift.non-scalable=true

## Connection ports for controlling the UI: 
# VNC port:5901 
# noVNC webport, connect via http://IP:6901/vnc_auto.html?password=vncpassword 
ENV DISPLAY :1 
ENV VNC_PORT 5901 
#ENV NO_VNC_PORT 6901 
#EXPOSE $VNC_PORT $NO_VNC_PORT 
EXPOSE $VNC_PORT

ENV HOME /headless 
WORKDIR $HOME

### Envrionment config 
ENV DEBIAN_FRONTEND noninteractive 
ENV NO_VNC_HOME /$HOME/noVNC 
ENV VNC_COL_DEPTH 24 
ENV VNC_RESOLUTION 1280x1024 
ENV VNC_PW vncpassword 

### Add all install scripts for further steps 
ENV INST_SCRIPTS $HOME/install 
ADD ./src/common/install/ $INST_SCRIPTS/
ADD ./src/ubuntu/install/ $INST_SCRIPTS/
RUN find $INST_SCRIPTS -name '*.sh' -exec chmod a+x {} +

### Install some common tools 
RUN $INST_SCRIPTS/tools.sh

### Install xvnc-server & noVNC - HTML5 based VNC viewer 
RUN $INST_SCRIPTS/tigervnc.sh
#RUN $INST_SCRIPTS/no_vnc.sh

### Install firfox and chrome browser 
RUN $INST_SCRIPTS/firefox.sh
#RUN $INST_SCRIPTS/chrome.sh

### Install xfce UI RUN $INST_SCRIPTS/xfce_ui.sh
ADD ./src/common/xfce/ $HOME/
### configure startup 
ADD ./src/common/scripts $HOME/scripts
RUN $INST_SCRIPTS/set_user_permission.sh
USER 1984 ENTRYPOINT ["scripts/vnc_startup.sh"]
CMD ["--tail-log"]

# Inspired by monokrome/wine 
ENV WINE_MONO_VERSION 0.0.8 
USER root 

# Install some tools required for creating the image 
RUN apt-get update \
	&& apt-get install -y \
		curl \
		unzip \
		ca-certificates
# Install wine and related packages 
RUN dpkg --add-architecture i386 \
		&& apt-get update \
		&& apt-get install -y \
				wine \
				wine32 \
		&& rm -rf /var/lib/apt/lists/*
# Use the latest version of winetricks 
RUN curl -SL 'https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks' -o /usr/local/bin/winetricks \
		&& chmod +x /usr/local/bin/winetricks

# Get latest version of mono for wine 
RUN mkdir -p /usr/share/wine/mono \
	&& curl -SL 'http://sourceforge.net/projects/wine/files/Wine%20Mono/$WINE_MONO_VERSION/wine-mono-$WINE_MONO_VERSION.msi/download' -o /usr/share/wine/mono/wine-mono-$WINE_MONO_VERSION.msi \
	&& chmod +x /usr/share/wine/mono/wine-mono-$WINE_MONO_VERSION.msi

# Wine really doesn't like to be run as root, so let's use a non-root user USER xclient ENV HOME /home/xclient ENV WINEPREFIX /home/xclient/.wine ENV WINEARCH win32 # Use xclient's home dir as working dir WORKDIR /home/xclient
RUN echo "alias winegui='wine explorer /desktop=DockerDesktop,1024x768'" > ~/.bash_aliases 
