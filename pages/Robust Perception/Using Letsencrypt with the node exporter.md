---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-letsencrypt-with-the-node-exporter
author: [[Brian Brazil]] 
---
> As of 1.0, the node exporter has experimental support for TLS. This can be hooked up to Letsencrypt.

# Using Letsencrypt with the node exporter


As of 1.0, the node exporter has experimental support for TLS. This can be hooked up to Letsencrypt.

The TLS feature of the node exporter doesn't have any in-built support for Letsencrypt or any other way to renew certificates itself. Instead what it does is that if the certificate files change on disk then it will automatically start using them. Thus by using [certbot](https://certbot.eff.org/) to automatically renew your certificates, the node exporter can use Letsencrypt certs. I'll now demonstrate.

Firstly certbot needs to be installed. I happen to be on an Ubuntu machine, so the instructions are:

apt-get update
apt-get install software-properties-common
add-apt-repository universe
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install certbot

You can find the instructions for your setup on the [certbot website](https://certbot.eff.org/instructions). Then we need to setup our certificates by running:

certbot certonly --webroot

If you already have a server running on port 80, you'll need to use `--standalone` instead and tell certbot where it is serving off on the filesystem. In either case, certbot will ask for the domain name of the server, which you will need to have already setup in DNS.

If that all works you will get output including:

 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/www.example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/www.example.com/privkey.pem

Now we can download, setup, and run the node exporter, adjusting the domain as needed:

DOMAIN=www.example.com
wget https://github.com/prometheus/node\_exporter/releases/download/v1.0.1/node\_exporter-1.0.1.linux-amd64.tar.gz
tar -xzf node\_exporter\*.tar.gz
cd node\_exporter\*amd64
cat <<EOF > web\_config.yml
tls\_server\_config:
  cert\_file: /etc/letsencrypt/live/$DOMAIN/fullchain.pem
  key\_file: /etc/letsencrypt/live/$DOMAIN/privkey.pem
EOF
./node\_exporter --web.config web\_config.yml

If you visit https://$DOMAIN:9100 you should be able to access the node exporter over TLS. Don't forget that when scraping with Prometheus that you'll need to specify `scheme: https`.

The Ubuntu packages will have setup a cronjob to automatically renew your certificates, so there's no need for you to do so yourself. Adding some [monitoring](https://www.robustperception.io/get-alerted-before-your-ssl-certificates-expire) may not be the worst of ideas though.

_Need help securing Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
