FROM redis:3.2.9

MAINTAINER Luke Liang <lukeliang@outlook.com>

EXPOSE 26379
ADD sentinel.conf /data/sentinel.conf
RUN chmod 777 /data/sentinel.conf
ENV SENTINEL_QUORUM 2
ENV SENTINEL_DOWN_AFTER 30000
ENV SENTINEL_FAILOVER 180000
COPY sentinel-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/sentinel-entrypoint.sh
WORKDIR /data