---
title: "Práctica unidad 2"
date: 2022-12-07T18:15:43+01:00
draft: false
---

## Escenario

Vamos a hacer un escenario de vagrant utilizando el siguiente `Vagrantfile`:

```ruby
Vagrant.configure("2") do |config|

config.vm.define :web do |web|
    web.vm.box = "debian/bullseye64"
    web.vm.hostname = "servidor-web-roberto"
    web.vm.synced_folder ".", "/vagrant", disabled: true
    web.vm.network :public_network,
      :dev => "bridge0",
      :mode => "bridge",
      :type => "bridge",
      use_dhcp_assigned_default_route: true
    web.vm.network :private_network,
      :libvirt__network_name => "net1",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.1",
      :libvirt__forward_mode => "veryisolated"
  end
  config.vm.define :bd do |bd|
    bd.vm.box = "debian/bullseye64"
    bd.vm.hostname = "servidor-bd-roberto"
    bd.vm.synced_folder ".", "/vagrant", disabled: true
    bd.vm.network :private_network,
      :libvirt__network_name => "net1",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.2",
      :libvirt__forward_mode => "veryisolated"
  end
end
```

## Configuración de resolución estática

Vamos a configurar la resolución estática de las páginas utilizando la IP pública de la máquina web:

![resolucion](https://i.imgur.com/n78aYSy.png)

## Instalación de un CMS PHP en mi servidor local

En este caso el CMS que vamos a instalar es [Media Wiki](https://www.mediawiki.org/wiki/Manual:Installing_MediaWiki). Ahora vamos a configurar el servidor y la base de datos.

En el servidor web instalamos apache con php:

```bash
apt update
apt install apache2 libapache2-mod-php php php-mysql
```

### Configuración del VirtualHost

Vamos a instalar el cms en `/var/www/mediawiki`, por lo configuramos el vhost en el fichero `etc/apache2/sites-available/mediawiki.conf`:

```apache2
<VirtualHost *:80>
        ServerName www.roberto.org

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/mediawiki

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

Ahora activamos el VirtualHost

```bash
a2ensite mediawiki
systemctl reload apache2
```

### Configuración de la base de datos

En el servidor que va a tener la base de datos, instalamos mariadb:

```bash
apt update
apt install mariadb-server
mariadb -u root
```

Dentro vamos a crear una base de datos para el CMS y un usuario con permisos:

```sql
GRANT ALL PRIVILEGES ON *.* TO 'remoto'@'%'
IDENTIFIED BY 'remoto' WITH GRANT OPTION;

create database mediawiki;
```

Para configurar el acceso remoto, tenemos que modificar en fichero `/etc/mysql/mariadb.conf.d/50-server.cnf`, la siguiente línea:

```bash
bind-address            = 0.0.0.0
```

Y reiniciamos el servicio.

Ahora desde el cliente creamos el siguiente usuario:

```sql
GRANT ALL PRIVILEGES ON *.* TO 'remoto'@'10.0.0.2' IDENTIFIED BY 'remoto' WITH GRANT OPTION;
```

y podemos conectarnos a la base de datos remota con el siguiente comando:

```bash
mysql -u remoto -h 10.0.0.2 --password=remoto
```

### Instalación MediaWiki

Para instalar media wiki, tenemos que descargar la última versión de la página oficial:

```bash
wget https://releases.wikimedia.org/mediawiki/1.38/mediawiki-1.38.4.tar.gz
tar -xf mediawiki-1.38.4.tar.gz
cp -r mediawiki-1.38.4/* /var/www/mediawiki/
chown -R www-data:www-data /var/www/mediawiki/
```

Antes de iniciar la instalación tenemos que instalar los siguientes paquetes:

```bash
apt install php-mbstring php-xml php-intl -y
systemctl restart apache2.service
```

Ahora seguimos la instalación normalmente, pero en la configuración de la base de datos es importante especificar la IP:

![p2-1](https://i.imgur.com/lUVzDxN.png)

Una vez finalizada la configuración inicial, se descarga el fichero `LocalSettings.php`:

![p2-2](https://i.imgur.com/Ecn7gng.png)

Tenemos que moverlo al Document Root de MediaWiki:

```bash
chown www-data:www-data LocalSettings.php
mv LocalSettings.php /var/www/mediawiki/
```

Una vez realizada la configuración, accediendo a `www.roberto.org` aparece la wiki:

![p2-3](https://i.imgur.com/mT4mBno.png)

#### Instalación de un módulo

Tras configurar el tema, vamos a instalar un módulo: [SimpleCalendar](https://www.mediawiki.org/wiki/Extension:SimpleCalendar):

```bash
wget https://extdist.wmflabs.org/dist/extensions/SimpleCalendar-REL1_38-b7a2f05.tar.gz
tar -xf SimpleCalendar-REL1_38-b7a2f05.tar.gz
mv SimpleCalendar /var/www/mediawiki/extensions/
```

Ahora añadimos la siguiente línea al final de `LocalSettings.php`:

```bash
echo "wfLoadExtension( 'SimpleCalendar' );" >> /var/www/mediawiki/LocalSettings.php
```

Una vez hecho, podemos comprobar que está correctamente instalado accediendo a [http://www.roberto.org/index.php/Especial:Versión](http://www.roberto.org/index.php/Especial:Versi%C3%B3n)

![p2-4](https://i.imgur.com/GYSoi3r.png)

Ya instalado, podemos añadir calendarios a las páginas de la wiki con el siguiente bloque:

```php
{{#calendar: year=2022 | month=nov | title="calendario" }}
```

Y quedaría de la siguiente forma:

![p2-5](https://i.imgur.com/bNKXNG6.png)

## Instalación del CMS PHP NextCloud

Primero descargamos la última versión y lo movemos al directorio de apache:

```bash
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip
cp -r nextcloud/ /var/www/
chown -R www-data:www-data /var/www/nextcloud/
```

Ahora configuramos el vhost en el fichero `etc/apache2/sites-available/nextcloud.conf`:

```bash
<VirtualHost *:80>
        ServerName www.cloud.roberto.org        
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/nextcloud        
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined     
</VirtualHost>
```

```bash
a2ensite nextcloud.conf
systemctl reload apache2.service
```

E instalamos los módulos de php necesarios:

```bash
  apt install php-zip php-gd php-curl -y
systemctl reload apache2.service
```

Ahora en la máquina con la base de datos creamos la base de datos para nextcloud:

```bash
create database nextcloud;
```

Tras este paso, podemos acceder a `cloud.roberto.org` para iniciar la instalación, es importante que en la base de datos especifiquemos la ip y el puerto de la máquina con la base de datos:

![p2-6](https://i.imgur.com/dSKyF4C.png)

Una vez finalizada la instalación, ya podremos utilizar nextcloud:

![p2-7](https://i.imgur.com/LKyMGco.png)
