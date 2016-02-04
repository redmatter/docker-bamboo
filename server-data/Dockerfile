FROM debian:jessie

MAINTAINER Dino.Korah@redmatter.com

ENV TZ="Europe/London" \
    BAMBOO_HOME="/var/atlassian/application-data/bamboo"
    # unprivileged users used to run the app
    RUN_USER=daemon \
    RUN_GROUP=daemon

RUN \
    mkdir -p ${BAMBOO_HOME} && \
    chown ${RUN_USER}:${RUN_GROUP} ${BAMBOO_HOME} && \
    ln -s ${BAMBOO_HOME} /bamboo

VOLUME ["${BAMBOO_HOME}"]

