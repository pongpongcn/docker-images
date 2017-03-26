FROM httpd:2.4

RUN export DEBIAN_FRONTEND='noninteractive' && \
apt-get update -qq && \
apt-get install -qqy --no-install-recommends python-certbot-apache -t jessie-backports
