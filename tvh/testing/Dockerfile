FROM ubuntu-debootstrap:14.04
MAINTAINER dreamcat4 <dreamcat4@gmail.com>


# Choose from: master, unstable, testing, or stable
ENV tvh_release=testing


ENV _clean="rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*"
ENV _apt_clean="eval apt-get clean && $_clean"


# Install s6-overlay
ENV s6_overlay_version="1.10.0.3"
ADD https://github.com/just-containers/s6-overlay/releases/download/v${s6_overlay_version}/s6-overlay-amd64.tar.gz /tmp/
RUN tar zxf /tmp/s6-overlay-amd64.tar.gz -C / && $_clean
ENV S6_LOGGING="1"
# ENV S6_KILL_GRACETIME="3000"


# Install pipework
ADD https://github.com/jpetazzo/pipework/archive/master.tar.gz /tmp/pipework-master.tar.gz
RUN tar -zxf /tmp/pipework-master.tar.gz -C /tmp && cp /tmp/pipework-master/pipework /sbin/ && $_clean


# Install xmltv & tvheadend
RUN apt-get update -qq && apt-get install -qqy apt-transport-https \
 && apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 379CE192D401AB61 DB87EE44DD382349 \
 && echo deb https://dl.bintray.com/dreamcat4/ubuntu ${tvh_release} main > /etc/apt/sources.list.d/tvheadend.list \
 && echo deb http://ppa.launchpad.net/alex-tomlins/xmltv/ubuntu trusty main > /etc/apt/sources.list.d/xmltv.list \
 && apt-get update -qq && apt-get install -qqy bzip2 libavahi-client3 xmltv \
 && apt-get update -qq && DEBIAN_FRONTEND=noninteractive apt-get install -qqy tvheadend \
 && rm -rf /home/hts && $_apt_clean


# Relocate the timezone file
RUN mkdir -p /config/.etc && mv /etc/timezone /config/.etc/ && ln -s /config/.etc/timezone /etc/


# Relocate the locale files
RUN mkdir -p /config/.var/lib/locales/ /config/.usr/lib/ \
 && mv /var/lib/locales/supported.d /config/.var/lib/locales/ \
 && mv /usr/lib/locale /config/.usr/lib/ \
 && ln -s /config/.var/lib/locales/supported.d /var/lib/locales/ \
 && ln -s /config/.usr/lib/locale /usr/lib/


# Global config
ADD config/backup/unknown.tar.bz2+ /config/
ADD config/dvr/recordings+ /config/
ADD config/users/admin+ /config/


# Configure hts home dir and /recordings folders
RUN usermod -a -G video hts -d /config && install -o hts -g video -d /recordings


# Launch script
ADD entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh


# Default container settings
VOLUME /recordings
EXPOSE 9981 9982
ENTRYPOINT ["/init","/entrypoint.sh","-u","hts","-g","video","-c","/config"]

