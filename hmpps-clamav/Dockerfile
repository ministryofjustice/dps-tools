FROM centos:latest

ENV CLAM_VERSION=0.103.2

COPY talos.pub /tmp/talos.pub

#Adding this for kubectl - used by the CronJob
COPY kubernetes.repo /etc/yum.repos.d/kubernetes.repo

RUN yum update -y -q

RUN yum install -y -q kubectl libcurl-devel gcc-c++ openssl-devel wget make

RUN wget -nv https://www.clamav.net/downloads/production/clamav-${CLAM_VERSION}.tar.gz && \
    wget -nv https://www.clamav.net/downloads/production/clamav-${CLAM_VERSION}.tar.gz.sig &&  \
    gpg --import /tmp/talos.pub && \
    gpg --decrypt clamav-${CLAM_VERSION}.tar.gz.sig && \
    tar xzf clamav-${CLAM_VERSION}.tar.gz

RUN cd clamav-${CLAM_VERSION} && \
    ./configure && \
    make && make install

RUN rm -rf /clamav-${CLAM_VERSION} && \
    rm -rf /tmp/talos.pub && \
    yum remove -y -q wget make gcc-c++ openssl-devel kernel-headers

RUN yum clean all

# Add clamav user
RUN groupadd --gid 1000 clamav && \
    useradd -r -g clamav -u 1000 clamav -d /var/lib/clamav && \
    mkdir -p /var/lib/clamav && \
    mkdir /usr/local/share/clamav && \
    chown -R clamav:clamav /var/lib/clamav /usr/local/share/clamav

# Configure Clam AV...
RUN chown clamav:clamav -R /usr/local/etc/
COPY --chown=clamav:clamav ./*.conf /usr/local/etc/
COPY --chown=clamav:clamav eicar.com /
COPY --chown=clamav:clamav ./readyness.sh /

# permissions
RUN mkdir /var/run/clamav && \
    chown clamav:clamav /var/run/clamav && \
    chmod 750 /var/run/clamav

USER 1000

RUN freshclam --no-dns && \
    freshclam

VOLUME /var/lib/clamav

COPY --chown=clamav:clamav docker-entrypoint.sh /

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 3310
