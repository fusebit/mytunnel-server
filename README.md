# fusetunnel-server

Fusetunnel exposes your localhost to the world for easy testing and sharing! No need to mess with DNS or deploy just to have others test out your changes.

This repo is the server component. If you are just looking for the CLI fusetunnel app, see (https://github.com/fusebit/tunnel).

## Overview

You can easily set up and run your own server. In order to run your own fusetunnel server you must ensure that your server can meet the following requirements:

* You can set up DNS entries for your `domain.tld` and `*.domain.tld` (or `sub.domain.tld` and `*.sub.domain.tld`).
* The server can accept incoming TCP connections for any non-root TCP port (i.e. ports over 1000).

The above are important as the client will ask the server for a subdomain under a particular domain. The server will listen on any OS-assigned TCP port for client connections.

## Setup

```shell
# pick a place where the files will live
git clone git://github.com/fusebit/tunnel-server.git
cd tunnel-server
npm install

# server set to run on port 1234
bin/server.js --port 1234
```

The fusetunnel server is now running and waiting for client requests on port 1234. You will most likely want to set up a reverse proxy to listen on port 80 (or start fusetunnel on port 80 directly).

**NOTE** By default, fusetunnel will use subdomains for clients, if you plan to host your fusetunnel server itself on a subdomain you will need to use the _--domain_ option and specify the domain name behind which you are hosting fusetunnel. (i.e. my-tunnel-server.example.com)

### Systemd config

Sample systemd config file:
```
[Unit]
Description=FuseTunnel service
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=ec2-user
Environment=DEBUG=*
ExecStart=/usr/bin/node /home/ec2-user/codebase/bin/server.js --port 80 --domain tunnel.example.com

[Install]
WantedBy=multi-user.target
```

Setup flow:
```bash
sudo setcap CAP_NET_BIND_SERVICE=+eip /usr/bin/node # allow non root access to bind below port number 1024
sudo vi /etc/systemd/system/fusetunnel.service
sudo systemctl start fusetunnel
sudo systemctl enable fusetunnel # Add to autoload
journalctl -f -u fusetunnel # See logs
```
## Enable TLS

You can enable TLS support for both API plane and data plane natively

To enable data plane TLS, enable these flags on fusetunnel-server

`--tunnel-ssl-cert` (the SSL certificate of the tunnel)

`--tunnel-ssl-key` (the SSL key of the tunnel)

`--tunnel-ssl-ca` (optional: the certificate authority/full chain certificate of the tunnel)

`--auto-generate-cert` (optional: automatically generate self signed certificate and key for tunnel data plane)

To enable control plane TLS, enable these flags in fusetunnel-server

`--web-cert` (the SSL cert of the API server)

`--web-key` (the SSL key of the API server)

`--web-ca` (the certificate authority/full chain certificate of the tunnel)

## Use behind reverse proxy/load balancer

If you need to override the IP sockets connect to for fuse-tunnel, enable these flags on fusetunnel-server

`--override-tunnel-ip` (optional: override the IP fusetunnel return to client)

`--auto-discover-tunnel-ip` (optional: automatically discover the public ip of the fusetunnel server through AWS' public API)

## Use your server

You can now use your domain with the `--host` flag for the `lt` client.

```shell
npx fusetunnel --host http://tunnel.example.com --port 9000
```

You will be assigned a URL similar to `http://heavy-puma-9.tunnel.example.com`.

If your server is acting as a reverse proxy (i.e. nginx) and is able to listen on port 80, then you do not need the `:1234` part of the hostname for the `lt` client.

## REST API

### POST /api/tunnels

Create a new tunnel. A FuseTunnel client posts to this enpoint to request a new tunnel with a specific name or a randomly assigned name.

### GET /api/status

General server information.

## Deploy

You can deploy your own fusetunnel server using the prebuilt docker image.

**Note** This assumes that you have a proxy in front of the server to handle the http(s) requests and forward them to the fusetunnel server on port 3000. You can use our [localtunnel-nginx](https://github.com/localtunnel/nginx) to accomplish this.

If you do not want ssl support for your own tunnel (not recommended), then you can just run the below with `--port 80` instead.


```
docker run -d \
    --restart always \
    --name fusetunnel \
    --net host \
    defunctzombie/localtunnel-server:latest --port 3000
```
