# -*- mode: ruby -*-
# vi: set ft=ruby :
options = {
  :cores => 2,
  :memory => 1536,
}
CENTOS = {
  box: "centos",
  url: "http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.4-x86_64-v20130427.box"
}
OS = CENTOS
$deploy = <<DEPLOY
#!/bin/bash

yum install -y python-devel python-setuptools gcc make python-virtualenv java-1.7.0-openjdk-devel git ntp wget unzip ant

wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
yum install -y jenkins
chkconfig jenkins on

yum install -y http://mirror.ancl.hawaii.edu/linux/epel/6/i386/epel-release-6-8.noarch.rpm
yum install -y PyYAML python-jinja2 python-paramiko
git clone https://github.com/ansible/ansible.git /opt/ansible

chkconfig ntpd on
service ntpd start
ntpdate -u pool.ntp.org

iptables -F
iptables -F -t nat
iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
iptables-save > /etc/sysconfig/iptables
rsync -va /vagrant/jenkins/ /var/lib/jenkins/
chown -R jenkins:jenkins /var/lib/jenkins
service jenkins start
SYNC_COMMAND=/usr/bin/jenkins-sync
cat > $SYNC_COMMAND <<EOF
#!/bin/bash
rsync -va /var/lib/jenkins/ /vagrant/jenkins/ --exclude workspace --exclude builds --exclude nextBuildNumber --exclude lastStable --exclude lastSuccessful --exclude .git
EOF
chmod +x $SYNC_COMMAND
DEPLOY

Vagrant.configure("2") do |config|
    config.vm.box = OS[:box]
    config.vm.box_url = OS[:url]
    config.vm.network :forwarded_port, guest: 80, host: 8080
    config.vm.provider :virtualbox do |v| 
        v.customize [ "modifyvm", :id, "--memory", options[:memory].to_i, "--cpus", options[:cores].to_i]
    end 
    config.vm.hostname = "micro-qa"
    config.vm.provision :shell, :inline => $deploy
end
