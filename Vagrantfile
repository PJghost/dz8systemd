# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos8"


  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
   config.vm.provision "shell", inline: <<-SHELL
     echo -e '# Configuration file for my watchlog service\n# Place it to /etc/sysconfig\n# File and word in that file that we will be monit\nWORD="ALERT"\nLOG=/var/log/watchlog.log' > /etc/sysconfig/watchlog
     echo -e '#!/bin/bash\nWORD=$1\nLOG=$2\nDATE=`date`\nif grep $WORD $LOG &> /dev/null\nthen\nlogger "$DATE: I found word, Master!"\nelse\nexit 0\nfi' > /opt/watchlog.sh
     chmod +x /opt/watchlog.sh
     echo -e '[Unit]\nDescription=My watchlog service\n[Service]\nType=oneshot\nEnvironmentFile=/etc/sysconfig/watchlog\nExecStart=/opt/watchlog.sh $WORD $LOG' > /etc/systemd/system/watchlog.service
     echo -e "[Unit]\nDescription=Run watchlog script every 30 second\n[Timer]\n# Run every 30 second\nOnActiveSec=1sec\n#OnBootSec=1min\n#OnUnitActiveSec=30sec\nOnCalendar=*:*:0/30\nAccuracySec=1us\nUnit=watchlog.service\n[Install]\nWantedBy=multi-user.target" > /etc/systemd/system/watchlog.timer
     systemctl enable watchlog.timer
     systemctl start watchlog.timer
     sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
     sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
     yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y
     sed -i 's/#SOCKET/SOCKET/g' /etc/sysconfig/spawn-fcgi
     sed -i 's/#OPTIONS/OPTIONS/g' /etc/sysconfig/spawn-fcgi
     echo -e '[Unit]\nDescription=Spawn-fcgi startup service by Otus\nAfter=network.target\n\n[Service]\nType=simple\nPIDFile=/var/run/spawn-fcgi.pid\nEnvironmentFile=/etc/sysconfig/spawn-fcgi\nExecStart=/usr/bin/spawn-fcgi -n $OPTIONS\nKillMode=process\n\n[Install]\nWantedBy=multi-user.target' > /etc/systemd/system/spawn-fcgi.service
     systemctl enable spawn-fcgi
     systemctl start spawn-fcgi
     sed -i 's/Environment=LANG=C/Environment=LANG=C\nEnvironmentFile=\/etc\/sysconfig\/httpd-%I/g' /usr/lib/systemd/system/httpd.service
     echo -e 'OPTIONS=-f conf/first.conf' > /etc/sysconfig/httpd-first
     echo -e 'OPTIONS=-f conf/second.conf' > /etc/sysconfig/httpd-second
     cp -f /etc/httpd/conf/httpd.conf /etc/httpd/conf/first.conf
     cp -f /etc/httpd/conf/httpd.conf /etc/httpd/conf/second.conf
     sed -i -r 's/Listen 80/Listen 8080\nPidFile \/var\/run\/httpd-second.pid/g' /etc/httpd/conf/second.conf
   SHELL
end
