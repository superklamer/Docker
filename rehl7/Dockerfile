FROM registry.access.redhat.com/rhel7
ARG USERNAME
ARG PASSWORD

USER root

WORKDIR /usr/src/app

RUN subscription-manager register --username ${USERNAME} --password ${PASSWORD} --force --auto-attach
RUN subscription-manager repos --enable rhel-7-server-ansible-2.9-rpms
RUN yum remove python-requests -y
RUN yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;


RUN yum install -y \ 
    git \
    ansible \
    python2-pip \
    gcc \
    python-devel \
    krb5-devel \
    krb5-workstation \
    cifs-utils \
    unzip \
    openssh-server \
    iputils \
    vim

RUN systemctl enable sshd

RUN pip install --upgrade pip
COPY requirements.txt ./
RUN pip install --force-reinstall --ignore-installed --upgrade -r requirements.txt
RUN subscription-manager remove --all && \
    subscription-manager unregister && \
    subscription-manager clean

CMD [ "/usr/sbin/init"]
