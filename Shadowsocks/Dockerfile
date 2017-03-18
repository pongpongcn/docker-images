FROM debian:stretch

RUN export DEBIAN_FRONTEND='noninteractive' && \
apt-get update -qq && \
apt-get install -qqy --no-install-recommends shadowsocks-libev && \
apt-get clean && \
rm -rf /var/lib/apt/lists/*

ENV SS_SERVER_HOST 0.0.0.0
ENV SS_SERVER_PORT 8388
ENV SS_PASSWORD barfoo!
ENV SS_METHOD aes-256-cfb
ENV SS_TIMEOUT 60

EXPOSE $SS_SERVER_PORT $SS_SERVER_PORT/udp

ADD start.sh /start.sh
RUN chmod 755 /start.sh

CMD ["sh", "-c", "/start.sh"]
