# Projet Infrastructure et SI

   [Client]
       |
       |
   [Reverse Proxy (Nginx)]
       |         
   [VM1: Wordpress,Wekan]  


VM1:Serveur Web

 - Distribution : Debian 12

    Wordpress :
    
    - Adresse IP : 192.168.1.10
    - Service : Apache , PHP , Wordpress , Wekan 
    - Sous domaine : devb12024.com (Wordpress) , devblogb12024.com (Wekan).


 Mise en Œuvre des Bonnes Pratiques

Sécurité :

TLS/SSL : Utilisation de certificats auto-signés pour HTTPS :

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt

 Configurations à Réaliser :

 Installation de Debian 12.04 :

    - Télécharger l'ISO depuis le site officiel.
    
    - Suivre l'assistant d'installation pour une configuration de base.

Installation de apache :

    - Mettre à jour le cache des paquets et Installation de Apache:

    sudo apt-get update
    sudo apt-get install -y apache2
    sudo systemctl enable apache2

    -Activation des modules d'Apache :

    sudo a2enmod rewrite
    sudo a2enmod deflate
    sudo a2enmod headers
    sudo a2enmod ssl

    sudo systemctl restart apache2

    -Installation de PHP :

    sudo apt-get install -y php
    
    sudo apt-get install -y php-pdo php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath

    - Installation de MySQL/MariaDB

    sudo apt-get install -y mariadb-server

    sudo mariadb-secure-installation(configuration)

    -Installation de Wordpress 

    cd /tmp
    wget https://wordpress.org/latest.zip

    -Creation de la Base de données et Configuration :

    mysql –u root –p

    CREATE DATABASE wordpress;
    
    CREATE USER 'adminwp'@'localhost' IDENTIFIED BY 'changeme';

    GRANT ALL PRIVILEGES ON wordpress.* TO adminwp@localhost;

    FLUSH PRIVILEGES;
 
    exit

    sudo chown -R www-data:www-data /var/www/html/

    Installation de Wekan

    
    -Installation de Wekan

    sudo apt-get install snapd

    sudo snap install wekan  

Configuration des Services :

    - Apache pour Wordpress :

    sudo nano /etc/apache2/sites-available/devb12024.com.conf 

  
<VirtualHost *:443>
ServerName devb12024.com

    DocumentRoot /var/www/html/wordpress
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/devb12024.crt
    SSLCertificateKeyFile /etc/ssl/private/devb12024.key
    
    <Directory /var/www/html/wordpress>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
<VirtualHost>


    sudo a2ensite devb12024.com
    sudo systemctl reload apache2

    - Nginx pour Wekan 

    sudo nano /etc/nginx/sites-enable/devblogb12024.com

    server {
listen 8443 ssl;
server_name devblogb12024.com;

    ssl_certificate /etc/ssl/certs/devblogb12024.crt;
    ssl_certificate_key /etc/ssl/private/devblogb12024.key;
    
    location / {
        proxy_pass http://10.0.2.15:8082;  # Modifiez selon votre configuration Wekan
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

}
