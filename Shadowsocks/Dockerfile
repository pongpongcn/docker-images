FROM debian:stretch

RUN export DEBIAN_FRONTEND='noninteractive' && \
apt-get update -qq && \
apt-get install -qqy --no-install-recommends shadowsocks-libev && \
apt-get clean

EXPOSE 8388 8388/udp