FROM java:8

MAINTAINER Dino.Korah@redmatter.com

ENV TZ="Europe/London" \
    BAMBOO_VERSION="5.10.1.1" \
    BAMBOO_HOME="/var/atlassian/application-data/bamboo" \
    BAMBOO_INSTALL_DIR="/opt/atlassian/bamboo" \
    # unprivileged users used to run the app
    RUN_USER=daemon \
    RUN_GROUP=daemon

ADD https://www.atlassian.com/software/bamboo/downloads/binary/atlassian-bamboo-${BAMBOO_VERSION}.tar.gz /tmp/files.tar.gz
#COPY atlassian-bamboo-${BAMBOO_VERSION}.tar.gz /tmp/files.tar.gz
COPY https_proxy_server_xml.patch /tmp/

RUN ( \
    export DEBIAN_FRONTEND=noninteractive; \
    export BUILD_DEPS="patch"; \
    export APP_DEPS="libtcnative-1"; \

    set -e -u -x; \

    apt-get update; \
    apt-get -y upgrade; \
    apt-get install -y --no-install-recommends ${APP_DEPS} ${BUILD_DEPS}; \

    mkdir -p ${BAMBOO_INSTALL_DIR}; \
    tar -C ${BAMBOO_INSTALL_DIR} --strip-components=1 -xzf /tmp/files.tar.gz; \
    echo "bamboo.home=${BAMBOO_HOME}" >> ${BAMBOO_INSTALL_DIR}/atlassian-bamboo/WEB-INF/classes/bamboo-init.properties; \

    # setup reverse proxy settings
    # More details here https://confluence.atlassian.com/bamboo/integrating-bamboo-with-apache-http-server-391087370.html
    patch -p1 -d ${BAMBOO_INSTALL_DIR} < /tmp/https_proxy_server_xml.patch; \
    rm /tmp/https_proxy_server_xml.patch; \

    mkdir ${BAMBOO_INSTALL_DIR}/lib/native; \
    ln --symbolic "/usr/lib/x86_64-linux-gnu/libtcnative-1.so" "${BAMBOO_INSTALL_DIR}/lib/native/libtcnative-1.so"; \

    wget -qO- https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.38.tar.gz | \
        tar -xz -O mysql-connector-java-5.1.38/mysql-connector-java-5.1.38-bin.jar >"${BAMBOO_INSTALL_DIR}/lib/mysql-connector-java-5.1.38-bin.jar"; \

    mkdir -p ${BAMBOO_HOME}; \

    chown -R ${RUN_USER}:${RUN_GROUP} ${BAMBOO_INSTALL_DIR} ${BAMBOO_HOME}; \
    chmod -R go-rwx ${BAMBOO_INSTALL_DIR}/logs ${BAMBOO_INSTALL_DIR}/temp ${BAMBOO_INSTALL_DIR}/work; \

    apt-get remove -y $BUILD_DEPS; \
    apt-get clean autoclean; \
    apt-get autoremove --yes; \
    rm -rf /var/lib/{apt,dpkg,cache,log}/; \
)

USER ${RUN_USER}:${RUN_GROUP}

VOLUME ["${BAMBOO_HOME}"]

# HTTP port
EXPOSE 8085
# Remote agent communication
EXPOSE 54663

WORKDIR ${BAMBOO_INSTALL_DIR}

CMD ["bin/start-bamboo.sh", "-fg"]
