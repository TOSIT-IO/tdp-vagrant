FROM ubuntu:24.04

RUN userdel --remove ubuntu

# Install the necessary programs
RUN apt-get update && apt-get install -y \
  libvirt-dev \
  libvirt-clients \
  ssh \
  gosu \
  pip \
  wget \
  && rm -rf /var/lib/apt/lists/*

# Download and install vagrant 2.4.1
RUN wget https://releases.hashicorp.com/vagrant/2.4.1/vagrant_2.4.1-1_amd64.deb && \
  dpkg -i vagrant_2.4.1-1_amd64.deb && \
  rm vagrant_2.4.1-1_amd64.deb

# Download vagrant-libvirt dependencies' gem file and
# install vagrant-libvirt and its dependencies in .vagrant.d
ENV VAGRANT_HOME=/home/tdp/.vagrant.d

RUN wget \
  https://rubygems.org/downloads/xml-simple-1.1.9.gem \
  https://rubygems.org/downloads/nokogiri-1.16.7.gem \
  https://rubygems.org/downloads/ruby-libvirt-0.8.4.gem \
  https://rubygems.org/downloads/formatador-1.1.0.gem \
  https://rubygems.org/downloads/fog-core-2.5.0.gem \
  https://rubygems.org/downloads/fog-xml-0.1.4.gem \
  https://rubygems.org/downloads/fog-json-1.2.0.gem \
  https://rubygems.org/downloads/fog-libvirt-0.12.2.gem \
  https://rubygems.org/downloads/diffy-3.4.3.gem \
  https://rubygems.org/downloads/mini_portile2-2.8.8.gem && \
  vagrant plugin install vagrant-libvirt --plugin-version=0.12.2 && \
  rm *.gem

# Set tdp as working directory
WORKDIR /home/tdp

# Install Ansible-Core 
RUN pip install ansible-core==2.15.1 --break-system-packages --no-cache

# Use docker-entrypoint.sh as entrypoint
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
