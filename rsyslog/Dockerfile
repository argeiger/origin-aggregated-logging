FROM registry.svc.ci.openshift.org/ocp/4.0:base as builder

# get pkg list from rpmspec -q --buildrequires vendored_src/*/*.spec | sort -u
RUN yum install -y --setopt=tsflags=nodocs \
      make redhat-rpm-config rpm-build \
      autoconf automake bison chrpath cyrus-sasl-devel \
      flex gcc gcc-c++ gnutls-devel krb5-devel libcurl-devel \
      libgcrypt-devel libtool libuuid-devel lz4-devel \
      mariadb-devel mariadb-connector-c-devel net-snmp-devel \
      pcre-devel pkgconfig postgresql-devel python-sphinx \
      python3 python3-docutils python3-sphinx \
      systemd-devel zlib-devel && \
    yum clean all

ADD vendored_src/ /vendored_src/
ADD install-from-src.sh /vendored_src/
# liblognorm configure is hardcoded with certain versions of
# aclocal and automake - if certain files are out of date
# with respect to certain other files, make will attempt to
# reconfigure with versions that may not exist on the platform -
# so make sure the file dependencies are such that aclocal.m4
# is newer than its dependencies, and the other files are newer
# than aclocal.m4, so that make will not attempt to
# reconfigure anything
RUN cd /vendored_src && \
    touch liblognorm/liblognorm/aclocal.m4 && \
    touch liblognorm/liblognorm/Makefile.in && \
    touch liblognorm/liblognorm/configure && \
    touch liblognorm/liblognorm/config.h.in && \
    ./install-from-src.sh

FROM registry.svc.ci.openshift.org/ocp/4.0:base

COPY --from=builder /vendored_src/RPMS/ /RPMS/

RUN yum -y install /RPMS/*/*.rpm && \
    rm -rf /RPMS && \
    yum clean all

ADD install.sh /bin/install.sh
ADD rsyslog.sh /bin/rsyslog.sh
ADD uninstall.sh /bin/uninstall.sh

CMD [ "/bin/rsyslog.sh" ]
