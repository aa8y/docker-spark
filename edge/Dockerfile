### Spark edge tarball builder.
ARG JDK_VERSION=8
FROM aa8y/core:jdk${JDK_VERSION} AS builder

ENV AMPLAB_JENKINS=true

ARG HADOOP_VERSION=2.7.7
ARG SCALA_VERSION=2.11
ARG SPARK_BRANCH=master

USER root
WORKDIR /tmp
RUN apk add --no-cache \
      ca-certificates \
      git \
      maven \
      py-setuptools \
      python \
      wget && \
    rm -rf /var/cache/apk/*
RUN git clone -b "${SPARK_BRANCH}" https://github.com/apache/spark.git && \
    cd spark && \
    ./dev/change-scala-version.sh "${SCALA_VERSION}" && \
    if [ "${SPARK_BRANCH}" = "branch-1.6" ]; then \
      DISTRIBUTOR=./make-distribution.sh && \
      export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=512m"; \
    else \
      DISTRIBUTOR=./dev/make-distribution.sh; \
    fi && \
    "${DISTRIBUTOR}" \
      --name edge \
      --tgz \
      "-Phadoop-$(echo "${HADOOP_VERSION}" | sed 's/\.[0-9]$//g')" \
      -Phive \
      -Phive-thriftserver \
      -Pmesos \
      -Pyarn

### Spark edge tarball consumer.
FROM aa8y/core:jdk${JDK_VERSION}

LABEL org.opencontainers.image.authors="https://github.com/aa8y" \
      org.opencontainers.image.description="Apache Spark image built from source (edge), on Alpine, bundled with Hadoop client libraries." \
      org.opencontainers.image.licenses="Apache-2.0" \
      org.opencontainers.image.source="https://github.com/aa8y/docker-spark" \
      org.opencontainers.image.title="aa8y/spark (edge)" \
      org.opencontainers.image.url="https://hub.docker.com/r/aa8y/spark" \
      org.opencontainers.image.vendor="https://github.com/aa8y"

ARG HADOOP_VERSION=2.7.7
ARG SPARK_VERSION=2.4.0

ENV HADOOP_HOME=/opt/hadoop \
    HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop \
    SPARK_HOME=/opt/spark \
    LD_LIBRARY_PATH=/opt/hadoop/lib/native \
    PATH=/opt/hadoop/bin:/opt/spark/bin:${PATH}
ENV SPARK_DIST_CLASSPATH="${HADOOP_HOME}/etc/hadoop/*:${HADOOP_HOME}/share/hadoop/common/lib/*:${HADOOP_HOME}/share/hadoop/common/*:${HADOOP_HOME}/share/hadoop/hdfs/*:${HADOOP_HOME}/share/hadoop/hdfs/lib/*:${HADOOP_HOME}/share/hadoop/hdfs/*:${HADOOP_HOME}/share/hadoop/yarn/lib/*:${HADOOP_HOME}/share/hadoop/yarn/*:${HADOOP_HOME}/share/hadoop/mapreduce/lib/*:${HADOOP_HOME}/share/hadoop/mapreduce/*:${HADOOP_HOME}/share/hadoop/tools/lib/*"

USER root
RUN adduser -Ds /bin/bash hadoop && \
    mkdir -p "${HADOOP_HOME}" && \
    apk add --no-cache wget && \
    wget -q -O- --retry-connrefused --waitretry=1 --read-timeout=20 --timeout=15 -t 0 \
      "https://archive.apache.org/dist/hadoop/common/hadoop-${HADOOP_VERSION}/hadoop-${HADOOP_VERSION}.tar.gz" | \
      tar -xz --strip 1 -C "${HADOOP_HOME}/" && \
    apk del --purge wget && \
    rm -rf /var/cache/apk/* "${HADOOP_HOME}/share/doc" && \
    chown -R hadoop:hadoop "${HADOOP_HOME}"

RUN adduser -Ds /bin/bash spark && \
    mkdir -p "${SPARK_HOME}"
WORKDIR ${SPARK_HOME}
COPY --from=builder /tmp/spark/spark-${SPARK_VERSION}-SNAPSHOT-bin-edge.tgz ./spark-edge.tgz
RUN tar -xzf spark-edge.tgz --strip 1 -C ./ && \
    chown -R spark:spark ./ && \
    rm ./spark-edge.tgz

USER spark
CMD ["bin/spark-class", "org.apache.spark.deploy.master.Master"]
