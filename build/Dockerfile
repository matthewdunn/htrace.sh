### BEG SCRIPT INFO
#
# Header:
#
#         fname : "Dockerfile"
#         cdate : "12.07.2018"
#        author : "Michał Żurawski <trimstray@gmail.com>"
#      tab_size : "2"
#     soft_tabs : "yes"
#
# Description:
#
#   This Dockerfile builds a static htrace.sh in a Docker container.
#
#   - converted Dockerfile to Alpine Linux
#     author: https://github.com/davidneudorfer
#
#   For build:
#     cd htrace.sh && build/build.sh
#
#   For init:
#     docker run --rm -it --name htrace.sh htrace.sh -u https://nmap.org -h
#
#   For debug:
#     docker exec -it htrace.sh /bin/bash
#     docker run --rm -it --entrypoint /bin/bash --name htrace.sh htrace.sh
#
# License:
#
#   htrace.sh, Copyright (C) 2018  Michał Żurawski
#
#   This program is free software: you can redistribute it and/or modify
#   it under the terms of the GNU General Public License as published by
#   the Free Software Foundation, either version 3 of the License, or
#   (at your option) any later version.
#
#   This program is distributed in the hope that it will be useful,
#   but WITHOUT ANY WARRANTY; without even the implied warranty of
#   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
#   GNU General Public License for more details.
#
#   You should have received a copy of the GNU General Public License
#   along with this program. If not, see <http://www.gnu.org/licenses/>.
#
### END SCRIPT INFO

FROM golang:alpine AS golang
RUN apk update && apk add --no-cache git
RUN go get github.com/ssllabs/ssllabs-scan
RUN go get github.com/maxmind/geoipupdate/cmd/geoipupdate

FROM alpine:latest AS builder
RUN \
  apk add --no-cache composer php7-curl php7-xml php7-dom \
  && rm -rf /var/cache/apk/* \
  && composer global require bramus/mixed-content-scan

FROM drwetter/testssl.sh:stable AS testssl

FROM alpine:latest

MAINTAINER trimstray "trimstray@gmail.com"

RUN \
  apk add --no-cache \
  bash \
  bc \
  bind-tools \
  ca-certificates \
  coreutils \
  curl \
  drill \
  git \
  gnupg \
  ncurses \
  nmap \
  nmap-nselibs \
  nmap-scripts \
  openssl \
  procps \
  unzip \
  wget \
  php \
  jq \
  libmaxminddb \
  && rm -rf /var/cache/apk/*

RUN \
  apk add --no-cache nodejs npm && \
  rm -rf /var/cache/apk/* && \
  npm config set unsafe-perm true && \
  npm install -g observatory-cli

COPY --from=golang /go/bin/ssllabs-scan /usr/bin/ssllabs-scan
COPY --from=golang /go/bin/geoipupdate /usr/bin/geoipupdate
COPY --from=testssl /usr/local/bin/testssl.sh /usr/bin/testssl.sh
COPY --from=testssl /home/testssl/etc/ /etc/testssl/etc/

RUN \
  ln -s /root/.composer/vendor/bramus/mixed-content-scan/bin/mixed-content-scan /usr/bin/mixed-content-scan

RUN \
  mkdir -p /usr/local/etc/ && \
  echo -en "AccountID 0\\nLicenseKey 000000000000\\nEditionIDs GeoLite2-Country GeoLite2-City" > /usr/local/etc/GeoIP.conf

RUN \
  mkdir -p /usr/local/share/GeoIP/ && \
  geoipupdate

RUN \
  cp -R /usr/local/share/GeoIP /usr/share/

ENV TESTSSL_INSTALL_DIR /etc/testssl

WORKDIR /opt/htrace.sh

COPY bin /opt/htrace.sh/bin/
COPY lib /opt/htrace.sh/lib/
COPY log /opt/htrace.sh/log/
COPY src /opt/htrace.sh/src/
COPY static /opt/htrace.sh/static/
COPY dependencies.sh setup.sh /opt/htrace.sh/

RUN ./setup.sh install

ENTRYPOINT ["/usr/local/bin/htrace.sh"]
CMD ["--help"]
