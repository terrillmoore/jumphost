# Use phusion/baseimage as base image. To make your builds reproducible, make
# sure you lock down to a specific version, not to `latest`!
# See https://github.com/phusion/baseimage-docker/blob/master/Changelog.md for
# a list of version numbers.
ARG PHUSION_VERSION

FROM phusion/baseimage:$PHUSION_VERSION

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]

# patch ubuntu
RUN apt-get update && apt-get upgrade -y -o Dpkg::Options::="--force-confold"

# install packages
RUN apt-get install -y sudo

# enable SSH
RUN rm -f /etc/service/sshd/down && /etc/my_init.d/00_regen_ssh_host_keys.sh

# these must be after FROM, as post-FROM any ARGs are forgotten.
# since they're not used until now, we might as well put them late, 
# making rebuilds quicker.
ARG SYSTEM_USER
ARG ANSIBLE_USER

# copy in the root public key(s)
COPY id_jumphost_admin.pub id_jumphost_system.pub id_jumphost_ansible.pub /tmp/

# create the remote system user and ansible user
RUN useradd -ms /bin/bash -p '*' $SYSTEM_USER && \
    sudo -u $SYSTEM_USER mkdir -m700 /home/$SYSTEM_USER/.ssh && \
    sudo -u $SYSTEM_USER touch /home/$SYSTEM_USER/.ssh/authorized_keys && \
    useradd -ms /bin/bash -p '*' $ANSIBLE_USER && \
    sudo -u $ANSIBLE_USER mkdir -m700 /home/$ANSIBLE_USER/.ssh && \
    sudo -u $ANSIBLE_USER touch /home/$ANSIBLE_USER/.ssh/authorized_keys && \
    cat /tmp/id_jumphost_admin.pub >> /root/.ssh/authorized_keys && \
    cat /tmp/id_jumphost_system.pub >> /home/$SYSTEM_USER/.ssh/authorized_keys && \
    cat /tmp/id_jumphost_ansible.pub >> /home/$ANSIBLE_USER/.ssh/authorized_keys && \
    rm /tmp/id_jumphost_*.pub && \
    printf "PasswordAuthentication no\nUsePAM no\n" >> /etc/ssh/sshd_config

# ...put your own build instructions here...

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
