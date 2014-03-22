docker-ghost
============

Run [Ghost](http://ghost.org) with Docker. As an aside, this variant is compatible with an existing Dokku install on Ubuntu that handles other vhosts (e.g., on DigitalOcean). To get started, run:

    $ docker run -d -p 2368:2368 -v /opt/ghost -v /data -name GHOST -e GHOST_URL=http://blog.mysite.com orchardup/ghost

Your Ghost blog will be running on [http://blog.mysite.com:2368](http://blog.mysite.com:2368). Set up NGINX config to reverse proxy:

    $ vi /etc/nginx/sites-available/proxy-ghost-blog.mysite.com

Use a variant on the following configuration:

    server {
        listen 0.0.0.0:80;
        server_name blog.mysite.com;
        access_log /var/log/nginx/blog.mysite.com.log;
    
        location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header HOST $http_host;
            proxy_set_header X-NginX-Proxy true;

            proxy_pass http://127.0.0.1:2368;
            proxy_redirect off;
        }
    }

Enable the NGINX proxy:

    $ ln -s /etc/nginx/sites-available/proxy-ghost-blog.mysite.com /etc/nginx/sites-enabled/proxy-ghost-blog.mysite.com
    $ nginx -s reload

Make any necessary configuration changes (e.g., email settings):

    $ docker run -i -t -volumes-from=GHOST orchardup/ghost /bin/bash
    (container) # apt-get install vim-tiny
    (container) # vi config.js
    (container) # exit

Configuration
-------------

This image can be configured with environment variables:

 - `GHOST_URL`: The url to use when providing links to the site, E.g. in RSS and email.


Caveats
-------

You will lose your uploads if you upgrade Ghost by replacing the image. This is because there is no way of specifying an upload directory for Ghost yet, but [they're working on it](https://github.com/TryGhost/Ghost/issues/635). The database is saved in a volume called `/data` so it will persist just fine.


