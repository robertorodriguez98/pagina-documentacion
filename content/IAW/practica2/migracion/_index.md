---
title: "Migración a la VPS"
date: 2022-12-07T18:17:31+01:00
draft: false
---

## DNS del dominio

Para configurar el dns tenemos que mirar la dirección en la configuración del VPS:

![p2-8](https://i.imgur.com/MZtPIqH.png)

y añadir un registro CNAME en la dirección [www.admichin.es](www.admichin.es) que lleve a esa dirección:

![p2-9](https://i.imgur.com/hnTB39X.png)

## Configuración del servidor LEMP

Para instalar el servidor lemp tenemos que instalar los siguientes paquetes:

```bash
apt install 
apt install nginx php php-mysql mariadb-server -y
```

Ahora, en la máquina con la base de datos, creamos un fichero con la copia de seguridad y lo enviamos a la vps:

```bash
mysqldump -u remoto -p -x -A > dbs.sql
scp dbs.sql calcetines@nodriza.admichin.es:/home/calcetines
```

Ahora creamos en la base de datos un usuario con permisos y restauramos las bases de datos:

```sql
GRANT ALL PRIVILEGES ON *.* TO 'admin'@localhost IDENTIFIED BY 'contraseña' WITH GRANT OPTION;
```

```bash
mysql --user admin --password < dbs.sql
```

## Migración de las aplicaciones

Para migrar las aplicaciones, tenemos que moverlas al servidor. En este caso utilizaré rsync ya que permite reanudar la transmisión si se interrumpe, además de ser más rápido que scp:

```bash
rsync -avP /var/www/mediawiki/ calcetines@nodriza.admichin.es:/home/calcetines/mediawiki
rsync -avP /var/www/nextcloud/ calcetines@nodriza.admichin.es:/home/calcetines/nextcloud
```

Una vez finalizado, en este caso vamos a utilizar un solo virtualhost, por lo que los directorios de ambas aplicaciones deben estar en `/var/www/html/nombreaplicacion`:

```bash
upstream php-handler {
    #server 127.0.0.1:9000;
    server unix:/var/run/php/php7.4-fpm.sock;
}

# Set the `immutable` cache control options only for assets with a cache busting `v` argument
map $arg_v $asset_immutable {
    "" "";
    default "immutable";
}

server {
    listen 80;
    listen [::]:80;
    server_name www.admichin.es;

    # Prevent nginx HTTP Server Detection
    #server_tokens off;
    rewrite ^/$ /portal;

    # Path to the root of the domain
    root /var/www/html;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location ^~ /.well-known {
        # The rules in this block are an adaptation of the rules
        # in the cloud `.htaccess` that concern `/.well-known`.

        location = /.well-known/carddav { return 301 /cloud/remote.php/dav/; }
        location = /.well-known/caldav  { return 301 /cloud/remote.php/dav/; }

        location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation    { try_files $uri $uri/ =404; }

        # Let cloud's API for `/.well-known` URIs handle all other
        # requests by passing them to the front-end controller.
        return 301 /cloud/index.php$request_uri;
    }
    location ^~ /portal {
        # set max upload size and increase upload timeout:
        client_max_body_size 512M;
        client_body_timeout 300s;
        fastcgi_buffers 64 4K;

        gzip on;
        gzip_vary on;
        gzip_comp_level 4;
        gzip_min_length 256;
        gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
        gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/wasm application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;


        client_body_buffer_size 512k;

        # HTTP response headers borrowed from cloud `.htaccess`
        add_header Referrer-Policy                      "no-referrer"   always;
        add_header X-Content-Type-Options               "nosniff"       always;
        add_header X-Download-Options                   "noopen"        always;
        add_header X-Frame-Options                      "SAMEORIGIN"    always;
        add_header X-Permitted-Cross-Domain-Policies    "none"          always;
        add_header X-Robots-Tag                         "none"          always;
        add_header X-XSS-Protection                     "1; mode=block" always;

        fastcgi_hide_header X-Powered-By;

        index index.php index.html /portal/index.php$request_uri;

        location ~ \.php(?:$|/) {

            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            set $path_info $fastcgi_path_info;

            try_files $fastcgi_script_name =404;

            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $path_info;

            fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
            fastcgi_param front_controller_active true;     # Enable pretty urls
            fastcgi_pass php-handler;

            fastcgi_intercept_errors on;
            fastcgi_request_buffering off;

            fastcgi_max_temp_file_size 0;
        }

        location /portal {
            try_files $uri $uri/ /portal/index.php$request_uri;
        }
    }
    location ~ /\.ht {
          deny all;
         }

    location ^~ /cloud {
        # set max upload size and increase upload timeout:
        client_max_body_size 512M;
        client_body_timeout 300s;
        fastcgi_buffers 64 4K;
        # Enable gzip but do not remove ETag headers
        gzip on;
        gzip_vary on;
        gzip_comp_level 4;
        gzip_min_length 256;
        gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
        gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/wasm application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;
        # HTTP response headers borrowed from cloud `.htaccess`
        add_header Referrer-Policy                      "no-referrer"   always;
        add_header X-Content-Type-Options               "nosniff"       always;
        add_header X-Download-Options                   "noopen"        always;
        add_header X-Frame-Options                      "SAMEORIGIN"    always;
        add_header X-Permitted-Cross-Domain-Policies    "none"          always;
        add_header X-Robots-Tag                         "none"          always;
        add_header X-XSS-Protection                     "1; mode=block" always;

        # Remove X-Powered-By, which is an information leak
        fastcgi_hide_header X-Powered-By;
        index index.php index.html /cloud/index.php$request_uri;

        # Rule borrowed from `.htaccess` to handle Microsoft DAV clients
        location = /cloud {
            if ( $http_user_agent ~ ^DavClnt ) {
                return 302 /cloud/remote.php/webdav/$is_args$args;
            }
        }

        # Rules borrowed from `.htaccess` to hide certain paths from clients
        location ~ ^/cloud/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)    { return 404; }
        location ~ ^/cloud/(?:\.|autotest|occ|issue|indie|db_|console)                  { return 404; }
        location ~ \.php(?:$|/) {
            # Required for legacy support
            rewrite ^/cloud/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+|.+\/richdocumentscode\/proxy) /cloud/index.php$request_uri;

            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            set $path_info $fastcgi_path_info;

            try_files $fastcgi_script_name =404;

            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $path_info;
           # fastcgi_param HTTPS on;

            fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
            fastcgi_param front_controller_active true;     # Enable pretty urls
            fastcgi_pass php-handler;

            fastcgi_intercept_errors on;
            fastcgi_request_buffering off;

            fastcgi_max_temp_file_size 0;
        }
        location ~ \.(?:css|js|svg|gif|png|jpg|ico|wasm|tflite|map)$ {
            try_files $uri /cloud/index.php$request_uri;
            add_header Cache-Control "public, max-age=15778463, $asset_immutable";
            access_log off;     # Optional: Don't log access to assets

            location ~ \.wasm$ {
                default_type application/wasm;
            }
        }
        location ~ \.woff2?$ {
            try_files $uri /cloud/index.php$request_uri;
            expires 7d;         # Cache-Control policy borrowed from `.htaccess`
            access_log off;     # Optional: Don't log access to assets
        }
        # Rule borrowed from `.htaccess`
        location /cloud/remote {
            return 301 /cloud/remote.php$request_uri;
        }
        location /cloud {
            try_files $uri $uri/ /cloud/index.php$request_uri;
        }
    }
}
```

### MediaWiki

para migrar la aplicación de mediawiki tenemos que realizar los siguientes pasos:

* Primero tenemos que transferir los ficheros que se encuentren en el document root de mediawiki a la vps, en este caso con rsync:

```bash
rsync -avP /var/www/mediawiki/ calcetines@nodriza.admichin.es:/home/calcetines/mediawiki
```

* dentro de la vps lo movemos a su nueva ubicación:

```bash
mv mediawiki /var/www/html/portal
```

* El último paso, es configurar el fichero `LocalSettings.php` (he quitado los comentarios para que sea menos largo):

```php
<?php
if ( !defined( 'MEDIAWIKI' ) ) { exit;
}
$wgSitename = "adMICHIn.es"; $wgMetaNamespace = "AdMICHIn.es";
$wgScriptPath = "/portal";
$wgServer = "http://www.admichin.es";
$wgResourceBasePath = $wgScriptPath;
$wgLogos = [ '1x' => "https://i.imgur.com/KqnlgCE.png",
	
	
	'icon' => "https://i.imgur.com/KqnlgCE.png", ];
$wgEnableEmail = true; $wgEnableUserEmail = true; # UPO $wgEmergencyContact = "apache@������.invalid"; $wgPasswordSender = 
"apache@������.invalid"; $wgEnotifUserTalk = false; # UPO $wgEnotifWatchlist = false; # UPO $wgEmailAuthentication = true;
$wgDBtype = "mysql"; $wgDBserver = "localhost"; $wgDBname = "mediawiki"; $wgDBuser = "admin"; $wgDBpassword = 
"contraseña";
$wgDBprefix = "";
$wgDBTableOptions = "ENGINE=InnoDB, DEFAULT CHARSET=binary";
$wgSharedTables[] = "actor";
$wgMainCacheType = CACHE_NONE; $wgMemCachedServers = [];
$wgEnableUploads = false;
$wgUseInstantCommons = false;
$wgPingback = true;
$wgLanguageCode = "es";
$wgLocaltimezone = "UTC";
$wgSecretKey = "61c7675d604b7bd8b0b434dd7c53d6470ff8797636136e4ef7ade0e08cdaba14";
$wgAuthenticationTokenVersion = "1";
$wgUpgradeKey = "696787eac8f24d0f";
$wgRightsPage = ""; # Set to the title of a wiki page that describes your license/copyright $wgRightsUrl = ""; $wgRightsText = 
""; $wgRightsIcon = "";
$wgDiff3 = "/usr/bin/diff3";
$wgDefaultSkin = "MonoBook";
wfLoadSkin( 'MinervaNeue' ); wfLoadSkin( 'MonoBook' ); wfLoadSkin( 'Timeless' ); wfLoadSkin( 'Vector' );
wfLoadExtension( 'SimpleCalendar' );
$wgUsePathInfo = TRUE;
```

Tras eso mediaWiki estaría totalmente configurado y podremos acceder con: [http://www.admichin.es](http://www.admichin.es)

![m1](https://i.imgur.com/3BgNcj6.png)

### NextCloud

Para migrar nextcloud, el método es diferente. Siguiendo [guía oficial](https://docs.nextcloud.com/server/latest/admin_manual/maintenance/migrating.html) hay que seguir los siguientes pasos:

* En la vps, tenemos que instalar los requisitos previos de nextcloud

```bash
pt install php-zip php-gd php-curl -y
```

* En la máquina original, hay que activar el modo mantenimiento del Nextcloud (desde el document root de nextcloud) y apagar el servidor tras 6-7 minutos:

```bash
sudo -u www-data php occ maintenance:mode --on
systemctl stop apache2
```

![m2](https://i.imgur.com/gFqYaJ9.png)

* Cuando termine copiamos los ficheros de nextcloud a la vps:

```bash
rsync -avP /var/www/nextcloud/ calcetines@nodriza.admichin.es:/home/calcetines/nextcloud
```

* Dentro de la vps lo movemos a su nueva ubicación:

```bash
mv nextcloud /var/www/html/cloud
```

* en el fichero config.php tenemos que adaptar las opciones:

```php
<?php
$CONFIG = array (
  'instanceid' => 'oct5tjcnoj2h',
  'passwordsalt' => 'uz7Kh0ZsihYsbbhIhL/HojMYSVc20y',
  'secret' => 'NxO/97NF1AoG+oCMrY3ryaefvSGO6SHczYjoc5x8NvsLZ1ma',
  'trusted_domains' => 
  array (
    0 => 'www.admichin.es',
  ),
  'datadirectory' => '/var/www/html/cloud/data',
  'dbtype' => 'mysql',
  'version' => '25.0.1.1',
  'overwrite.cli.url' => 'http://www.admichin.es/cloud',
  'dbname' => 'nextcloud',
  'dbhost' => 'localhost',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'oc_roberto',
  'dbpassword' => '}QR947P6^,vV%K$ATu]$W~%)AUhj6X',
  'installed' => true,
  'maintenance' => true,
);
```

* Ahora, teniendo en cuenta que el serverblock de nginx está configurado, comprobamos que al acceder a [http://www.admichin.es/cloud](http://www.admichin.es/cloud) aparece el modo mantenimiento.

* Si aparecece la misma imagen que en el caso anterior, entonces podemos cambiar el valor de configuración de `mainteinance` a false, y recargamos la página:

![m3](https://i.imgur.com/7jYwYOI.png)
![m4](https://i.imgur.com/06VeYQ7.png)