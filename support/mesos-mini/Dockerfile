FROM mesos/mesos-centos

RUN yum install -y java-1.8.0-openjdk

# Prepare systemd environment.
ENV container docker

RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
    rm -f /lib/systemd/system/multi-user.target.wants/*; \
    rm -f /etc/systemd/system/*.wants/*; \
    rm -f /lib/systemd/system/local-fs.target.wants/*; \
    rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
    rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
    rm -f /lib/systemd/system/basic.target.wants/*; \
    rm -f /lib/systemd/system/anaconda.target.wants/*; \
    ln -vf /lib/systemd/system/multi-user.target /lib/systemd/system/default.target

RUN for service in\
    console-getty.service\
    dbus.service\
    dbus.socket\
    dev-hugepages.mount\
    getty.target\
    sys-fs-fuse-connections.mount\
    systemd-logind.service\
    systemd-remount-fs.service\
    systemd-vconsole-setup.service\
    ; do systemctl mask $service; done

CMD ["/usr/sbin/init"]

STOPSIGNAL SIGRTMIN+3

# Prepare Mesos environment.
RUN chmod +x /usr/bin/mesos-init-wrapper && \
    rm -f /etc/mesos-master/work_dir && \
    rm -f /etc/mesos-slave/work_dir

COPY mesos_master_environment /etc/default/mesos-master
COPY mesos_agent_environment /etc/default/mesos-agent

# Prepare Marathon environment.
ARG MARATHON_URL=https://downloads.mesosphere.com/marathon/releases/1.5.5/marathon-1.5.5.tgz
ARG MARATHON_INSTALL_DIR=/usr/local/marathon

RUN mkdir -p $MARATHON_INSTALL_DIR && \
    curl -s $MARATHON_URL -o /marathon.tgz && \
    tar -xzvf /marathon.tgz -C $MARATHON_INSTALL_DIR --strip 1 && \
    rm -f /marathon.tgz

COPY marathon.sh $MARATHON_INSTALL_DIR/bin/
COPY marathon.service /usr/lib/systemd/system/marathon.service

RUN systemctl enable mesos-slave && \
    systemctl enable mesos-master && \
    systemctl enable marathon
