#Splunk Connect for Syslog (SC4S) by Splunk, Inc.
#
#To the extent possible under law, the person who associated CC0 with
#Splunk Connect for Syslog (SC4S) has waived all copyright and related or neighboring rights
#to Splunk Connect for Syslog (SC4S).
#
#You should have received a copy of the CC0 legalcode along with this
#work.  If not, see <http://creativecommons.org/publicdomain/zero/1.0/>.
FROM centos:centos8

#ARG RH_ORG
#ARG RH_ACTIVATION

ENV CONFIGURE_FLAGS="--prefix=/opt/syslog-ng --with-ivykis=internal --with-jsonc=system --disable-env-wrapper --disable-memtrace --disable-tcp-wrapper --disable-linux-caps --disable-man-pages --enable-all-modules --enable-force-gnu99 --enable-json --enable-native --enable-python --enable-http --disable-kafka --disable-java --disable-java-modules --disable-spoof_source --disable-sun_streams --disable-sql --disable-pacct --disable-mongodb --disable-amqp --disable-stomp --disable-redis --disable-systemd --disable-geoip --disable-geoip2 --disable-riemann --disable-smtp --disable-snmp_dest --with-python=3 --disable-manpages --enable-dynamic-linking"

ENV DISTCHECK_CONFIGURE_FLAGS="--prefix=/opt/syslog-ng --with-ivykis=internal --with-jsonc=system --disable-env-wrapper --disable-memtrace --disable-tcp-wrapper --disable-linux-caps --disable-man-pages --enable-all-modules --enable-force-gnu99 --enable-json --enable-native --enable-python --enable-http --disable-kafka --disable-java --disable-java-modules --disable-spoof_source --disable-sun_streams --disable-sql --disable-pacct --disable-mongodb --disable-amqp --disable-stomp --disable-redis --disable-systemd --disable-geoip --disable-geoip2 --disable-riemann --disable-smtp --disable-snmp_dest --with-python=3 --disable-manpages --enable-dynamic-linking"

RUN dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
RUN dnf update -y && \
    dnf install -y findutils \
    autoconf \
    automake ca-certificates git libtool pkgconfig bison byacc file \
    flex pcre-devel glib2-devel openssl-devel libcurl-devel \
    net-snmp-devel \
    libuuid-devel cmake make libxslt docbook-style-xsl gcc-c++ tzdata libxml2 sqlite \
    wget curl which bzip2 libsecret python3 python3-devel python3-pip json-c man-pages && \
    dnf --enablerepo=PowerTools -y install autoconf-archive json-c-devel


RUN CRITERION_VERSION=2.3.3 ;\
    cd /tmp/;\
    wget https://github.com/Snaipe/Criterion/releases/download/v${CRITERION_VERSION}/criterion-v${CRITERION_VERSION}.tar.bz2 ;\
    tar xvf /tmp/criterion-v${CRITERION_VERSION}.tar.bz2;cd /tmp/criterion-v${CRITERION_VERSION} ;\
    cmake -DCMAKE_INSTALL_PREFIX=/usr . ;\
    make install ;\
    ldconfig ;\
    rm -rf /tmp/criterion.tar.bz2 /tmp/criterion-v${CRITERION_VERSION}


COPY syslog-ng /work

RUN cd /work;\
    pip3 install -r requirements.txt
RUN cd /work;\
    ./autogen.sh ;\
    ./configure $CONFIGURE_FLAGS

RUN make -j -l 2.5 install


FROM registry.access.redhat.com/ubi8/ubi

RUN dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
RUN dnf update -y && \
    dnf install -y tzdata libdbi libsecret libxml2 sqlite libcurl curl wget openssl json-c python3

RUN curl -fsSL https://goss.rocks/install | GOSS_VER=v0.3.7 sh
COPY goss.yaml /etc/goss.yaml

COPY --from=0 /opt/syslog-ng /opt/syslog-ng
COPY --from=hairyhenderson/gomplate:v3.5.0-slim /gomplate /bin/gomplate

COPY etc/syslog-ng.conf /opt/syslog-ng/etc/syslog-ng.conf
COPY etc/conf.d /opt/syslog-ng/etc/conf.d
COPY etc/go_templates /opt/syslog-ng/etc/go_templates
COPY etc/context_templates /opt/syslog-ng/etc/context_templates
COPY etc/local_config /opt/syslog-ng/etc/local_config

COPY sbin/entrypoint.sh /
RUN mkdir -p /opt/syslog-ng/var/data/disk-buffer
RUN /opt/syslog-ng/sbin/syslog-ng -V

EXPOSE 514
EXPOSE 601/tcp
EXPOSE 6514/tcp

ENTRYPOINT ["/entrypoint.sh", "-F"]

HEALTHCHECK --interval=1s --timeout=6s CMD source scl_source enable rh-python36 ;goss -g /etc/goss.yaml validate