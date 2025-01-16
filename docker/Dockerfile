#### Build Stage ####
ARG BASE_IMAGE="ubuntu:20.04"
FROM $BASE_IMAGE AS base_layer

### Environment config
ARG BG_IMG=darkweb_back.png
ARG EXTRA_SH=noop.sh
ARG DISTRO=ubuntu
ARG LANG='en_US.UTF-8'
ARG LANGUAGE='en_US:en'
ARG LC_ALL='en_US.UTF-8'
ARG TZ='America/Vancouver'
ARG USER='dark'
ENV DEBIAN_FRONTEND=noninteractive \
    DISTRO=$DISTRO \
    HOME=/home/kasm-default-profile \
    INST_SCRIPTS=/dockerstartup/install \
    KASM_VNC_PATH=/usr/share/kasmvnc \
    LANG=$LANG \
    LANGUAGE=$LANGUAGE \
    LC_ALL=$LC_ALL \
    TZ=$TZ \
    STARTUPDIR=/dockerstartup

### Home setup
WORKDIR $HOME
RUN mkdir -p $HOME/Desktop

### Setup package rules
COPY ./src/ubuntu/install/package_rules $INST_SCRIPTS/package_rules/
RUN bash $INST_SCRIPTS/package_rules/package_rules.sh && rm -rf $INST_SCRIPTS/package_rules/

### Install common tools
COPY ./src/ubuntu/install/tools $INST_SCRIPTS/tools/
RUN bash $INST_SCRIPTS/tools/install_tools.sh && rm -rf $INST_SCRIPTS/tools/

### Copy over the maximization script to our startup dir for use by app images.
COPY ./src/ubuntu/install/maximize_script $STARTUPDIR/

### Install custom fonts
COPY ./src/ubuntu/install/fonts $INST_SCRIPTS/fonts/
RUN bash $INST_SCRIPTS/fonts/install_custom_fonts.sh && rm -rf $INST_SCRIPTS/fonts/

### Install xfce UI
COPY ./src/ubuntu/install/xfce $INST_SCRIPTS/xfce/
RUN bash $INST_SCRIPTS/xfce/install_xfce_ui.sh && rm -rf $INST_SCRIPTS/xfce/
ADD ./src/$DISTRO/xfce/.config/ $HOME/.config/
RUN mkdir -p /usr/share/extra/backgrounds/
RUN mkdir -p /usr/share/extra/icons/
ADD /src/common/resources/images/darkweb_back.png  /usr/share/backgrounds/darkweb_back.png
ADD /src/common/resources/images/$BG_IMG  /usr/share/backgrounds/bg_default.png
ADD /src/common/resources/images/icon_ubuntu.png /usr/share/extra/icons/icon_ubuntu.png
ADD /src/common/resources/images/icon_ubuntu.png /usr/share/extra/icons/icon_default.png
ADD /src/common/resources/images/icon_kasm.png /usr/share/extra/icons/icon_kasm.png
ADD /src/common/resources/images/egress_info.svg /usr/share/extra/icons/egress_info.svg
ADD /src/common/resources/images/egress_error.svg /usr/share/extra/icons/egress_error.svg
ADD /src/common/resources/images/egress_offline.svg /usr/share/extra/icons/egress_offline.svg

### Install kasm_vnc dependencies and binaries
COPY ./src/ubuntu/install/kasm_vnc $INST_SCRIPTS/kasm_vnc/
RUN bash $INST_SCRIPTS/kasm_vnc/install_kasm_vnc.sh && rm -rf $INST_SCRIPTS/kasm_vnc/
COPY ./src/common/install/kasm_vnc/kasmvnc.yaml /etc/kasmvnc/

### Install Kasm Profile Sync
COPY ./src/ubuntu/install/profile_sync $INST_SCRIPTS/profile_sync/
RUN bash $INST_SCRIPTS/profile_sync/install_profile_sync.sh

### configure startup
COPY ./src/common/scripts/kasm_hook_scripts $STARTUPDIR
ADD ./src/common/startup_scripts $STARTUPDIR
RUN bash $STARTUPDIR/set_user_permission.sh $STARTUPDIR $HOME && \
    echo 'source $STARTUPDIR/generate_container_user' >> $HOME/.bashrc

### extra configurations needed per distro variant
COPY ./src/ubuntu/install/extra $INST_SCRIPTS/extra/
RUN bash $INST_SCRIPTS/extra/$EXTRA_SH  && rm -rf $INST_SCRIPTS/extra/

### Sysbox support
COPY ./src/ubuntu/install/sysbox $INST_SCRIPTS/sysbox/
RUN bash $INST_SCRIPTS/sysbox/install_systemd.sh && rm -rf $INST_SCRIPTS/sysbox/

# Install Tor Browser
COPY ./src/ubuntu/install/torbrowser $INST_SCRIPTS/torbrowser/
RUN bash $INST_SCRIPTS/torbrowser/install_torbrowser.sh  && rm -rf $INST_SCRIPTS/torbrowser/

# Install Firefox
COPY ./src/ubuntu/install/firefox/ $INST_SCRIPTS/firefox/
COPY ./src/ubuntu/install/firefox/firefox.desktop $HOME/Desktop/
RUN bash $INST_SCRIPTS/firefox/install_firefox.sh && rm -rf $INST_SCRIPTS/firefox/

# Copy all the Docs to the container
COPY ./src/ubuntu/install/files $HOME/Desktop/

### Create user and home directory for base images that don't already define it
RUN (groupadd -g 1000 dark \
    && useradd -M -u 1000 -g 1000 dark \
    && usermod -a -G dark dark) ; exit 0
ENV HOME=/home/dark
WORKDIR $HOME
RUN mkdir -p $HOME && chown -R 1000:0 $HOME


RUN mkdir -p ${HOME}/.vnc
COPY ./src/common/resources/certificate ${HOME}/.vnc/
RUN chown -R 1000:0 $HOME

### FIX PERMISSIONS ## Objective is to change the owner of non-home paths to root, remove write permissions, and set execute where required
# these files are created on container first exec, by the default user, so we have to create them since default will not have write perm
RUN touch $STARTUPDIR/wm.log \
    && touch $STARTUPDIR/window_manager_startup.log \
    && touch $STARTUPDIR/vnc_startup.log \
    && touch $STARTUPDIR/no_vnc_startup.log \
    && chown -R root:root $STARTUPDIR \
    && find $STARTUPDIR -type d -exec chmod 755 {} \; \
    && find $STARTUPDIR -type f -exec chmod 644 {} \; \
    && find $STARTUPDIR -type f -iname "*.sh" -exec chmod 755 {} \; \
    && find $STARTUPDIR -type f -iname "*.py" -exec chmod 755 {} \; \
    && find $STARTUPDIR -type f -iname "*.rb" -exec chmod 755 {} \; \
    && find $STARTUPDIR -type f -iname "*.pl" -exec chmod 755 {} \; \
    && find $STARTUPDIR -type f -iname "*.log" -exec chmod 666 {} \; \
    && chmod 755 $STARTUPDIR/generate_container_user \
    && rm -rf $STARTUPDIR/install \
    && chown -R root:root /usr/local/bin \
    && rm -Rf /home/kasm-default-profile/.launchpadlib

### Cleanup job
COPY ./src/ubuntu/install/cleanup $INST_SCRIPTS/cleanup/
RUN bash $INST_SCRIPTS/cleanup/cleanup.sh && rm -rf $INST_SCRIPTS/cleanup/

#### Runtime Stage ####
FROM scratch
COPY --from=base_layer / /

### Environment config
ARG DISTRO=ubuntu
ARG EXTRA_SH=noop.sh
ARG LANG='en_US.UTF-8'
ARG LANGUAGE='en_US:en'
ARG LC_ALL='en_US.UTF-8'
ARG START_XFCE4=1
ARG TZ='America/Vancouver'
ENV DEBIAN_FRONTEND=noninteractive \
    DISPLAY=:1 \
    DISTRO=$DISTRO \
    GOMP_SPINCOUNT=0 \
    HOME=/home/dark \
    INST_SCRIPTS=/dockerstartup/install \
    KASMVNC_AUTO_RECOVER=true \
    KASM_VNC_PATH=/usr/share/kasmvnc \
    LANG=$LANG \
    LANGUAGE=$LANGUAGE \
    LC_ALL=$LC_ALL \
    LD_LIBRARY_PATH=/opt/libjpeg-turbo/lib64/:/usr/local/lib/ \
    LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:/usr/lib/i386-linux-gnu${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}:/usr/local/nvidia/lib:/usr/local/nvidia/lib64 \
    MAX_FRAME_RATE=24 \
    NO_VNC_PORT=6080 \
    OMP_WAIT_POLICY=PASSIVE \
    SHELL=/bin/bash \
    STARTUPDIR=/dockerstartup \
    START_XFCE4=$START_XFCE4 \
    TERM=xterm \
    VNC_COL_DEPTH=24 \
    VNCOPTIONS="-PreferBandwidth -DynamicQualityMin=4 -DynamicQualityMax=7 -DLP_ClipDelay=0" \
    VNC_PORT=5900 \
    VNC_PORT=5900 \
    VNC_PW=vncpassword \
    VNC_USER=vncuser \
    VNC_RESOLUTION=1280x1024 \
    VNC_RESOLUTION=1280x720 \
    VNC_VIEW_ONLY_PW=vncviewonlypassword \
    TZ=$TZ

### Ports and user
EXPOSE $VNC_PORT \
       $NO_VNC_PORT
WORKDIR $HOME
USER 1000

ENTRYPOINT ["/dockerstartup/kasm_default_profile.sh", "/dockerstartup/vnc_startup.sh", "/dockerstartup/kasm_startup.sh"]
CMD ["--wait"]
