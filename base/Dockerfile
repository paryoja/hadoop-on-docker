FROM openjdk:8-jdk as base

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      net-tools \
      curl \
      netcat \
      gnupg \
      libsnappy-dev \
      vim \
    && rm -rf /var/lib/apt/lists/*

RUN curl -O https://dist.apache.org/repos/dist/release/hadoop/common/KEYS
RUN gpg --import KEYS

ARG HADOOP_VERSION
ARG HADOOP_MAJOR_VERSION
ARG SPARK_VERSION
ARG HIVE_VERSION

ENV HADOOP_VERSION ${HADOOP_VERSION}
ENV HADOOP_URL https://archive.apache.org/dist/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz

RUN set -x \
    && curl -fSL "$HADOOP_URL" -o /tmp/hadoop.tar.gz \
    && curl -fSL "$HADOOP_URL.asc" -o /tmp/hadoop.tar.gz.asc \
    && gpg --verify /tmp/hadoop.tar.gz.asc \
    && tar -xvf /tmp/hadoop.tar.gz -C /opt/ \
    && rm /tmp/hadoop.tar.gz*

ENV SPARK_VERSION ${SPARK_VERSION}
ENV HADOOP_MAJOR_VERSION ${HADOOP_MAJOR_VERSION}
ENV SPARK_URL https://mirror.navercorp.com/apache/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_MAJOR_VERSION}.tgz

RUN set -x \
    && curl -fSL "${SPARK_URL}" -o /tmp/spark.tar.gz \
    && tar -xvf /tmp/spark.tar.gz -C /opt/ \
    && rm /tmp/spark.tar.gz*

ENV HIVE_VERSION ${HIVE_VERSION}
ENV HIVE_URL https://mirror.navercorp.com/apache/hive/hive-${HIVE_VERSION}/apache-hive-${HIVE_VERSION}-bin.tar.gz

RUN set -x \
    && curl -fSL "${HIVE_URL}" -o /tmp/hive.tar.gz \
    && tar -xvf /tmp/hive.tar.gz -C /opt/ \
    && rm /tmp/hive.tar.gz*

RUN ln -s /opt/hadoop-$HADOOP_VERSION/etc/hadoop /etc/hadoop

RUN mkdir /opt/hadoop-$HADOOP_VERSION/logs
RUN mkdir /hadoop-data

ENV HADOOP_HOME=/opt/hadoop-$HADOOP_VERSION
ENV HADOOP_CONF_DIR=/etc/hadoop
ENV SPARK_HOME=/opt/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_MAJOR_VERSION}
ENV HIVE_HOME=/opt/apache-hive-${HIVE_VERSION}-bin

ENV MULTIHOMED_NETWORK=1
ENV USER=root
ENV PATH $HADOOP_HOME/bin/:$SPARK_HOME/bin/:${HIVE_HOME}/bin:$PATH


ADD entrypoint.sh /entrypoint.sh

RUN chmod a+x /entrypoint.sh
RUN sed -i 's/\r$//g' /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

# NAMENODE
FROM base as namenode

HEALTHCHECK CMD curl -f http://localhost:50070/ || exit 1

ENV HDFS_CONF_dfs_namenode_name_dir=file:///hadoop/dfs/name
RUN mkdir -p /hadoop/dfs/name
VOLUME /hadoop/dfs/name

ADD namenode/run.sh /run.sh
RUN chmod a+x /run.sh
RUN sed -i 's/\r$//g' /run.sh

# FOR Hadoop 3
EXPOSE 9870
# FOR Hadoop 2
EXPOSE 50070

CMD ["/run.sh"]

# DATANODE
FROM base as datanode

# HEALTHCHECK CMD curl -f http://localhost:9864/ || exit 1
HEALTHCHECK CMD curl -f http://localhost:50075/ || exit 1

ENV HDFS_CONF_dfs_datanode_data_dir=file:///hadoop/dfs/data
RUN mkdir -p /hadoop/dfs/data
VOLUME /hadoop/dfs/data

ADD datanode/run.sh /run.sh
RUN chmod a+x /run.sh
RUN sed -i 's/\r$//g' /run.sh

EXPOSE 9864

CMD ["/run.sh"]

# RESOURCEMANAGER
FROM base as resourcemanager

HEALTHCHECK CMD curl -f http://localhost:8088/ || exit 1

ADD resourcemanager/run.sh /run.sh
RUN chmod a+x /run.sh
RUN sed -i 's/\r$//g' /run.sh

EXPOSE 8088

CMD ["/run.sh"]

# NODEMANAGER
FROM base as nodemanager

HEALTHCHECK CMD curl -f http://localhost:8042/ || exit 1

ADD nodemanager/run.sh /run.sh
RUN chmod a+x /run.sh
RUN sed -i 's/\r$//g' /run.sh

EXPOSE 8042

CMD ["/run.sh"]

# HISTORYSERVER
FROM base as historyserver

HEALTHCHECK CMD curl -f http://localhost:8188/ || exit 1

ENV YARN_CONF_yarn_timeline___service_leveldb___timeline___store_path=/hadoop/yarn/timeline
RUN mkdir -p /hadoop/yarn/timeline
VOLUME /hadoop/yarn/timeline

ADD historyserver/run.sh /run.sh
RUN chmod a+x /run.sh
RUN sed -i 's/\r$//g' /run.sh

EXPOSE 8188

CMD ["/run.sh"]

# CONSOLE
FROM base as console

RUN echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | tee /etc/apt/sources.list.d/sbt.list
RUN echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | tee /etc/apt/sources.list.d/sbt_old.list
RUN curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | apt-key add
RUN apt-get update
RUN apt-get install sbt

WORKDIR /work

CMD ["echo 'This container is for console login. Try docker-compose run console bash'"]
