VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT
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
    # PostgreSQL database and user setup

    # fix permissions
    echo "-------------------- fixing listen_addresses on postgresql.conf"
    sudo sed -i "s/#listen_address.*/listen_addresses '*'/" /etc/postgresql/9.1/main/postgresql.conf
    sudo cp /vagrant/pg_hba.conf /etc/postgresql/9.1/main/pg_hba.conf
    sudo service postgresql restart

      echo '===== Creating PostgreSQL databases and users'
      ### DO NOT CHANGE THE TABS!!!
su postgres << EOF
psql -c "CREATE USER vipa WITH PASSWORD 'vipa'"
EOF
su postgres << EOF
psql -c "CREATE DATABASE vipa;"
EOF
su postgres << EOF
psql -c "GRANT ALL PRIVILEGES ON DATABASE vipa to vipa"
EOF
    sudo service postgresql restart

    
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
    && git clone https://github.com/academic/vipa.git \
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
    sudo cp /vagrant/nginx.conf /etc/nginx/conf.d/default.conf
    service nginx restart

    sudo su -s /bin/bash www-data \
    && cd /var/www/vipa \
    && app/console vipa:install:package citation

SCRIPT


Vagrant.configure("2") do |config|

  config.vm.box = "hashicorp/precise64"
  
  # speed up apt-get
  config.cache.auto_detect = true

  config.vm.network :forwarded_port, guest: 5432, host: 5432
  config.vm.network :forwarded_port, guest: 9000, host: 9000
  config.vm.network :forwarded_port, guest: 80, host: 81

  config.vm.provision "shell", inline: $script

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "2048"
  end

end
