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
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "hashicorp/precise64"

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
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

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
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = false
  
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    
    # Java
    sudo apt-get update
    sudo apt-get -y --no-install-recommends install apt-utils software-properties-common python-software-properties curl ppa-purge
    sudo add-apt-repository ppa:openjdk-r/ppa
    sudo apt-get update
    sudo apt-get -y --no-install-recommends install openjdk-8-jre
    sudo update-alternatives --config java

    # Elastic
    sudo wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
    sudo echo "deb http://packages.elastic.co/elasticsearch/1.7/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-1.7.list 
    sudo apt-get update 
    sudo apt-get -y install elasticsearch 
    sudo update-rc.d elasticsearch defaults 95 10 
    service elasticsearch restart
     
    # Install PostgreSQL and Git
    sudo apt-get install -y postgresql git

    # Add php7 repo and update
    sudo ppa-purge ppa:ondrej/php-7.0
    sudo add-apt-repository ppa:ondrej/php -y
    
    sudo apt-get update

    # Install PHP and its extensions
    sudo apt-get install -y --force-yes php7.0-cli php7.0-dev \
    php-pgsql php-sqlite3 php-gd php-apcu php7.0-curl php7.0-mcrypt \
    php-imap php7.0-gd php-memcached php7.0-pgsql php7.0-readline \
    php-xdebug php-mbstring php7.0-xml php7.0-zip php7.0-intl php7.0-bcmath

    # Install Composer
    curl -sS https://getcomposer.org/installer | php
    mv composer.phar /usr/local/bin/composer
    # Add Composer executable to PATH if you're using Vagrant
    printf "\nPATH=\"$(composer config -g home 2>/dev/null)/vendor/bin:\$PATH\"\n" | tee -a /home/vagrant/.profile

    # Configure PHP CLI
    sudo sed -i "s/error_reporting = .*/error_reporting = E_ALL/" /etc/php/7.0/cli/php.ini
    sudo sed -i "s/display_errors = .*/display_errors = On/" /etc/php/7.0/cli/php.ini
    sudo sed -i "s/memory_limit = .*/memory_limit = 512M/" /etc/php/7.0/cli/php.ini
    sudo sed -i "s/;date.timezone.*/date.timezone = UTC/" /etc/php/7.0/cli/php.ini

    # Install Nginx & PHP-FPM
    sudo apt-get install -y --force-yes nginx php7.0-fpm
   #  # If you don't have any websites which using files below
   #  rm /etc/nginx/sites-enabled/default
   #  rm /etc/nginx/sites-available/default
    service nginx restart

    # Configure PHP-FPM
    sudo sed -i "s/error_reporting = .*/error_reporting = E_ALL/" /etc/php/7.0/fpm/php.ini
    sudo sed -i "s/display_errors = .*/display_errors = On/" /etc/php/7.0/fpm/php.ini
    sudo sed -i "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/" /etc/php/7.0/fpm/php.ini
    sudo sed -i "s/memory_limit = .*/memory_limit = 512M/" /etc/php/7.0/fpm/php.ini
    sudo sed -i "s/upload_max_filesize = .*/upload_max_filesize = 100M/" /etc/php/7.0/fpm/php.ini
    sudo sed -i "s/post_max_size = .*/post_max_size = 100M/" /etc/php/7.0/fpm/php.ini
    sudo sed -i "s/;date.timezone.*/date.timezone = UTC/" /etc/php/7.0/fpm/php.ini

    service nginx restart
    service php7.0-fpm restart

    # PostgreSQL database and user setup
    su - postgres
    psql -d template1 -U postgres

    CREATE USER vipa WITH PASSWORD 'vipa';
    CREATE DATABASE vipa;
    GRANT ALL PRIVILEGES ON DATABASE vipa to vipa;
    \q

    su
    cd $OLDPWD

    sudo apt-get autoremove
    sudo apt-get update
    sudo apt-get -y purge nodejs npm
    sudo apt-get install -y build-essential

    # Node & Bower
    curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
    sudo apt-get -y install nodejs nodejs-legacy
    # sudo apt-get -y install npm already installed 
    sudo npm config set registry http://registry.npmjs.org/
    sudo npm config set strict-ssl false
    sudo npm install -g bower

    # Create the directory and set permissions
    sudo mkdir -p /var/www
    sudo chown -R www-data:www-data /var/www \
    && su -s /bin/bash www-data \
    && cd /var/www \
    && rm -rf html \
    && git clone https://github.com/tugrulcan/vipa.git \
    && cd vipa \
    && echo "{}" > ~/.composer/composer.json \
    && cp /vagrant/config.json  /root/.composer/config.json \
    && composer update -vvv -o \
    && bower update --allow-root \
    && php app/console assets:install web --symlink \
    && php app/console assetic:dump \
    && php app/console doctrine:schema:drop --force \
    && php app/console doctrine:schema:create \
    && php app/console vipa:install \
    && php app/console vipa:install:samples

    sudo mkdir /etc/nginx/sites-available/vipa/ 
    sudo touch /etc/nginx/sites-available/vipa/nginx.conf
    cp /vagrant/nginx.conf /etc/nginx/sites-available/vipa/nginx.conf
    service nginx restart

    sudo su -s /bin/bash www-data \
    && cd /var/www/vipa \
    && app/console vipa:install:package citation

  SHELL
end
