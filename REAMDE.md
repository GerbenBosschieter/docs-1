# Basisimplementatie platform Forus
written by: **Jimmy Habing / TeamForus**

## Getting started
### 1. Backend (API)
### 2. Front-end (Koppeling front-end met backend) 
### 3. Koppeling Me App met backend

## API Deployment
Bij deze deployment documentatie wordt gebruik gemaakt van een **TransIP Blade VPS**, deze is voorgeïnstalleerd met een image van **Ubuntu 16.04** met de **OpenSSH package**.

#### Voorbereiding
	$ sudo apt-get update
	$ sudo apt-get upgrade
	
#### PHP7.2 installatie
	$ sudo apt-get install software-properties-common
	$ sudo add-apt-repository ppa:ondrej/php
	$ sudo apt-get install php7.2
	
#### Controleren of PHP goed is geïnstalleerd
	$ php -v

#### PHP extensies installatie
	$ sudo apt-get install php-pear php7.2-curl php7.2-dev php7.2-gd php7.2-mbstring php7.2- zip php7.2-mysql php7.2-xml
	
### MySQL-server installatie
 	$ sudo apt install mysql-server
Tijdens de setup van MySQL moet er een root wachtwoord worden ingevoerd. Dit wachtwoord moet later weer gebruikt worden. Vergeet deze dus niet!

### Check of de nieuwste versie is geïnstalleerd
	$ sudo mysql -u root -p
  
### Installeren Apache2, CertBot
##### *Apache2*
	$ sudo apt-get update
 	$ sudo apt-get install apache2
 	
##### *Certbot*

Certbot zal tijdens de installatie vragen om het domein dat je gekoppeld hebt aan de servers, eveneens geeft de bot een optie om redirects te kiezen. De voorkeur gaat hierbij naar de optie die alles over SSL drukt.

	$ sudo apt-get update
	$ sudo apt-get install software-properties-common 
	$ sudo add-apt-repository ppa:certbot/certbot
	$ sudo apt-get update
	$ sudo apt-get install python-certbot-apache
	$ sudo certbot –apache
	$ sudo certbot --apache certonly
### Download source code
    $ sudo cd /var/www
    $ sudo mkdir forus-backend
    $ sudo chown -R participatieadmin forus-backend
    $ git clone https//:github.com/teamforus/forus- backend.git
    $ cd forus-backend
    $ ls
   
### Composer installatie
**Let op! ga niet uit de forus-backend map!**

	$ sudo apt install composer
#### Update composer
	
	$ sudo composer install
	$ sudo composer update
	$ sudo apt-get update
	$ sudo apt-get upgrade
	
### Database voorbereiden en initialiseren
##### *Genereer een Artisian key*
	$ cp .env.example .env
 	$ php artisan key:generate

**Pas de volgende opties aan: APP-URL (wij gebruikten https://participatieapi.ddns.net) en onze mysql database wachtwoord.**

	$ sudo nano .env
	
	APP_URL=[hostname(if using localhost -> 	http://localhost] DB_PORT=3306
	DB_DATABASE=homestead
	DB_USERNAME=homestead
	DB_PASSWORD=[password]

##### *Aanmaken van een database en een gebruiker*
	
	$ mysql -u root -p
	  mysql> create user homestead identified by 'password';
##### *password moet vervangen worden door het wachtwoord wat in de vorige stap in de .env bestand is gezet, dit is eveneens het mysql database wachtwoord.*

	mysql> create database homestead;
	mysql> grant all privileges on homestead.* to 'homestead';

##### *Artisan migrate*
	$ php artisan migrate:refresh --seed
     
##### *Apache2 config files aanpassen*
##### *Nadat alle bovenstaande stappen voltooid zijn dan kan apache2 geconfigureerd worden.*

##### *Pas de beide config files in /etc/apache2/sites-enabled aan, hieronder zijn 2 voorbeelden te vinden van de door ons gebruikte config files.*

#### *Voorbeeld 000-default.conf*
	<VirtualHost *:80>
			ServerName participatieapi.ddns.net
	
			ServerAdmin webmaster@localhost 			DocumentRoot /var/www/forus-backend/public
			#LogLevel info ssl:warn
			
	<Directory /var/www/forus-backend/public> 		AllowOverride All
	</Directory>
	
		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	
		#Include conf-available/serve-cgi-bin.conf
		RewriteEngine on
		RewriteCond %{SERVER_NAME} 	=participatieapi.ddns.net
		RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
	</VirtualHost>
	
#### *Voorbeeld 000-default-le-ssl.conf*
	<ifModule mod_ssl.c>
	<VirtualHost *:443>
			ServerName participatieapi.ddns.net
			
			ServerAdmin webmaster@localhost
			DocumentRoot /var/www/forus-backend/public
			#LogLevel info ssl:warn

	<Directory /var/www/forus-backend/public> 		AllowOverride All
	</Directory>

		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	
		SSLCertificateFile /etc/letsencrypt/live/participatieapi.ddns.net/fullchain.pem
		SSLCertificateKeyFile /etc/letsencrypt/live/participatieapi.ddns.net/privkey.pem Include /etc/letsencrypt/options-ssl-apache.conf
	</VirtualHost>
	</IfModule>
	
### Apache2 configuratie afronden

##### Restart de apache2 service
	$ sudo systemctl reload apache2

##### Permissies toewijzen
	$ sudo chown participatieadmin:www-data -R .

### Updaten van de API

##### Controleren of er een nieuwe versie aanwezig is
	$ git status

##### Ophalen van een nieuwe versie (mocht deze beschikbaar zijn)
	$ git pull
