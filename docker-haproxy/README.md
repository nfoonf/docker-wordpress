# HAProxy + Lua on Alpine Linux

Check the Dockerfile for the most recent build information. This image was
originally created with HAProxy 1.6.2 and Lua 5.3 on Alpine Linux.

## Usage

```sh
docker run --rm --name haproxy \
  -v /data/haproxy/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg \
  -p 80:80 \
  -p 443:443 \
  ecor/proxy
```

You must specify the configuration file.

If you also want to supply your own error files:

```sh
docker run --rm --name haproxy \
  -v /data/haproxy/errors:/usr/local/etc/haproxy/errors \
  -v /data/haproxy/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg \
  -p 80:80 \
  -p 443:443 \
  ecor/proxy
```

**Reloading**

You can soft-reload HAProxy using the following command:

`docker exec -it docker-proxy haproxy reload`

## Default configuration

This should be updated for your environment.

```sh
global
  log 127.0.0.1 local0
  #chroot /var/lib/haproxy
  #stats socket /tmp/haproxy mode 660 level admin
  #stats timeout 30s

  # Default SSL material locations
  #crt-base /etc/letsencrypt/live

  lua-load /etc/haproxy/acme-http01-webroot.lua

  # Default ciphers to use on SSL-enabled listening sockets.
  # For more information, see ciphers(1SSL).
  #ssl-default-bind-ciphers AES256+EECDH:AES256+EDH:!aNULL;

  #tune.ssl.default-dh-param 4096

  ssl-default-bind-ciphers kEECDH+aRSA+AES:kRSA+AES:+AES256:RC4-SHA:!kEDH:!LO$
  ssl-default-bind-options no-sslv3

defaults
  log         global
  mode        http
  option	    httplog
  option	    dontlognull
  option	    dontlog-normal
  option	    abortonclose
  timeout     connect 5000
  timeout     client  50000
  timeout     server  50000
  monitor-uri /monitor

frontend http
  bind *:80

  #redirect scheme https if !{ ssl_fc }
  acl acme_challenge path_beg /.well-known/acme-challenge/
  use_backend unencrypted

  default_backend not_found

frontend https
  bind *:443
  use_backend encrypted

backend not_found
  errorfile 400 /etc/haproxy/errors/400.http

backend unencrypted
  errorfile 400 /etc/haproxy/errors/400.http
  # option forwardfor
  # server myserver 127.0.0.1:8080 check

backend encrypted
  errorfile 400 /etc/haproxy/errors/400.http
  # option forwardfor
  # server myserver 127.0.0.1:4443 check
  # http-request set-header X-Forwarded-Port %[dst_port]
  # http-request add-header X-Forwarded-Proto https if { ssl_fc }
```
