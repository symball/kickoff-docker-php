FROM openjdk:8-jre-alpine

RUN addgroup -S neo4j && adduser -S -H -h /var/lib/neo4j -G neo4j neo4j

ENV NEO4J_SHA256=af53823776645e11d04436a513368e7e417b515572d6228da6b2977c8490ffbb \
    NEO4J_TARBALL=neo4j-community-3.4.5-unix.tar.gz \
    NEO4J_EDITION=community
ARG NEO4J_URI=http://dist.neo4j.org/neo4j-community-3.4.5-unix.tar.gz

COPY ./local-package/* /tmp/

ENV SPATIAL_RELEASE 0.25.6-neo4j-3.4.5
ENV SPATIAL_RELEASES_URL https://github.com/neo4j-contrib/m2/blob/master/releases/org/neo4j/neo4j-spatial
ENV SPATIAL_PLUGIN_JAR neo4j-spatial-${SPATIAL_RELEASE}-server-plugin.jar
ENV SPATIAL_PLUGIN_URL ${SPATIAL_RELEASES_URL}/${SPATIAL_RELEASE}/${SPATIAL_PLUGIN_JAR}

ENV APOC_RELEASE 3.4.0.2
ENV APOC_RELEASES_URL https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/download
ENV APOC_PLUGIN_JAR apoc-${APOC_RELEASE}-all.jar
ENV APOC_PLUGIN_URL ${APOC_RELEASES_URL}/${APOC_RELEASE}/${APOC_PLUGIN_JAR}
# --quiet
RUN apk add --no-cache  \
    bash \
    curl \
    tini \
    su-exec \
    # --silent
    && curl --fail  --show-error --location --remote-name ${NEO4J_URI} \
    && echo "${NEO4J_SHA256}  ${NEO4J_TARBALL}" | sha256sum -csw - \
    && tar --extract --file ${NEO4J_TARBALL} --directory /var/lib \
    && mv /var/lib/neo4j-* /var/lib/neo4j \
    && rm ${NEO4J_TARBALL} \
    && mv /var/lib/neo4j/data /data \
    && curl --output /var/lib/neo4j/plugins/${SPATIAL_PLUGIN_JAR} --location "${SPATIAL_PLUGIN_URL}?raw=true" \
    && curl --output /var/lib/neo4j/plugins/${APOC_PLUGIN_JAR} --location "${APOC_PLUGIN_URL}?raw=true" \
    && chown -R neo4j:neo4j /data \
    && chmod -R 777 /data \
    && chown -R neo4j:neo4j /var/lib/neo4j \
    && chmod -R 777 /var/lib/neo4j \
    && ln -s /data /var/lib/neo4j/data \
    && apk del curl

ENV PATH /var/lib/neo4j/bin:$PATH
WORKDIR /var/lib/neo4j
VOLUME /data
COPY docker-entrypoint.sh /docker-entrypoint.sh
EXPOSE 7474 7473 7687

ENTRYPOINT ["/sbin/tini", "-g", "--", "/docker-entrypoint.sh"]
CMD ["neo4j"]