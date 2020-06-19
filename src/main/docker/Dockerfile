FROM registry.access.redhat.com/ubi8/ubi-minimal:latest
ARG MAVEN_VERSION=3.6.1
ARG GRAAL_VM_VERSION=19.1.1
ENV USER_HOME_DIR="/root"
ARG SHA=b4880fb7a3d81edd190a029440cdf17f308621af68475a4fe976296e71ff4a4b546dd6d8a58aaafba334d309cc11e638c52808a4b0e818fc0fd544226d952544
ARG MAVEN_BASE_URL=https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries
ARG GRAAL_VM_BASE_URL=https://github.com/oracle/graal/releases/download/vm-${GRAAL_VM_VERSION}

ARG PKGS="tar zip gzip findutils wget curl unzip gcc glibc-devel zlib-devel"

USER root

RUN mkdir -p /usr/share/maven /usr/share/maven/ref \ 
    && microdnf -y install --nodocs $PKGS \
    && curl -fsSL -o /tmp/apache-maven.tar.gz ${MAVEN_BASE_URL}/apache-maven-$MAVEN_VERSION-bin.tar.gz \
    && echo "${SHA}  /tmp/apache-maven.tar.gz" | sha512sum -c - \
    && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
    && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn \
    && mkdir -p /opt/graalvm  \
    && curl -fsSL -o /tmp/graalvm-ce-amd64.tar.gz ${GRAAL_VM_BASE_URL}/graalvm-ce-linux-amd64-${GRAAL_VM_VERSION}.tar.gz \
    && tar -xzf /tmp/graalvm-ce-amd64.tar.gz -C /opt/graalvm --strip-components=1  \
    && /opt/graalvm/bin/gu --auto-yes install native-image \
    && rm -f /tmp/apache-maven.tar.gz  /tmp/graalvm-ce-amd64.tar.gz \
    && mkdir -p /project

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"
ENV GRAALVM_HOME /opt/graalvm
ENV JAVA_HOME /opt/graalvm
ENV WORK_DIR /project
ENV PATH $PATH:$JAVA_HOME/bin

RUN $MAVEN_HOME/bin/mvn package

WORKDIR /work/
COPY target/*-runner /work/application
RUN chmod 775 /work
EXPOSE 8080
CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
