`upstartable-nginx`
===================

Debian packaging of Nginx 1.3.13 with WebSockets for supervision by Upstart.

Usage
-----

Build `nginx_1.3.13-betable1_amd64.deb` using [FPM](https://github.com/jordansissel/fpm):

    sh bootstrap.sh

Add `nginx_1.3.13-betable1_amd64.deb` to a Debian archive using [Freight](https://github.com/rcrowley/freight) or some other less awesome tools or just install it directly:

    dpkg -i nginx_1.3.13-betable1_amd64.deb

The package doesn't come with a complete configuration so you'll have to adjust `/etc/nginx/nginx.conf`, `/etc/nginx/conf.d`, `/etc/nginx/sites-available/*`, and `/etc/nginx/sites-enabled/*` as you see fit before it will start.

Configure Upstart.  Here's how we do it at Betable:

    description "nginx"

    start on runlevel [2345]
    stop on runlevel [!2345]

    respawn
    respawn limit 360 180

    chdir /etc/nginx

    script
        set -e
        rm -f "/tmp/nginx.log"
        mkfifo "/tmp/nginx.log"
        (logger -t"nginx" <"/tmp/nginx.log" &)
        exec >"/tmp/nginx.log" 2>"/tmp/nginx.log"
        rm "/tmp/nginx.log"
        exec /usr/sbin/upstartable-nginx
    end script

Start Nginx:

    start nginx

Reload Nginx gracefully:

    reload-upstartable-nginx

Restart Nginx less gracefully:

    restart nginx

Stop Nginx gracefully:

    stop nginx

Assumptions
-----------

We assume Nginx will always store its process ID in `/var/run/nginx.pid` and that it will always be listening on `0.0.0.0:80`.
