# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "alpine/alpine64"

  config.vm.network "forwarded_port", guest: 8080, host: 8088

  config.vm.provider "virtualbox" do |vb|
     vb.memory = "3072"
     vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
     # Provision with a JDK and checkout from Github
     apk add openjdk8 maven git htop linux-pam
     git clone https://github.com/sakaiproject/sakai.git /opt/sakai-source

     # Download Tomcat
     wget -q https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.34/bin/apache-tomcat-8.5.34.tar.gz
     tar xzf apache-tomcat-8.5.34.tar.gz -C /opt
     rm -rf /opt/tomcat/webapps/*

     # Set higher limits
     echo "fs.file-max = 64000" > /etc/sysctl.d/90-sakai.conf
     echo "vm.max_map_count=262144" >> /etc/sysctl.d/90-sakai.conf
     echo "vm.swappiness=0" >> /etc/sysctl.d/90-sakai.conf

     mkdir /etc/security
     echo "* - nofile 65535" > /etc/security/limits.conf
     echo "* - memlock unlimited" >> /etc/security/limits.conf
     ulimit -n 65535
     sysctl -w kernel.grsecurity.chroot_deny_chmod=0
     sysctl -p

     # Start building Sakai
     cd /opt/sakai-source/
     ulimit -a
     mvn -Dmaven.test.skip clean install sakai:deploy-exploded -Dmaven.tomcat.home=/opt/apache-tomcat-8.5.34 > /tmp/sakai-build.log
  SHELL

  config.vm.provision "shell", run: "always", inline: <<-SHELL
     cd /opt/apache-tomcat-8.5.34
     rm -rf work/Catalina logs/*
     bin/catalina.sh start
  SHELL
end
