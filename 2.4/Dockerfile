FROM hypriot/rpi-java:1.8.0

# add our user and group first to make sure their IDs get assigned consistently
RUN groupadd -r elasticsearch && useradd -r -m -g elasticsearch elasticsearch

RUN apt-get update && apt-get install -y \
        ca-certificates \
        wget \
    --no-install-recommends

# grab gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
    && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
    && wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
    && rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true

# install elasticsearch
# distribution built with seccomp ARM support from source code at https://github.com/kitsook/elasticsearch
COPY elasticsearch /usr/share/elasticsearch
RUN chmod +x /usr/share/elasticsearch/bin/elasticsearch

ENV PATH /usr/share/elasticsearch/bin:$PATH

WORKDIR /usr/share/elasticsearch

RUN set -ex \
    && for path in \
        ./data \
        ./logs \
        ./config \
        ./config/scripts \
        ./plugins \
    ; do \
        mkdir -p "$path"; \
        chown -R elasticsearch:elasticsearch "$path"; \
        chmod -R o+w "$path"; \
    done

COPY config ./config

VOLUME /usr/share/elasticsearch/data

COPY docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh

EXPOSE 9200 9300
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["elasticsearch"]
