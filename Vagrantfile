# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"

  config.vm.network "forwarded_port", guest: 8080, host: 8088

  config.vm.provider "virtualbox" do |vb|
     vb.memory = "3072"
     vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
     # Provision with a JDK and checkout from Github
     DEBIAN_FRONTEND=noninteractive apt install -y openjdk-8-jdk git mysql-server-5.7 curl
     git clone https://github.com/sakaiproject/sakai.git /opt/sakai-source

     # Download Tomcat
     wget -q https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.34/bin/apache-tomcat-8.5.34.tar.gz
     tar xzf apache-tomcat-8.5.34.tar.gz -C /opt
     rm -rf /opt/tomcat/webapps/*

     # Download Maven
     wget -q https://archive.apache.org/dist/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
     tar xzf apache-maven-3.5.2-bin.tar.gz -C /opt

     # Set higher limits
     echo "fs.file-max = 64000" > /etc/sysctl.d/90-sakai.conf
     #echo "vm.max_map_count=262144" >> /etc/sysctl.d/90-sakai.conf
     #echo "vm.swappiness=0" >> /etc/sysctl.d/90-sakai.conf
     echo "* - nofile 65535" >> /etc/security/limits.conf
     ulimit -n 8096
     sysctl -p

     # Start building Sakai
     cd /opt/sakai-source/
     /opt/apache-maven-3.5.2/bin/mvn -Pmysql -Dmaven.test.skip clean install sakai:deploy-exploded -Dmaven.tomcat.home=/opt/apache-tomcat-8.5.34 > /tmp/sakai-build.log

     # Sakai properties
     mkdir /opt/apache-tomcat-8.5.34/sakai
     curl -o /opt/apache-tomcat-8.5.34/sakai/sakai.properties https://raw.githubusercontent.com/sakaiproject/nightly-config/master/experimental.properties

     # MySQL setup
     mysql -u root -e "CREATE DATABASE sakai character set utf8"
     mysql -u root -e "GRANT ALL PRIVILEGES ON sakai.* TO sakai@localhost IDENTIFIED BY 'passxxxx'"
     echo "hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect" >> /opt/apache-tomcat-8.5.34/sakai/local.properties
     echo "vendor@org.sakaiproject.db.api.SqlService=mysql" >> /opt/apache-tomcat-8.5.34/sakai/local.properties
     echo "driverClassName@javax.sql.BaseDataSource=com.mysql.jdbc.Driver" >> /opt/apache-tomcat-8.5.34/sakai/local.properties
     echo "url@javax.sql.BaseDataSource=jdbc:mysql://127.0.0.1/sakai?useUnicode=true&characterEncoding=UTF-8" >> /opt/apache-tomcat-8.5.34/sakai/local.properties
     echo "username@javax.sql.BaseDataSource=sakai" >> /opt/apache-tomcat-8.5.34/sakai/local.properties
     echo "password@javax.sql.BaseDataSource=passxxxx" >> /opt/apache-tomcat-8.5.34/sakai/local.properties
  SHELL

  config.vm.provision "shell", run: "always", inline: <<-SHELL
     cd /opt/apache-tomcat-8.5.34
     rm -rf work/Catalina logs/*
     bin/catalina.sh start
  SHELL
end
