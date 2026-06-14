FROM aa8y/core:jdk8

LABEL org.opencontainers.image.authors="https://github.com/aa8y" \
      org.opencontainers.image.description="Apache Spark image on Alpine, bundled with Hadoop client libraries." \
      org.opencontainers.image.licenses="Apache-2.0" \
      org.opencontainers.image.source="https://github.com/aa8y/docker-spark" \
      org.opencontainers.image.title="aa8y/spark" \
      org.opencontainers.image.url="https://hub.docker.com/r/aa8y/spark" \
      org.opencontainers.image.vendor="https://github.com/aa8y"

USER root

# Hadoop
# TODO: Move this to a different image this can use as a base.
ENV HADOOP_HOME /opt/hadoop
ENV HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
ENV PATH $PATH:$HADOOP_HOME/bin
ENV LD_LIBRARY_PATH $LD_LIBRARY_PATH:$HADOOP_HOME/lib/native

ARG HADOOP_VERSION=2.7.7

RUN adduser -Ds /bin/bash hadoop \
 && mkdir -p $HADOOP_HOME \
 && apk add --no-cache --update wget \
 && wget -q -O- --retry-connrefused --waitretry=1 --read-timeout=20 --timeout=15 -t 0 \
      "http://apache.cs.utah.edu/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz" | \
      tar -xz --strip 1 -C $HADOOP_HOME/ \
 && apk del --purge wget \
 && rm -rf /var/cache/apk/* $HADOOP_HOME/share/doc \
 && chown -R hadoop:hadoop $HADOOP_HOME


# Spark
ENV SPARK_HOME /opt/spark
ENV SPARK_DIST_CLASSPATH="$HADOOP_HOME/etc/hadoop/*:$HADOOP_HOME/share/hadoop/common/lib/*:$HADOOP_HOME/share/hadoop/common/*:$HADOOP_HOME/share/hadoop/hdfs/*:$HADOOP_HOME/share/hadoop/hdfs/lib/*:$HADOOP_HOME/share/hadoop/hdfs/*:$HADOOP_HOME/share/hadoop/yarn/lib/*:$HADOOP_HOME/share/hadoop/yarn/*:$HADOOP_HOME/share/hadoop/mapreduce/lib/*:$HADOOP_HOME/share/hadoop/mapreduce/*:$HADOOP_HOME/share/hadoop/tools/lib/*"
ENV PATH $PATH:${SPARK_HOME}/bin

ARG SPARK_VERSION=2.3.1

RUN adduser -Ds /bin/bash spark \
 && mkdir -p $SPARK_HOME \
 && apk add --no-cache --update wget \
 && export SPARK_PACKAGE=spark-${SPARK_VERSION}-bin-hadoop$(echo $HADOOP_VERSION | sed 's/\.[0-9]$//g') \
 && wget -q -O- --retry-connrefused --waitretry=1 --read-timeout=20 --timeout=15 -t 0 \
      "https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/${SPARK_PACKAGE}.tgz" | \
      tar -xz --strip 1 -C $SPARK_HOME/ \
 && apk del --purge wget \
 && rm -rf /var/cache/apk/* \
 && chown -R spark:spark $SPARK_HOME

WORKDIR $SPARK_HOME
USER spark
CMD ["su", "-c", "bin/spark-class org.apache.spark.deploy.master.Master", "spark"]
