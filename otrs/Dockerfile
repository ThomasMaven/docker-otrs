# OTRS ticketing system docker image for Rocky Linux 9
FROM rockylinux:8
LABEL maintainer="Juan Luis Baptiste <juan@juanbaptiste.tech>"

ENV OTRS_VERSION=6.5.9-01
ENV OTRS_ROOT="/opt/otrs/"
ENV OTRS_BACKUP_DIR="/var/otrs/backups"
ENV OTRS_CONFIG_DIR="${OTRS_ROOT}Kernel"
ENV OTRS_CONFIG_MOUNT_DIR="/config/"
ENV OTRS_SKINS_MOUNT_DIR="/skins/"
ENV SKINS_PATH="${OTRS_ROOT}/var/httpd/htdocs/skins/"
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8

# Enable PowerTools repository (CRB in Rocky Linux 9)
RUN dnf install -y dnf-plugins-core && \
    dnf config-manager --set-enabled powertools
RUN dnf update -y
# Install EPEL and required packages
RUN dnf install -y glibc-langpack-en httpd-devel epel-release https://rpms.remirepo.net/enterprise/remi-release-8.rpm \
    https://rpm.ocsinventory-ng.org/ocsinventory-release-latest.el8.ocs.noarch.rpm
ENV LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8

RUN dnf install -y perl-Data-UUID perl-DateTime-Format-ICal perl-YAML-LibYAML perl-GDGraph perl-Type-Tiny

RUN dnf install -y epel-release && \
    dnf install -y bzip2 cronie httpd mariadb-server mod_perl perl-App-cpanminus \
    perl-core perl-Crypt-SSLeay perl-Net-DNS perl-URI procmail perl-DateTime \
    perl-IO-Socket-SSL perl-XML-Parser perl-Encode-HanExtra perl-GD \
    perl-Mail-IMAPClient perl-Text-CSV_XS perl-DBD-MySQL perl-Moo\
    perl-Crypt-Random-Source perl-Exporter-Tiny perl-Math-Random-ISAAC \
    perl-Math-Random-Secure perl-Module-Find rsyslog supervisor which perl-Encode perl-Clone perl-HTTP-Message perl-HTML-Parser perl-HTTP-Cookies perl-Params-Validate

# Install missing Perl modules using cpanminus

# RUN cpanm --notest --no-lwp LWP::UserAgent
#
#
RUN cpanm --notest --no-lwp iCal::Parser
#  RUN cpanm --notest --no-lwp \
#     GD::Text GD::Graph
#
# # RUN cpanm --notest --no-lwp \
# #     YAML::XS
# RUN cpanm --notest --no-lwp \
#     Types::TypeTiny

# Install Znuny RPM (updated source for OTRS)
RUN curl -O https://download.znuny.org/releases/RPMS/rhel/7/znuny-${OTRS_VERSION}.noarch.rpm && \
    dnf install -y znuny-${OTRS_VERSION}.noarch.rpm && \
    dnf clean all

# Verify module dependencies
RUN /opt/otrs/bin/otrs.CheckModules.pl

# Add scripts and configurations
COPY *.sh /
COPY etc/supervisord.d/otrs.ini /etc/supervisord.d/

# Configure permissions and HTTP setup
RUN chmod 755 /*.sh && \
    cp ${OTRS_ROOT}/var/httpd/htdocs/index.html /var/www/html && \
    chmod 644 /var/www/html/index.html && \
    sed -i 's/\bindex.html\b/& index.pl/' /etc/httpd/conf/httpd.conf && \
    echo "+ : otrs : cron crond" >> /etc/security/access.conf && \
    sed -i -e '/pam_loginuid.so/ s/^#*/#/' /etc/pam.d/crond && \
    sed -i -e "s/^nodaemon=false/nodaemon=true/" /etc/supervisord.conf && \
    cat /etc/supervisord.d/otrs.ini >> /etc/supervisord.conf && \
    sed -i -e '/<ValidateModule>Kernel::System::SysConfig::StateValidate<\/ValidateModule>/ s/^#*/#/' \
        ${OTRS_ROOT}Kernel/Config/Files/XML/Ticket.xml && \
    mkdir -p ${OTRS_ROOT}var/{run,tmp}

# Handle host-mounted volumes for configuration and skins
RUN mv ${OTRS_CONFIG_DIR} / && \
    mv ${SKINS_PATH} / && \
    touch ${OTRS_ROOT}var/tmp/firsttime && \
    ln -sf /dev/stdout /var/log/httpd/access_log && \
    ln -sf /dev/stderr /var/log/httpd/error_log && \
    ln -sf /dev/stdout /var/log/cron

EXPOSE 80
CMD ["/run.sh"]
