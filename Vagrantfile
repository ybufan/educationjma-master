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
  config.vm.box = "bento/centos-8"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  
  config.vm.network "forwarded_port", guest: 9090, host: 19090, host_ip: "127.0.0.1"
  
  # SMTP
  config.vm.network "forwarded_port", guest: 25, host: 1025, host_ip: "127.0.0.1"

  # SUBMISSION
  config.vm.network "forwarded_port", guest: 587, host: 1587, host_ip: "127.0.0.1"

  # IMAP
  config.vm.network "forwarded_port", guest: 143, host: 1143, host_ip: "127.0.0.1"

  # POP3 
  config.vm.network "forwarded_port", guest: 110, host: 1110, host_ip: "127.0.0.1"
  
  #for Apache
  config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  
  # dnsauth 
  config.vm.network "forwarded_port", guest: 54, host: 154, host_ip: "127.0.0.1"
  
  # dnsrecurs
  config.vm.network "forwarded_port", guest: 53, host: 153, host_ip: "127.0.0.1"
  
  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
   config.vm.provision "install and enable software", type: "shell", inline: <<-SHELL
     # Installation block
     yum -y install epel-release
     yum -y install vim cockpit bash-completion postfix dovecot telnet nc
     yum -y install cyrus-sasl cyrus-sasl-plain
     yum -y install pdns pdns-recursor
     yum -y install bind-utils
  	 # Service configuration block
     systemctl enable --now postfix 
     systemctl enable --now dovecot
     systemctl enable --now cockpit.socket
     systemctl enable --now pdns-recursor
     systemctl enable --now pdns
	 systemctl enable --now php:remi-7.4
	 systemctl enable --now mysqld
	 systemctl enable --now httpd
	 systemctl enable --now php-fpm.service
	 # OS configuration block
     hostnamectl set-hostname allinone-by.localhost
	 
	 #Installation of Roundcube Webmail
	 # Installing Apache
	 yum -y install httpd 
	 # Installing mysql and mysql server
	 yum -y install mysql-server mysql
	 # Install PHP
	 yum -y install php-fpm.x86_64 
	 # Installing package repository
	 dnf -y install  https://rpms.remirepo.net/enterprise/remi-release-8.rpm
	 # Install PHP modules required or recommended by Roundcube.
	 dnf -y install  php-ldap php-imagick php-common php-gd php-imap php-json php-curl php-zip php-xml php-mbstring php-bz2 php-intl php-gmp
	 #Creation /var/www
	 mkdir -p /var/www
	 # Mysql Sserver set up
	 mysql -u root -p roundcube < /var/www/roundcube/SQL/mysql.initial.sql
	 # Downloading RoundCube
     wget https://github.com/roundcube/roundcubemail/releases/download/1.4.2/roundcubemail-1.4.2-complete.tar.gz
     # Extracting RoundCube to /var/www
     tar -xf roundcubemail-1.4.2-complete.tar.gz -C /var/www
     # Renaming RoundCube Directory
     mv /var/www/roundcubemail-1.4.2 /var/www/roundcube
	 # Reset PHP module streams.
     dnf -y module reset php
	 # Enable the php:remi-7.4 module stream.
     dnf -y module enable php:remi-7.4
	 #Install puppppet
	 dnf install https://yum.puppetlabs.com/puppet-release-el-8.noarch.rpm
	 yum install puppet



	  # Create a MySQL Database and User for Roundcube
     systemctl start mysqld
     mysql -u root --execute="CREATE DATABASE roundcube DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;"
     mysql -u root --execute="CREATE USER roundcubeuser@localhost IDENTIFIED BY 'password';"
     mysql -u root --execute="GRANT ALL PRIVILEGES ON roundcube.* TO roundcubeuser@localhost;"
     mysql -u root --execute="flush privileges;"
     mysql -u root roundcube < /var/www/roundcube/SQL/mysql.initial.sql
	 
  SHELL
   config.vm.provision "email service config", type: "shell", inline: <<-SHELL
	   chmod 0600 /var/mail/*
     # Setting up inet_interfaces to all(listening all addresses)
     sed -i 's/^\(inet_interfaces\s*=\s*\).*$/\1all/' /etc/postfix/main.cf
     # Enabling sasl authentication
     sed -i "\$a# Custom added options\nsmtpd_sasl_auth_enable = yes" /etc/postfix/main.cf
     # Setting up mbox path
     sed -i "/mail_location\ =\ mbox:~\/mail:INBOX=\/var\/mail\/%u/s/^#//" /etc/dovecot/conf.d/10-mail.conf
     # Removing manager from alias list
     sed -i '/^manager/d' /etc/aliases && newaliases
  SHELL
   config.vm.provision "pdns server config", type: "shell", inline: <<-SHELL
     # Uncomment local-port at pdns config file and changing port value to 54
     sed -i '/local-port=/s/^# //' /etc/pdns/pdns.conf
     sed -i 's/^\(local-port\s*=\s*\).*$/\154/' /etc/pdns/pdns.conf
     # Creating directory for zone file and copying it to his directory
     mkdir -p /var/lib/pdns
	 
	 EOF
     # PHP-MySQL extension
     yum -y install php-mysql
     # SE Linux permission configuration
     chcon -t httpd_sys_content_t /var/www/roundcube/ -R
     chcon -t httpd_sys_rw_content_t /var/www/roundcube/temp/ /var/www/roundcube/logs/ -R
     setfacl -R -m u:apache:rwx /var/www/roundcube/temp/ /var/www/roundcube/logs/
     setsebool -P httpd_can_network_connect 1
     
     
     cat > /var/www/roundcube/config/config.inc.php << EOF
     # Add zone file for domain
     cat > /var/lib/pdns/youdidnotevenimaginethisdomainexists.com.db << EOF
\\$ORIGIN youdidnotevenimaginethisdomainexists.com.
@                      3600 SOA   ns1.allinone-mh.localhost. (
                              postmaster.allinone-mh.localhost.     ; address of responsible party
                              2016072701                 ; serial number
                              3600                       ; refresh period
                              600                        ; retry period
                              604800                     ; expire time
                              1800                     ) ; minimum ttl
                      86400 NS    ns1.p30.dynect.net.
                      86400 NS    ns2.p30.dynect.net.
                      86400 NS    ns3.p30.dynect.net.
                      86400 NS    ns4.p30.dynect.net.
                       3600 MX    10 mx1.allinone-mh.localhost.
                       3600 MX    20 mx2.allinone-mh.localhost.
                       3600 MX    30 mx3.allinone-mh.localhost.
                         60 A     10.0.2.15
EOF
     # Creating custom named config and adding path to him to recursor.conf
     cat > /etc/pdns/named.conf << EOF
zone "youdidnotevenimaginethisdomainexists.com" {
  file "/var/lib/pdns/youdidnotevenimaginethisdomainexists.com.db";
  type master;
};
EOF
    systemctl restart pdns
  SHELL
   # Moving sed commands into separate shell provisioner
   config.vm.provision "pdns recursor config",type: "shell", inline: <<-SHELL
     # Uncomment local-port at pdns-recursor config file
     sed -i '/local-port=/s/^# //' /etc/pdns-recursor/recursor.conf
     sed -i \"\\$abind-config=/etc/pdns/named.conf\" /etc/pdns/pdns.conf
     # Configuring forward-zones
     sed -i '/forward-zones=/s/^# //' /etc/pdns-recursor/recursor.conf
     sed -i 's/^\\(forward-zones\s*=\s*\\).*$/\\1youdidnotevenimaginethisdomainexists.com=127.0.0.1:54/' /etc/pdns-recursor/recursor.conf
    systemctl restart pdns-recursor
  SHELL
  config.vm.provision "user creation",type: "shell", inline: <<-SHELL
     # User configuration block
     useradd engineer 
     usermod -p '$6$xyz$.UccqMWqX8VK4PRzmKTR1woU2y5IgDas9n.XPkhgK8M62yVqI4sLx.Yw2AC5z7t4Ke3NiU7aq7i3Su5QdrRcF1' engineer
     useradd manager
     usermod -p '$6$xyz$PcPt/h72LIQm.YoxBmDLqfpbX1w3vhcJ1LwyYjOaslRr67l0g3ZkE5nKN0c4Ed98wYTvMWvhlGcV7NZorCE2i/' manager
     useradd contractor
     usermod -p '$6$xyz$tlQI91A01E6TWfFL6jqBSSLdzLKJtFyF2aWfdTZyOBUn56UjQbMyecGla5IMGqX./neusxkBsr3IwUGZhTnel0' contractor
     
   SHELL
end
