FROM eclipse-temurin:8-jre-jammy

# ── Versions (pinned to match gradiant/hbase-master = HBase 2.0.1) ──────────
ENV FLUME_VERSION=1.11.0
ENV HBASE_VERSION=2.0.1
ENV FLUME_HOME=/opt/flume
ENV PATH=$FLUME_HOME/bin:$PATH

# ── System deps ──────────────────────────────────────────────────────────────
RUN apt-get update && \
    apt-get install -y --no-install-recommends wget netcat && \
    rm -rf /var/lib/apt/lists/*

# ── Install Flume 1.11 ───────────────────────────────────────────────────────
RUN wget -q "https://archive.apache.org/dist/flume/${FLUME_VERSION}/apache-flume-${FLUME_VERSION}-bin.tar.gz" && \
    tar -xzf "apache-flume-${FLUME_VERSION}-bin.tar.gz" && \
    mv "apache-flume-${FLUME_VERSION}-bin" "${FLUME_HOME}" && \
    rm "apache-flume-${FLUME_VERSION}-bin.tar.gz"

# ── Strip Flume's bundled HBase 1.x JARs & Protobuf ──────────────────────────
# We MUST remove protobuf-java-2.5.0.jar as it conflicts with HBase 2.x
RUN find "${FLUME_HOME}/lib" -name "hbase-*.jar" -delete && \
    find "${FLUME_HOME}/lib" -name "asynchbase-*.jar" -delete && \
    find "${FLUME_HOME}/lib" -name "protobuf-java-2.5.0.jar" -delete

# ── Add HBase 2.0.1 Libraries ────────────────────────────────────────────────
# Added hbase-protocol which is often required by the sink
RUN wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/hbase-client/${HBASE_VERSION}/hbase-client-${HBASE_VERSION}.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/hbase-common/${HBASE_VERSION}/hbase-common-${HBASE_VERSION}.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/hbase-protocol/${HBASE_VERSION}/hbase-protocol-${HBASE_VERSION}.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/hbase-protocol-shaded/${HBASE_VERSION}/hbase-protocol-shaded-${HBASE_VERSION}.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/hbase-zookeeper/${HBASE_VERSION}/hbase-zookeeper-${HBASE_VERSION}.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/hbase-hadoop-compat/${HBASE_VERSION}/hbase-hadoop-compat-${HBASE_VERSION}.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/hbase-hadoop2-compat/${HBASE_VERSION}/hbase-hadoop2-compat-${HBASE_VERSION}.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-common/2.7.7/hadoop-common-2.7.7.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-auth/2.7.7/hadoop-auth-2.7.7.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/hbase-metrics-api/${HBASE_VERSION}/hbase-metrics-api-${HBASE_VERSION}.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/hbase-metrics/${HBASE_VERSION}/hbase-metrics-${HBASE_VERSION}.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/hbase-procedure/${HBASE_VERSION}/hbase-procedure-${HBASE_VERSION}.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/thirdparty/hbase-shaded-miscellaneous/2.1.0/hbase-shaded-miscellaneous-2.1.0.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/thirdparty/hbase-shaded-protobuf/2.1.0/hbase-shaded-protobuf-2.1.0.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/thirdparty/hbase-shaded-netty/2.1.0/hbase-shaded-netty-2.1.0.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/htrace/htrace-core4/4.2.0-incubating/htrace-core4-4.2.0-incubating.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/commons-configuration/commons-configuration/1.10/commons-configuration-1.10.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/commons-lang/commons-lang/2.6/commons-lang-2.6.jar" && \
    # ── FINAL MISSING PIECE: The Shaded Client itself ──
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/org/apache/hbase/hbase-shaded-client/${HBASE_VERSION}/hbase-shaded-client-${HBASE_VERSION}.jar" && \
    wget -q -P "${FLUME_HOME}/lib" \
    "https://repo1.maven.org/maven2/com/google/protobuf/protobuf-java/2.5.0/protobuf-java-2.5.0.jar"

# ── Copy conf (flume.conf + hbase-site.xml) ──────────────────────────────────
COPY conf/ ${FLUME_HOME}/conf/

# ── Netcat source listens on 44444 ───────────────────────────────────────────
EXPOSE 44444

ENV FLUME_AGENT_NAME=agent

# Flume's flume-ng script hardcodes -Xmx20m and ignores JAVA_OPTS.
# We patch the script directly to replace that value.
RUN sed -i 's/-Xmx20m/-Xms256m -Xmx512m/g' ${FLUME_HOME}/bin/flume-ng

CMD flume-ng agent \
    --name ${FLUME_AGENT_NAME} \
    --conf ${FLUME_HOME}/conf \
    --conf-file ${FLUME_HOME}/conf/flume.conf \
    -Dflume.root.logger=INFO,console