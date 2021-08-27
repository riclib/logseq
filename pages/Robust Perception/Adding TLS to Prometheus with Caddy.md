---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/adding-tls-to-prometheus-with-caddy
author: [[Brian Brazil]] 
---
> Prometheus doesn't come with server-side security features out of the box, but there's many options out there to provide them.

# Adding TLS to Prometheus with Caddy


Prometheus doesn't come with server-side security features out of the box, but there's many options out there to provide them.

I'll presume that you've already a Prometheus running locally on port 9090. There's many pieces of software that implement all the TLS features you could want, here I'm going to use Caddy. To start off download Caddy and configure it with a self-signed cert:

wget https://github.com/caddyserver/caddy/releases/download/v1.0.3/caddy\_v1.0.3\_linux\_amd64.tar.gz
tar -xzf caddy\_v\*\_linux\_amd64.tar.gz
cat > Caddyfile << EOF
localhost:443
tls self\_signed
proxy / localhost:9090
EOF
# Allow caddy to bind to 443.
sudo setcap cap\_net\_bind\_service=+ep caddy
./caddy

If you now visit [https://localhost](https://localhost/) (and bypass the security warning about the self-signed cert) you'll see Prometheus:

[![](https://www.robustperception.io/wp-content/uploads/2019/08/Screenshot_2019-08-20_14-18-36.png)](https://www.robustperception.io/wp-content/uploads/2019/08/Screenshot_2019-08-20_14-18-36.png)

This is fine for local testing, but a real cert is needed for production. You can use [Let's Encrypt](https://letsencrypt.org/) for this, so change the configuration to use a real DNS working name:

cat > Caddyfile << EOF
# Put in your own domain and email address here.
demo.robustperception.io:443
tls your.email@example.com
proxy / localhost:9090
EOF
# -agree is to the CA's Terms and Conditions
./caddy -agree

And if you visit your website you'll see it's serving properly over HTTPS:

[![](https://www.robustperception.io/wp-content/uploads/2019/08/Screenshot_2019-08-20_14-26-07.png)](https://www.robustperception.io/wp-content/uploads/2019/08/Screenshot_2019-08-20_14-26-07.png)

This isn't quite everything. Prometheus is probably still listening on all interfaces, allowing bypassing of Caddy. You can pass `--web.listen-address=localhost:9090` to Prometheus to have it only listen locally. Another issue is that Prometheus doesn't know about the proper URL to use in alerts etc., which in this example can be handled with `--web.external-url=https://demo.robustperception.io`.

_Need help securing Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
