FROM debian:jessie

RUN export DEBIAN_FRONTEND='noninteractive' && \
apt-get update -qq && \
apt-get install -qqy --no-install-recommends samba

COPY docker-entrypoint.sh /usr/bin/

EXPOSE 137/udp 138/udp 139 445

ENTRYPOINT ["docker-entrypoint.sh"]
