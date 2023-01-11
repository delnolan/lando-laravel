Lando Example Setup
==============

This is an example setup for local development using Lando.
In this example we create a LAMP stack that uses different subdomains and subdirectories for different elements of the project.
The stack used for the project in this example was:
1. Laravel backend api - **app.laravel.lndo.site/api**
2. Angular frontend - **app.laravel.lndo.site**
3. Wordpress on subdomain - **wp.laravel.lndo.site**
4. Laravel websockets - **ws-laravel.lndo.site**
The configuration for the above can be viewed in **config/default.conf**

Windows & WSL2
--------------
Although this should be irrelevant, the device and system used when working through this example:
1. Windows
2. Windows Subsystem for Linux (WSL2)
3. Docker Desktop

It's nice to be using Linux on Windows. Thanks WSL2.

If you're using a similar setup using Windows you'll have to take some steps to get going. 
At a high level they are:
1. Expose WSL2 on your windows machine
2. Install Ubuntu CLI from Windows Store
3. Install Docker Desktop if not already done
4. Install Lando

For WSL2, make sure your project is set up in **~** or **/home/yourusername** directory.

Mac and Ubuntu should be fine, although not tested. 

Get Going
--------------
First create a docroot directory in the root of this project (where .lando.yml lives)

```bash
 mkdir docroot
 # and put your php project in docroot
```
Run Lando:
```bash
 lando start
```

Some custom apache2 setup commands are included. Namely, setting up supervisor for laravel-websockets. 
Feel free to remove this. For that matter, feel free to chop and change the .lando.yml file any way you see fit. Who am I to tell you what to do???

Run Lando:
```bash
name: laravel
services:
  phpmyadminservice: # run phpmyadmin
    type: phpmyadmin
    hosts:
      - mysql57
  mysql57: # sets up our database. I manually added 2x databases after setup and access them via root credentials. Important! When connecting host will NOT be localhost but the internal host (mysql57.laravel.internal). Run lando info after startup to see credential. 
    type: mysql:5.7
    portforward: true
  appservice: # our php service
    type: php
    webroot: docroot
    via: apache
    build_as_root: # enables apache modules on build
      - 'a2enmod headers'
      - 'a2enmod proxy'
      - 'a2enmod proxy_http'
      - 'service apache2 restart'
    run_as_root: 
      # this is to set up supervisor for laravel-websockets. It doesn't work in build_as_root. 
      # 1. Install supervisor
      # 2. Create websockets.conf file
      # 3. Starts supervisor
      - 'apt update -y && apt install -y supervisor && touch /etc/supervisor/conf.d/websockets.conf && echo "[program:websockets]" > /etc/supervisor/conf.d/websockets.conf && echo "command=/usr/local/bin/php /app/docroot/artisan websockets:serve" >> /etc/supervisor/conf.d/websockets.conf && echo "numprocs=1" >> /etc/supervisor/conf.d/websockets.conf && echo "autostart=true" >> /etc/supervisor/conf.d/websockets.conf && echo "autorestart=true" >> /etc/supervisor/conf.d/websockets.conf  && echo "redirect_stderr=true" >> /etc/supervisor/conf.d/websockets.conf && echo "stdout_logfile=/app/worker.log" >> /etc/supervisor/conf.d/websockets.conf && service supervisor stop && service supervisor start && supervisorctl stop websockets && supervisorctl start websockets'
    config: # Add additional conf without having to ssh in to machine and edit files... nice. 
      vhosts: config/default.conf
      php: config/php.ini
  node: # for npm command
    type: node

proxy:
  phpmyadminservice:
    - pma.laravel.lndo.site # db
  appservice:
    - wp.laravel.lndo.site # wordpress
    - app.laravel.lndo.site # angular
    - ws-laravel.lndo.site # websockets proxy - comment this out if not using websockets

tooling:
  php: # lando php artisan cache:clear (for artisan and other php commands)
    service: appservice
  composer: # lando composer install (to set up composer)
    service: appservice
  npm: # lando npm run dev (to build laravel js)
    service: node
  # Anything below this may be superfluous but I haven't removed it
  node:
    service: node
  gulp:
    service: node
  yarn:
    service: node
bindAddress: "0.0.0.0"

```

To rebuild
```bash
lando rebuild -y
```

There are other Lando commands but you can go figure them out yourself. 
```bash
lando --help
```

Note: red urls
--------------
Firstly, if you're using Windows and you get red urls, just add the urls to your hosts file and you'll be golden. Google it (or ask ChatGPT). 

Secondly, you may get the red url on ws-laravel.lndo.site. This is because the config is for a proxy url. It works with laravel-websockets (even when red) BUT it causes a short hang when building with lando as it waits for the url to timeout. 
If you're not using websockets, comment it out and you'll get a speedy lando build and a clean sweep of green urls. 

Readme written retrospectively
------------------------------
This readme was written retrospectively so I'm probably missing a few bits and pieces but feel free to let me know and I'll rectify. Thanks. 
