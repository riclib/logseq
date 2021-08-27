---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/adding-basic-auth-to-prometheus-with-apache
author: [[Brian Brazil]] 
---
> Having previously discussed why the Prometheus project does not support SSL and user authentication out of the box and detailing how to add basic authentication with Nginx, we will now demonstrate how to do the same with Apache.

# Adding Basic Auth to Prometheus with Apache


Having previously discussed why the [Prometheus project does not support SSL and user authentication out of the box](https://www.robustperception.io/prometheus-security-authentication-authorization-and-encryption/) and detailing [how to add basic authentication with Nginx](https://www.robustperception.io/adding-basic-auth-to-prometheus-with-nginx/), we will now demonstrate how to do the same with Apache.

Assuming you have Apache server installed on your machine, open up the `httpd.conf` configuration file located at `/etc/apache2/` and uncomment the following lines:

LoadModule proxy\_module libexec/apache2/mod\_proxy.so
LoadModule proxy\_http\_module libexec/apache2/mod\_proxy\_http.so

Next add the following to the bottom of `apache2/extra/httpd-vhosts.conf` to proxy requests made to the server to our running Prometheus instance:

<VirtualHost \*:80>
    ProxyPreserveHost On
    ProxyPass / http://localhost:9090/
    ProxyPassReverse / http://localhost:9090/
</VirtualHost>

You can test that this is working by restarting Apache with the command

$ sudo apachectl restart

and visiting [localhost](http://localhost/).

With the reverse proxy setup we can now add basic user authentication.

First we will create a password file containing a username and password that will grant access to our Prometheus server:

$ sudo htpasswd -c /etc/apache2/.htpasswd admin
New password: 
Re-type new password: 
Adding password for user admin

Finally amend the previous section we added to require auth when accessing the Prometheus server:

<VirtualHost \*:80>
    ProxyPreserveHost on
    ProxyPass / http://localhost:9090/
    ProxyPassReverse / http://localhost:9090/
    <Location />
        AuthType Basic
        AuthName "Restricted Content"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Location>
</VirtualHost>

To see the changes we must again restart Apache:

$ sudo apachectl restart

Upon trying to access the Prometheus GUI, you should now be greeted with a prompt for a username and password.

Don’t forget to lock down file permissions on the `.htpasswd file`, and keep it outside of any paths that are served over HTTP. The same approach can be used with other components of Prometheus, such as the Alertmanager and Node Exporter.

_Need help securing Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by Conor Broderick in [Posts](https://www.robustperception.io/category/posts)
