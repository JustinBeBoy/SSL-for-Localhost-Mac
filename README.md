# Local SSL websites on macOS Sierra

These instructions will guide you through the process of setting up local, trusted websites on your own computer.

These instructions are intended to be used on macOS Sierra, but they have been known to work in El Capitan, Yosemite, Mavericks, and Mountain Lion.

**NOTE:** You may substitute the `edit` command for `nano`, `vim`, or whatever the editor of your choice is. Personally, I forward the `edit` command to [Sublime Text](http://www.sublimetext.com):

```sh
alias edit="/Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl"
```

---

## Configuring Apache

Within **Terminal**, start **Apache**.
```sh
sudo apachectl start
```

In a **web browser**, visit [http://localhost](http://localhost). You should see a message stating that **It works!**.

### Configuring Apache: Setting up a Virtual Host

Within **Terminal**, edit the Apache Configuration.
```sh
edit /etc/apache2/httpd.conf
```

Within the editor, replace line 212 to supress messages about the server’s fully qualified domain name.
```conf
ServerName localhost
```

Next, uncomment line 160 and line 499 to enable Virtual Hosts.
```conf
LoadModule vhost_alias_module libexec/apache2/mod_vhost_alias.so
```
```conf
Include /private/etc/apache2/extra/httpd-vhosts.conf
```

Optionally, uncomment line 169 to enable PHP.
```conf
LoadModule php5_module libexec/apache2/libphp5.so
```

Within **Terminal**, edit the Virtual Hosts configuration.
```sh
edit /etc/apache2/extra/httpd-vhosts.conf
```

Within the editor, replace the entire contents of this file with the following, replacing *indieweb* with your user name.
```conf
<VirtualHost *:80>
    ServerName localhost
    DocumentRoot "/Users/indieweb/Sites/localhost"

    <Directory "/Users/indieweb/Sites/localhost">
        Options Indexes FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
</VirtualHost>
```

Within **Terminal**, restart Apache.
```sh
sudo apachectl restart
```

### Configuring Apache: Creating a Site

Within **Terminal**, create a **Sites** parent directory and a **localhost** subdirectory, which will be our first site.
```sh
mkdir -p ~/Sites/localhost
```

Next, create a test HTML document within **localhost**.
```sh
echo "<h1>localhost works</h1>" > ~/Sites/localhost/index.html
```

Now, in a **web browser**, visit [http://localhost](http://localhost). You should see a message stating that **localhost works**.

---

## Configuring SSL

Within **Terminal**, create an SSL directory.
```sh
sudo mkdir /etc/apache2/ssl
```

Next, generate a private key and certificate for your site.
```sh
sudo openssl genrsa -out /etc/apache2/ssl/localhost.key 2048
sudo openssl req -new -x509 -key /etc/apache2/ssl/localhost.key -out /etc/apache2/ssl/localhost.crt -days 3650 -subj /CN=localhost
```

Finally, add the certificate to Keychain Access.
```sh
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /etc/apache2/ssl/localhost.crt
```

### Configuring SSL: Setting up a Trusted Virtual Host

Within **Terminal**, edit the Apache Configuration.
```sh
edit /etc/apache2/httpd.conf
```

Within the editor, uncomment lines 89 and 143 to enable modules required by HTTPS.
```conf
LoadModule socache_shmcb_module libexec/apache2/mod_socache_shmcb.so
```
```conf
LoadModule ssl_module libexec/apache2/mod_ssl.so
```

Next, uncomment line 516 to enable Trusted Virtual Hosts.
```conf
Include /private/etc/apache2/extra/httpd-ssl.conf
```

Back in **Terminal**, edit the Virtual Hosts configuration.
```sh
edit /etc/apache2/extra/httpd-vhosts.conf
```

Within the editor, add a **443** VirtualHost Name and **localhost** <VirtualHost> Directive at the end of the file, replacing *indieweb* with your user name.
```conf
<VirtualHost *:443>
    ServerName localhost
    DocumentRoot "/Users/indieweb/Sites/localhost"

    SSLEngine on
    SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
    SSLCertificateFile /etc/apache2/ssl/localhost.crt
    SSLCertificateKeyFile /etc/apache2/ssl/localhost.key

    <Directory "/Users/indieweb/Sites/localhost">
        Options Indexes FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
</VirtualHost>
```

Back in **Terminal**, edit the SSL configuration.
```sh
edit /etc/apache2/extra/httpd-ssl.conf
```

Next, comment line 144 and 154 to skip the default Server Certificate and Server Private Key.
```conf
#SSLCertificateFile "/private/etc/apache2/server.crt"
```
```conf
#SSLCertificateKeyFile "/private/etc/apache2/server.key"
```

Next, beneath the commented certificates or keys, add references to your certificate and key.
```conf
SSLCertificateFile "/etc/apache2/ssl/localhost.crt"
```
```conf
SSLCertificateKeyFile "/etc/apache2/ssl/localhost.key"
```

Back in **Terminal**, restart Apache.
```sh
sudo apachectl restart
```

Now, in a **web browser**, visit [https://localhost](https://localhost). The domain should appear trusted, and you should see a message stating that **localhost works!**.
