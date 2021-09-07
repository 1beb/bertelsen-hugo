---
author: "Brandon Erik Bertelsen"
title: "TLS for RStudio and Shiny using Let's Encrypt"
date: "2016-07-14"
categories:
- R
- Docker
---

Below is a very simple way to have https connections for the open source version of RStudio Server and Shiny Server.

```shell
sudo apt-get install docker  
sudo apt-get install python-pip  
sudo pip install docker-compose  
```

Now, in your user directory, create a text file using your preferred text editor, called docker-compose.yml, that looks like the following, keeping in mind that you need to replace: username with your ssh user name, SUB.YOURDOMAIN.COM, and YOUREMAIL. You can also change the docker images.

```yaml
nginx:  
  image: nginx
  container_name: nginx
  ports:
    - "80:80"
    - "443:443"
  volumes:
    - /etc/nginx/conf.d
    - /etc/nginx/vhost.d
    - /usr/share/nginx/html
    - /home/username/proxy/certs:/etc/nginx/certs:ro
nginx-gen:  
  image: jwilder/docker-gen
  container_name: nginx-gen
  volumes:
    - /var/run/docker.sock:/tmp/docker.sock:ro
    - /home/username/proxy/templates/nginx.tmpl:/etc/docker-gen/templates/nginx.tmpl:ro
  volumes_from:
    - nginx
  entrypoint: /usr/local/bin/docker-gen -notify-sighup nginx -watch -only-exposed -wait 5s:30s /etc/docker-gen/templates/nginx.tmpl /etc/nginx/conf.d/default.conf
letsencrypt-nginx-proxy-companion:  
  image: jrcs/letsencrypt-nginx-proxy-companion
  container_name: letsencrypt-nginx-proxy-companion
  volumes_from:
    - nginx
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
    - /home/username/proxy/certs:/etc/nginx/certs:rw
  environment:
    - NGINX_DOCKER_GEN_CONTAINER=nginx-gen

rstudio:  
  image: rocker/hadleyverse
  container_name: rstudio
  restart: always
  ports:
    - "8787:8787"
  environment:
    - VIRTUAL_PORT=8787
    - ROOT=TRUE
    - VIRTUAL_HOST=SUB.YOURDOMAIN.COM
    - USER=rstudio
    - PASSWORD=password
    - LETSENCRYPT_HOST=SUB.YOURDOMAIN.COM
    - LETSENCRYPT_EMAIL=YOUREMAIL
```

Adding a shiny server is trivial, simply add the following, note that the SUB domain should be different.

```yaml
shiny:  
  image: rocker/shiny
  container_name: shiny
  restart: always
  environment:
    - VIRTUAL_HOST=SUB.YOURDOMAIN.COM
    - LETSENCRYPT_EMAIL=YOUREMAIL
    - LETSENCRYPT_HOST=SUB.YOURDOMAIN.COM
  volumes:
    - /home/username/shiny/apps:/srv/shiny-server/
    - /home/username/shiny/logs:/var/log/
    - /home/username/shiny/packages:/home/shiny/
```

Now at the command line, type `docker-compose up -d` and watch magic happen.

