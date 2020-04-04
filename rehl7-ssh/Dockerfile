FROM registry.access.redhat.com/rhel7

ARG RELEASE_VERSION="1.0.0"

RUN subscription-manager register --username radra --password Superzebra86 --force --auto-attach
RUN subscription-manager repos --enable rhel-7-server-ansible-2.9-rpms

# ------------------------------------------------------------------------------
# - Import the RPM GPG keys for repositories
# - Base install of required packages
# - Install supervisord (used to run more than a single process)
# - Install supervisor-stdout to allow output of services started by
#  supervisord to be easily inspected with "docker logs".
# ------------------------------------------------------------------------------
RUN rpm --rebuilddb \
	&& rpm --import \
		https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7 \
	&& rpm --import \
		https://dl.iuscommunity.org/pub/ius/IUS-COMMUNITY-GPG-KEY \
	&& yum -y install \
			--setopt=tsflags=nodocs \
			--disableplugin=fastestmirror \
      https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm \
		epel-release \
	&& yum -y install \
			--setopt=tsflags=nodocs \
			--disableplugin=fastestmirror \
		inotify-tools\
		openssh-clients \
		openssh-server \
		openssl \
		python-setuptool \
		sudo \
		yum-plugin-versionlock \
	&& yum versionlock add \
		inotify-tools \
		openssh \
		openssh-server \
		openssh-clients \
		python-setuptools \
		sudo \
		yum-plugin-versionlock \
	&& yum clean all \
	&& easy_install \
		'supervisor == 4.0.4' \
		'supervisor-stdout == 0.1.1' \
	&& mkdir -p \
		/var/log/supervisor/ \
	&& rm -rf /etc/ld.so.cache \
	&& rm -rf /sbin/sln \
	&& rm -rf /usr/{{lib,share}/locale,share/{man,doc,info,cracklib,i18n},{lib,lib64}/gconv,bin/localedef,sbin/build-locale-archive} \
	&& rm -rf /{root,tmp,var/cache/{ldconfig,yum}}/* \
	&& > /etc/sysconfig/i18n

# ------------------------------------------------------------------------------
# Copy files into place
# ------------------------------------------------------------------------------
ADD src /

# ------------------------------------------------------------------------------
# Provisioning
# - UTC Timezone
# - Networking
# - Configure SSH defaults for non-root public key authentication
# - Enable the wheel sudoers group
# - Replace placeholders with values in systemd service unit template
# - Set permissions
# ------------------------------------------------------------------------------
RUN ln -sf \
		/usr/share/zoneinfo/UTC \
		/etc/localtime \
	&& echo "NETWORKING=yes" \
		> /etc/sysconfig/network \
	&& sed -i \
		-e 's~^PasswordAuthentication yes~PasswordAuthentication no~g' \
		-e 's~^#PermitRootLogin yes~PermitRootLogin no~g' \
		-e 's~^#UseDNS yes~UseDNS no~g' \
		-e 's~^\(.*\)/usr/libexec/openssh/sftp-server$~\1internal-sftp~g' \
		/etc/ssh/sshd_config \
	&& sed -i \
		-e 's~^# %wheel\tALL=(ALL)\tALL~%wheel\tALL=(ALL) ALL~g' \
		-e 's~\(.*\) requiretty$~#\1requiretty~' \
		/etc/sudoers \
	&& sed -i \
		-e "s~{{RELEASE_VERSION}}~${RELEASE_VERSION}~g" \
		/etc/systemd/system/ssh@.service \
	&& chmod 644 \
		/etc/{supervisord.conf,supervisord.d/{20-sshd-bootstrap,50-sshd-wrapper}.conf} \
	&& chmod 700 \
		/usr/{bin/healthcheck,sbin/{scmi,sshd-{bootstrap,wrapper},system-{timezone,timezone-wrapper}}}

EXPOSE 22

# ------------------------------------------------------------------------------
# Set default environment variables
# ------------------------------------------------------------------------------
ENV \
	ENABLE_SSHD_BOOTSTRAP="true" \
	ENABLE_SSHD_WRAPPER="true" \
	ENABLE_SUPERVISOR_STDOUT="false" \
	SSH_AUTHORIZED_KEYS="" \
	SSH_CHROOT_DIRECTORY="%h" \
	SSH_INHERIT_ENVIRONMENT="false" \
	SSH_PASSWORD_AUTHENTICATION="false" \
	SSH_SUDO="ALL=(ALL) ALL" \
	SSH_USER="app-admin" \
	SSH_USER_FORCE_SFTP="false" \
	SSH_USER_HOME="/home/%u" \
	SSH_USER_ID="500:500" \
	SSH_USER_PASSWORD="" \
	SSH_USER_PASSWORD_HASHED="false" \
	SSH_USER_PRIVATE_KEY="" \
	SSH_USER_SHELL="/bin/bash" \
	SYSTEM_TIMEZONE="UTC"

# ------------------------------------------------------------------------------
# Set image metadata
# ------------------------------------------------------------------------------
LABEL \
	install="docker run \
--rm \
--privileged \
--volume /:/media/root \
jdeathe/regl-ssh:${RELEASE_VERSION} \
/usr/sbin/scmi install \
--chroot=/media/root \
--name=\${NAME} \
--tag=${RELEASE_VERSION} \
--setopt='--volume {{NAME}}.config-ssh:/etc/ssh'" \
	uninstall="docker run \
--rm \
--privileged \
--volume /:/media/root \
/usr/sbin/scmi uninstall \
--chroot=/media/root \
--name=\${NAME} \
--tag=${RELEASE_VERSION} \
--setopt='--volume {{NAME}}.config-ssh:/etc/ssh'"

HEALTHCHECK \
	--interval=1s \
	--timeout=1s \
	--retries=5 \
	CMD ["/usr/bin/healthcheck"]

CMD ["/usr/bin/supervisord", "--configuration=/etc/supervisord.conf"]