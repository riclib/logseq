---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/adding-basic-auth-to-prometheus-with-nginx
author: [[Brian Brazil]] 
---
# Adding Basic Auth to Prometheus with Nginx

> Prometheus doesn't provide authentication support in order to focus energy on making an awesome monitoring tool. Instead users can take advantage of a more purpose designed tool such as Nginx to do so. This post will look at how you can do that.

---
Prometheus doesn't provide authentication support in order to focus energy on making an awesome monitoring tool. Instead users can take advantage of a more purpose designed tool such as Nginx to do so. This post will look at how you can do that.

To start you should [install Nginx](http://nginx.org/en/docs/install.html).

Next let's get a basic Ngingx setup working. Here's an Nginx configuration that simply acts as a reverse proxy from Prometheus on port 9090 to port 19090:
```json
http {
  server {
    listen 0.0.0.0:19090;
    location / {
      proxy_pass http://localhost:9090/;
    }
  }
}
events {
}
```

If you start Nginx and visit [http://localhost:19090](http://localhost:19090/) you'll see the Prometheus status page.

Now that Nginx is working we can add [basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication). In order to authenticate users we need a list of usernames and passwords. We'll use the `htpasswd` utility for this. This is in the `apache2-utils` packages on Debian based systems such as Ubuntu. We'll add a user called "myuser":

$ htpasswd -c .htpasswd myuser
New password: 
Re-type new password: 
Adding password for user myuser

Then configure basic auth in the Nginx configuration file:
```json
http {
  server {
    listen 0.0.0.0:19090;
    location / { 
      proxy_pass http://localhost:9090/;

      auth_basic "Prometheus";
      auth_basic_user_file ".htpasswd";
    }
  }
}
events {
}
```

If you restart Nginx and once again visit [http://localhost:19090](http://localhost:19090/) you'll now be asked for your username and password.

Don't forget to lock down file permissions on the `.htpasswd` file, and keep it outside of any paths that are served over HTTP. The same approach can be used with other components of Prometheus, such as the Alertmanager and Node Exporter.
