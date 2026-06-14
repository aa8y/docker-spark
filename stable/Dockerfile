ARG JDK_VERSION=8
FROM aa8y/core:jdk${JDK_VERSION}

LABEL org.opencontainers.image.authors="https://github.com/aa8y" \
      org.opencontainers.image.description="Apache Spark image on Alpine, bundled with Hadoop client libraries." \
      org.opencontainers.image.licenses="Apache-2.0" \
      org.opencontainers.image.source="https://github.com/aa8y/docker-spark" \
      org.opencontainers.image.title="aa8y/spark" \
      org.opencontainers.image.url="https://hub.docker.com/r/aa8y/spark" \
      org.opencontainers.image.vendor="https://github.com/aa8y"

ARG HADOOP_VERSION=2.7.7
ARG SPARK_VERSION=2.4.0

ENV HADOOP_HOME=/opt/hadoop \
    HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop \
    SPARK_HOME=/opt/spark \
    LD_LIBRARY_PATH=/opt/hadoop/lib/native \
    PATH=/opt/hadoop/bin:/opt/spark/bin:${PATH}

# SPARK_DIST_CLASSPATH is what wires Spark to the bundled Hadoop client jars.
# Defined separately because every entry expands $HADOOP_HOME and that's
# easier to read on one logical line than crammed into the block above.
ENV SPARK_DIST_CLASSPATH="${HADOOP_HOME}/etc/hadoop/*:${HADOOP_HOME}/share/hadoop/common/lib/*:${HADOOP_HOME}/share/hadoop/common/*:${HADOOP_HOME}/share/hadoop/hdfs/*:${HADOOP_HOME}/share/hadoop/hdfs/lib/*:${HADOOP_HOME}/share/hadoop/hdfs/*:${HADOOP_HOME}/share/hadoop/yarn/lib/*:${HADOOP_HOME}/share/hadoop/yarn/*:${HADOOP_HOME}/share/hadoop/mapreduce/lib/*:${HADOOP_HOME}/share/hadoop/mapreduce/*:${HADOOP_HOME}/share/hadoop/tools/lib/*"

USER root
# Hadoop: install the client tarball that matches the major.minor of the
# requested HADOOP_VERSION, then strip docs to keep the layer small. We pull
# from archive.apache.org because the old cs.utah.edu mirror is long dead and
# the archive is the only host guaranteed to keep older Hadoop releases.
RUN adduser -Ds /bin/bash hadoop && \
    mkdir -p "${HADOOP_HOME}" && \
    apk add --no-cache wget && \
    wget -q -O- --retry-connrefused --waitretry=1 --read-timeout=20 --timeout=15 -t 0 \
      "https://archive.apache.org/dist/hadoop/common/hadoop-${HADOOP_VERSION}/hadoop-${HADOOP_VERSION}.tar.gz" | \
      tar -xz --strip 1 -C "${HADOOP_HOME}/" && \
    apk del --purge wget && \
    rm -rf /var/cache/apk/* "${HADOOP_HOME}/share/doc" && \
    chown -R hadoop:hadoop "${HADOOP_HOME}"

# Spark: pick the bin-hadoopX.Y tarball matching the host Hadoop major.minor.
RUN adduser -Ds /bin/bash spark && \
    mkdir -p "${SPARK_HOME}" && \
    apk add --no-cache wget && \
    SPARK_PACKAGE="spark-${SPARK_VERSION}-bin-hadoop$(echo "${HADOOP_VERSION}" | sed 's/\.[0-9]$//g')" && \
    wget -q -O- --retry-connrefused --waitretry=1 --read-timeout=20 --timeout=15 -t 0 \
      "https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/${SPARK_PACKAGE}.tgz" | \
      tar -xz --strip 1 -C "${SPARK_HOME}/" && \
    apk del --purge wget && \
    rm -rf /var/cache/apk/* && \
    chown -R spark:spark "${SPARK_HOME}"

WORKDIR ${SPARK_HOME}
USER spark
CMD ["bin/spark-class", "org.apache.spark.deploy.master.Master"]
