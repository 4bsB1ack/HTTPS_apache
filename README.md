# HTTPS Lab Setup

index | content
--- | ---
1 | [Create Directory](#create-directory)
2 | [Generate Root CA Private Key](#generate-root-ca-private-key)
3 | [Create Self-Signed Root CA Certificate](#create-self-signed-root-ca-certificate)
4 | [Generate Server Certificate Signing Request (CSR)](#generate-server-certificate-signing-request-csr)
5 | [Sign Server CSR with Root CA](#sign-server-csr-with-root-ca)
6 | [Enable UFW Firewall](#enable-ufw-firewall)
7 | [Check UFW Status](#check-ufw-status)
8 | [List Available Application Profiles in UFW](#list-available-application-profiles-in-ufw)
9 | [Allow Web Traffic (HTTP and HTTPS)](#allow-web-traffic-http-and-https)
10 | [Enable SSL Module for Apache](#enable-ssl-module-for-apache)
11 | [Edit Apache Virtual Host Configuration](#edit-apache-virtual-host-configuration)
12 | [Create Document Root Directory](#create-document-root-directory)
13 | [Create Index File](#create-index-file)
14 | [Enable Site  Configuration](#enable-site--configuration)
15 | [Restart Apache](#restart-apache)
16 | [Test Apache Configuration](#test-apache-configuration)
17 | [Reload Apache](#reload-apache)



## Create Directory
```bash
mkdir https_lab
cd https_lab
```
~~~
mkdir https_lab: Creates a new directory named https_lab.
cd https_lab: Changes the current working directory to https_lab.
~~~
## Generate Root CA Private Key

```bash
openssl genrsa -aes256 -out MyOrg_rootCA.key 4096
```
`the command in details :`
~~~
openssl: OpenSSL command line tool.
genrsa: Generates an RSA private key.
-aes256: Encrypts the private key with AES-256 encryption.
-out: Specifies the filename to write the key to.
4096: Specifies the number of bits in the key to create.
~~~


## Create Self-Signed Root CA Certificate
```bash
openssl req -x509 -new -nodes -key MyOrg_rootCA.key -sha256 -days 1826 -out MyOrg_rootCA.crt -subj '/CN=My RootCA/C=DZ/ST=TO/L=TiziOuzou/O=MyOrganisation'
```
`the command in details :`
~~~
req: PKCS#10 certificate request and certificate generating utility.
-x509: Creates a self-signed certificate.
-new: Generates a new certificate request.
-nodes: Creates a key without a passphrase.
-key: Specifies the filename of the key to be used.
-sha256: Uses the SHA-256 algorithm to create the certificate hash.
-days: Specifies the number of days to certify the certificate for.
-out: Specifies the filename to write the newly created certificate to.
-subj: Specifies the subject information to be used for the certificate.
'/CN=My RootCA/C=DZ/ST=TO/L=TiziOuzou/O=MyOrganisation' : Specifies the subject information to be used for the certificate.
~~~
## Generate Server Certificate Signing Request (CSR)
```bash
openssl req -new -nodes -out myserver.local.csr -newkey rsa:4096 -keyout myserver.local.key -subj '/CN=My firewall/C=DZ/ST=TO/L=TiziOuzou/O=MyOrganisation'
```
`the command in details :`
~~~
req: PKCS#10 certificate request and certificate generating utility.
-new: Generates a new certificate request.
-nodes: Creates a key without a passphrase.
-out: Specifies the filename to write the newly created certificate to.
-newkey: Creates a new certificate request and a new private key.
rsa:4096: Specifies the number of bits in the key to create.
-keyout: Specifies the filename to write the newly created private key to.
-subj: Specifies the subject information to be used for the certificate.
'/CN=My firewall/C=DZ/ST=TO/L=TiziOuzou/O=MyOrganisation' : Specifies the subject information to be used for the certificate.
~~~

## Sign Server CSR with Root CA
```bash
openssl x509 -req -in myserver.local.csr -CA MyOrg_rootCA.crt -CAkey MyOrg_rootCA.key -CAcreateserial -out myserver.local.crt -days 365 -sha256
```
`the command in details :`
~~~
x509: Certificate display and signing utility.
-req: Creates a certificate request.
-in: Specifies the filename to read the certificate request from.
-CA: Specifies the filename of the CA certificate to be used.
-CAkey: Specifies the filename of the CA key to be used.
-CAcreateserial: Creates a certificate serial number file if one does not exist.
-out: Specifies the filename to write the newly created certificate to.
-days: Specifies the number of days to certify the certificate for.
-sha256: Uses the SHA-256 algorithm to create the certificate hash.
~~~

## Enable UFW Firewall
```bash
sudo ufw enable
```
`the command in details :`
~~~
sudo: Runs the command as root.
ufw: Uncomplicated Firewall.
enable: Enables the firewall.
~~~

## Check UFW Status
```bash
sudo ufw status
```
`the command in details :`
~~~
sudo: Runs the command as root.
ufw: Uncomplicated Firewall.
status: Displays the status of the firewall.
~~~

## List Available Application Profiles in UFW
```bash
sudo ufw applist
```
`the command in details :`
~~~
sudo: Runs the command as root.
ufw: Uncomplicated Firewall.
applist: Displays a list of available application profiles.
~~~

## Allow Web Traffic (HTTP and HTTPS)
```bash
sudo ufw allow "WWW Full"
```
`the command in details :`
~~~
sudo: Runs the command as root.
ufw: Uncomplicated Firewall.
allow: Allows traffic.
"WWW Full": Specifies the application profile to allow.
~~~

## Enable SSL Module for Apache
```bash
sudo a2enmod ssl
```
`the command in details :`
~~~
sudo: Runs the command as root.
a2enmod: Enables an Apache module.
ssl: Specifies the module to enable.
~~~

## Edit Apache Virtual Host Configuration
```bash
sudo nano /etc/apache2/sites-available/myserver.local.conf
```
`the command in details :`
~~~
sudo: Runs the command as root.
nano: A text editor.
/etc/apache2/sites-available/myserver.local.conf: Specifies the file to edit.
~~~

`the content of the file :`
```bash
<VirtualHost *:80>
        ServerName myserver.local
        Redirect / https://localhost/
</VirtualHost>

<VirtualHost *:443>
   ServerName myserver.local
   DocumentRoot /var/www/myserver.local

   SSLEngine on
   SSLCertificateFile /home/kali/rmse/https_lab/myserver.local.crt
   SSLCertificateKeyFile /home/kali/rmse/https_lab/myserver.local.key
</VirtualHost>
```

## Create Document Root Directory
```bash
sudo mkdir /var/www/myserver.local
```
`the command in details :`
~~~
sudo: Runs the command as root.
mkdir: Creates a new directory.
/var/www/myserver.local: Specifies the directory to create.
~~~

## Create Index File
```bash
sudo nano /var/www/myserver.local/index.html
```
`the content of the file :`
```bash
<html>
  <head>
    <title>HTTPS Lab</title>
  </head>
  <body>
    <h1>helo HTTPS Lab !</h1>
  </body>
</html>
```

## Enable Site  Configuration

```bash
sudo a2ensite myserver.local.conf
```

`the command in details :`
~~~
sudo: Runs the command as root.
a2ensite: Enables an Apache site.
myserver.local.conf: Specifies the site to enable.
~~~

## Restart Apache
```bash
sudo systemctl restart apache2
```

`the command in details :`
~~~
sudo: Runs the command as root.
systemctl: Controls the systemd system and service manager.
restart: Restarts the Apache service.
~~~

## Test Apache Configuration
```bash
sudo apachectl configtest
```
`the command in details :`
~~~
sudo: Runs the command as root.
apachectl: Controls the Apache HTTP server.
configtest: Tests the Apache configuration file for errors.
~~~

## Reload Apache
```bash
sudo systemctl reload apache2
```
`the command in details :`
~~~
sudo: Runs the command as root.
systemctl: Controls the systemd system and service manager.
reload: Reloads the Apache service.
~~~
