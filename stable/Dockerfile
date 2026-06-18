ARG JDK_VERSION=8
FROM aa8y/core:jdk${JDK_VERSION}

LABEL org.opencontainers.image.authors="https://github.com/aa8y" \
      org.opencontainers.image.description="Apache Spark image on Alpine, bundled with Hadoop client libraries." \
      org.opencontainers.image.licenses="Apache-2.0" \
      org.opencontainers.image.source="https://github.com/aa8y/docker-spark" \
      org.opencontainers.image.title="aa8y/spark" \
      org.opencontainers.image.url="https://hub.docker.com/r/aa8y/spark" \
      org.opencontainers.image.vendor="https://github.com/aa8y"

ARG HADOOP_VERSION=3.3.6
ARG SPARK_VERSION=3.5.8
# Suffix on the Spark "bin-<profile>" tarball. The 2.x line tags this with the
# Hadoop major.minor (`hadoop2.7`), 3.x drops to a single digit (`hadoop3`),
# so we take it as an explicit build-arg rather than trying to derive it.
ARG SPARK_HADOOP_PROFILE=hadoop3

ENV HADOOP_HOME=/opt/hadoop \
    HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop \
    SPARK_HOME=/opt/spark \
    LD_LIBRARY_PATH=/opt/hadoop/lib/native \
    PATH=/opt/hadoop/bin:/opt/spark/bin:${PATH}

# Wires Spark to the bundled Hadoop client jars.
ENV SPARK_DIST_CLASSPATH="${HADOOP_HOME}/etc/hadoop/*:${HADOOP_HOME}/share/hadoop/common/lib/*:${HADOOP_HOME}/share/hadoop/common/*:${HADOOP_HOME}/share/hadoop/hdfs/*:${HADOOP_HOME}/share/hadoop/hdfs/lib/*:${HADOOP_HOME}/share/hadoop/hdfs/*:${HADOOP_HOME}/share/hadoop/yarn/lib/*:${HADOOP_HOME}/share/hadoop/yarn/*:${HADOOP_HOME}/share/hadoop/mapreduce/lib/*:${HADOOP_HOME}/share/hadoop/mapreduce/*:${HADOOP_HOME}/share/hadoop/tools/lib/*"

USER root
# Hadoop client install. Stays root-owned read-only; the spark user reads from
# it via SPARK_DIST_CLASSPATH and doesn't need write access. Progress is
# emitted at every 1 GiB tick so the build log stays readable but a hung
# fetch is still obvious — `-q` silently kept this layer dark. Retry budget
# is intentionally tight (2 tries, 60s read idle) so a slow/dead archive
# mirror fails inside the CI job timeout instead of grinding silently.
RUN mkdir -p "${HADOOP_HOME}" && \
    apk add --no-cache wget && \
    wget -O- --progress=dot:giga --retry-connrefused --waitretry=2 --read-timeout=60 --timeout=30 -t 2 \
      "https://archive.apache.org/dist/hadoop/common/hadoop-${HADOOP_VERSION}/hadoop-${HADOOP_VERSION}.tar.gz" | \
      tar -xz --strip 1 -C "${HADOOP_HOME}/" && \
    apk del --purge wget && \
    rm -rf /var/cache/apk/* "${HADOOP_HOME}/share/doc"

# Spark install. The `bin-${SPARK_HADOOP_PROFILE}` tarball ships Spark plus
# the JARs needed to talk to that Hadoop line; the Hadoop client above
# provides the matching binaries and native libs.
RUN adduser -Ds /bin/bash spark && \
    mkdir -p "${SPARK_HOME}" && \
    apk add --no-cache wget && \
    wget -O- --progress=dot:giga --retry-connrefused --waitretry=2 --read-timeout=60 --timeout=30 -t 2 \
      "https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-${SPARK_HADOOP_PROFILE}.tgz" | \
      tar -xz --strip 1 -C "${SPARK_HOME}/" && \
    apk del --purge wget && \
    rm -rf /var/cache/apk/* && \
    chown -R spark:spark "${SPARK_HOME}"

WORKDIR ${SPARK_HOME}
USER spark
CMD ["bin/spark-class", "org.apache.spark.deploy.master.Master"]
