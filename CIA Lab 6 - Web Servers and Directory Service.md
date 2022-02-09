---
tags: Classical Internet Applications
---
:::success
# CIA Lab 6 - Web Servers and Directory Service
Name: Ivan Okhotnikov
:::

:::warning
**Introduction**
This lab has two sections. The first section covers web servers and the second section covers
directory services. Please do not use docker-ready images, try to install yourself.
Note: TAs might come to see your demo
:::

# Section 1: Web Server
## Task 1 - Install & Configure Virtual Hosts

:::warning
1. Fetch, verify, build and install the webserver daemon from the source.
Note: Some features of your web server may be built-in or modularized. Enable at least SSL/TLS during your installation.
2. Define the root directory and then two virtual hosts (and configure DNS records or wildcard accordingly):
aaa.stX.sne21.ru
bbb.stX.sne21.ru
3. Create a simple, unique HTML page for each virtual host to make sure that the server can correctly serve it.
Testing:
4. Check the configuration syntax, start the daemon and enable it at boot time.
5. Use curl to display the contents of a full HTTP/1.1 session served by your server.
6. Explain the meaning of each request and reply header.
:::

## Implementation:

:::info
>1. Fetch, verify, build and install the webserver daemon from the source.
>>Note: Some features of your web server may be built-in or modularized. Enable at least SSL/TLS during your installation.

### Fetch & verify
Before starting work, I will download the public key, signature and archive with the `NGINX` program

```
wget http://nginx.org/download/nginx-1.21.3.tar.gz
wget http://nginx.org/download/nginx-1.21.3.tar.gz.asc
wget https://nginx.org/keys/mdounin.key
```
After downloading, I will import the public key and check the downloaded archive with the file (Figure 1)

```
gpg --import mdounin.key
gpg --verify nginx-1.21.3.tar.gz.asc nginx_signing.key
```
<center>

![](https://i.imgur.com/eIQZWp6.png)
Figure 1: Checking the signed archive with the program
</center>

### Build & install
First, I will unpack the archive with the program and go to the unpacked directory
```
tar xzf nginx-1.21.3.tar.gz
cd nginx-1.21.3/
```

Next, you will need to install the `zlib1g` library (you will not be able to install NGINX without it)
```
sudo apt install zlib1g-dev
```

After installing the necessary library, you can start configuring the installation
I specified the path to the program configuration files, the pid of the file and forcibly indicated that I needed the ssl module

```
sudo ./configure \
--sbin-path=/usr/sbin/nginx \
--prefix=/usr/local/etc/nginx \
--conf-path=/usr/local/etc/nginx/nginx.conf \
--pid-path=/run/nginx.pid \
--with-http_ssl_module
```


After configuration, you can perform the installation
```
make
sudo make install
```

After successful installation, a welcome page will be displayed when accessing the ip address of the server `NGINX` (figure 2)
<center>

![](https://i.imgur.com/OK43EAM.png)
Figure 2: `NGINX` welcome Page
</center>

:::

:::info
>2. Define the root directory and then two virtual hosts (and configure DNS records or wildcard accordingly):
>>aaa.stX.sne21.ru
bbb.stX.sne21.ru

First I added A entries pointing to my nginx web server
```
aaa                IN      A       3.22.164.217
bbb                IN      A       3.22.164.217
```


Next, I need to create a directory for storing logs and gave the www-data user rights to it
```
sudo mkdir /var/log/nginx
sudo chown www-data:www-data /var/log/nginx
```

Then, in `/usr/local/etc/nginx/nginx.conf`, I specified the standard nginx settings, under which the daemon process will be started by the www-data user
I also specified the path to the pid file, files common to all access and error logs. To simplify working with the new subdomains, I specified the loading of all `*.conf` files in the sites directory
Yes, I could specify a log file for all sites separately, but since I don't plan to have a large load, I will specify one common file for troubleshutting
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;

http {
        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;
        include sites/*.conf;
}

```


Since I still don't have a sites directory, I'll create one
```
sudo mkdir /usr/local/etc/nginx/sites
sudo chown www-data:www-data /usr/local/etc/nginx/sites
```

Since I have two subdomains, I will create my own configuration files for them. For example, the configuration for `aaa.st11.sne21.ru` (I listen to port `80`, specify server_name and the root directory of the domain)
```
    server {
        listen       80;
        server_name  aaa.st11.sne21.ru;

        location / {
            root   /var/www/aaa.st11.sne21.ru;
            index  index.html index.htm;
        }

}
```


Similarly, I create a configuration for `bbb.st11.sne21.ru`
Since the root directories are specified, I will create them and give the rights to the `www-data` user

```
sudo mkdir /var/www/aaa.st11.sne21.ru
sudo mkdir /var/www/bbb.st11.sne21.ru
sudo chown www-data:www-data /var/www/*
```


I didn't think much about the content and just copied the `standard html contents` of the `nginx` folder
```
sudo cp /usr/local/etc/nginx/html/* /var/www/aaa.st11.sne21.ru
sudo cp /usr/local/etc/nginx/html/* /var/www/bbb.st11.sne21.ru
```

This completes the setup
:::

:::info
>3. Create a simple, unique HTML page for each virtual host to make sure that the server can correctly serve it.


I did all the setup earlier
Now I'll just correct the contents of the `<h1>` tag to show that the pages are unique 
(figure 3 & figure 4)
<center>

![](https://i.imgur.com/2x8Kp0J.png)
Figure 3: A unique page for a subdomain `aaa.st11.sne21.ru`

![](https://i.imgur.com/NefchwQ.png)
Figure 4: A unique page for a subdomain `bbb.st11.sne21.ru`
</center>
:::

:::info
>4. Check the configuration syntax, start the daemon and enable it at boot time.

To check the correctness of the configuration files, there is a `-t` flag
I can use it like this: `nginx -t`
And in combination with the `-c` flag, you can check the correctness of individual `nginx` configuration files. (figure 5)

<center>

![](https://i.imgur.com/Z75rQGJ.png)
Figure 5: Checking the nginx configuration file
</center>

To simplify working with nginx and solve the problem of starting the service after a reboot, it was decided to create a service file

`sudo nano /lib/systemd/system/nginx.service`


The `After` variable in the `Unit` section just tells you when to start the service,

In the `Service` section, commands are specified to `start/restart or stop` the daemon (also indicated that the directory of temporary files will be private)
```
[Unit]
Description=My nginx server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
```

:::

:::info
>5. Use curl to display the contents of a full HTTP/1.1 session served by your server.
>6. Explain the meaning of each request and reply header.


To send requests using http/1.1, I forcibly specified this in the request
To see all the steps when sending the request and response, I used the -v flag
```
curl --http1.1 -v http://aaa.st11.sne21.ru
curl --http1.1 -v http://bbb.st11.sne21.ru
```


<center>

![](https://i.imgur.com/8Oh9ceG.png)
Figure 6: HTTP request to aaa.st11.sne21.ru
![](https://i.imgur.com/35yMUsI.png)
Figure 7: HTTP request to bbb.st11.sne21.ru
</center>

Request headers:
In my case (since I did not specify additional headers), only the necessary headers were used:
- The method of sending the request (`GET`) and the path on the server also exist (`POST`, `DELETE`, `OPTIONS`, `PUT`)
- The host name (where to send the request)
- User-agent (this header specifies the name of the http client that sent the request). Since I used curl - it is specified in the header
- Accept: */* (indicates that I am ready to accept any content as a response)


Response headers: these are the headers that come from the web server in response to the request
In my case it is:
- HTTPs status (`200` indicates that everything is fine and there are no problems)
- Server (Server information)
- Date (Current date)
- Content type (Plain html/txt)
- Content length
- Last change
- Type of connection (keep alive means that the connection will be established all the time)
- ETag (used to check the web cache)
:::



## Task 2 - SSL/TLS

:::warning
1. Enable SSL/TLS and tune the various settings to make it as secure as possible 2
2. Describe how you created your own certificate(s) e.g. with Let’s encrypt (certbot) or self-signed and re-validate every virtual-host. Explain your security tuning process.
:::

## Implementation:

:::info
> 1. Enable SSL/TLS and tune the various settings to make it as secure as possible 2
> 2. Describe how you created your own certificate(s) e.g. with Let’s encrypt (certbot) or self-signed and re-validate every virtual-host. Explain your security tuning process.

To get an `SSL certificate`, I used the `certbot` program
To install it:
```
sudo apt-get install certbot
```

To get a certificate for my `aaa` and `bbb` subdomains, I need to run the following commands:
```
sudo certbot -d aaa.st11.sne21.ru --webroot
sudo certbot -d bbb.st11.sne21.ru --webroot
```

In the commands, I specified the domain itself with the `-d` flag, and `--webroot` indicates that I already have a web server installed (since by default certbot will try to launch its web server to verify the domain)
 
<center>

![](https://i.imgur.com/yKoKedy.png)
Figure 8: Successfully obtaining a certificate for `aaa`
![](https://i.imgur.com/gQVcFCS.png)
Figure 9: Successfully obtaining a certificate for `bbb`
</center>


After receiving the certificates, you will need to change the configuration for the `aaa` and `bbb` domains
To do this, you need to specify listening on port 443 and specify the path to the certificate and the certificate key

I also:
- Indicated the forced use of ssl protocols (including the new one - TLSv1.3)
- Enabled GSM and RC4 support because they are resistant to time attacks and I used faster ECDHE kits to improve performance.
- Enabled ocsp stapling support (to check the revocation of digital signatures)
- For the greatest security, I installed adding information about hosts to the header of the response. After seeing it, the browser learns that this domain can only be accessed using https (SSL/TLS). It will cache this information for `31536000` seconds, which is 1 year


For `aaa`:
```
listen 443 ssl;
ssl_protocols       TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
ssl_ciphers "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA RC4 !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS";
ssl_prefer_server_ciphers on;
ssl_ocsp                on;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
ssl_certificate /etc/letsencrypt/live/aaa.st11.sne21.ru/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/aaa.st11.sne21.ru/privkey.pem;
```

For `bbb`:
```
listen 443 ssl;
ssl_protocols       TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
ssl_ciphers "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA RC4 !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS";
ssl_prefer_server_ciphers on;
ssl_ocsp                on;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
ssl_certificate /etc/letsencrypt/live/bbb.st11.sne21.ru/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/bbb.st11.sne21.ru/privkey.pem;
```


After applying the changes in the configuration files, you can make sure that the secure connection works for `aaa` and `bbb`
<center>

![](https://i.imgur.com/D8BsbMq.png)
Figure 10: SSL report for `aaa`

![](https://i.imgur.com/PWkF0hN.png)
Figure 11: SSL report for `bbb`
</center>

:::

## Task 3 - Choose one of the options from the following:

:::warning
**1. Web Server Security**
>Create a new virtual host and a HTML page with content (i.e Administrative area).
Enable basic authentication in your web server for your new virtual host. Use password file creation utility and create two users. Verify your work by authenticating against your
webpage.
:::

## Implementation:

:::info
**1. Web Server Security**
>Create a new virtual host and a HTML page with content (i.e Administrative area).
Enable basic authentication in your web server for your new virtual host. Use password file creation utility and create two users. Verify your work by authenticating against your webpage.

To solve the problem, I created another configuration file:
`security.st11.sne21.ru.conf`
I also created a root directory in the image and likeness of past domains

To implement basic authentication, you need to specify this in the configuration
```
location / {
    root   /var/www/security.st11.sne21.ru;
    index  index.html index.htm;
    auth_basic           "security";
    auth_basic_user_file .htpasswd;
}
```

`auth_basic` gives the name for the protected area
`auth_basic_user_file` contains information where the authorization file (.htpasswd) is located


To generate a password, I will use the `htpasswd program`
It is included in the `apache2-utils` software package, so you need to install it first:

```
sudo apt install apache2-utils
```

To create a password, I will specify the `-c` flag indicating the path to the authorization file and the user name
Then the program will ask me to enter a password
After entering the password, information about the new user and his password will be added to the file (Figure 12)

```
sudo htpasswd -c /var/www/security.st11.sne21.ru/.htpasswd test1
sudo htpasswd -c /var/www/security.st11.sne21.ru/.htpasswd test2
```

Example of data after generating a password file
```
test1:$apr1$gv0KuqUN$z0UC.Qq.WfljFjEuMbuKb1
```
Here, the first parameter is the user name (it is in its pure form), and the second parameter reflects the password (in hashed form)

After applying the settings, I will open `security.st11.sne21.ru` in the browser (Figure 13)

It is clearly visible that the web server asked me for a login and password to log in

After entering the correct password, the page is successfully loaded and I will not need to enter the password anymore
<center>

![](https://i.imgur.com/OVLL2Oq.png)
Figure 12: Adding users to the authorization file

![](https://i.imgur.com/ONyDMTe.png)
Figure 13: The first login to the site with basic auth

![](https://i.imgur.com/zWDKWYL.png)
Figure 14: Successfully opening a web page after entering the correct password
</center>


:::

# Section 2: Directory server (Optional)
## Task 1: Directory Server installation and configuration

:::warning
In this section, you will learn how to deploy and configure a directory service. More precisely
deploying your server with one client. Select one of the following directory services of your
choice, please research before choosing.
➢ FreeIPA (medium)
➢ OpenLDAP + phpLDAPadmin (medium)
➢ MS Active Directory (large)
Note: below tasks are based on FreeIPA directory service, you can also perform similar tasks
with the above options.
:::

:::warning
1. Get familiar with your directory server. What features and capabilities? (very short)
2. Prepare the environment, create/use two VMs (i.e server and client)
3. Install the directory server. you DON’T need to install the directory server from the source. (e.g. use apt-get).
Hint: you can add FQDN for your server in the /etc/hosts file to make it resolvable, in
case you don’t have access to your previews lab or DNS server.
4. (For FreeIPA) get a Kerberos ticket for admin and list it. Check the admin account exists in the FreeIPA server.
5. Access/log-in to the web dashboard of FreeIPA server. Create a test user (e.g. your name) for the next task.
:::

## Implementation:

:::info
>1. Get familiar with your directory server. What features and capabilities? (very short)

OpenLDAP uses only the LDAP protocol, which in turn:
- It uses TLS protocol to ensure protection and confidentiality
- Implements reliable authentication via SASL
- Since Unicode is used - it is international
- It is extensible (extended operations can be implemented)
:::

:::info
>2. Prepare the environment, create/use two VMs (i.e server and client)

In gns3 I created two virtual machines (server and client) (figure 15)

<center>

![](https://i.imgur.com/bFzJYah.png)
Figure 15: My scheme
</center>

:::

:::info
>3. Install the directory server. you DON’T need to install the directory server from the source. (e.g. use apt-get).
>>Hint: you can add FQDN for your server in the /etc/hosts file to make it resolvable, in case you don’t have access to your previews lab or DNS server.


### Installing and configuring an LDAP server

In order for my `ldap-server` to work, you need to download the `slapd` `ldap-utils` packages
```
sudo apt-get install slapd ldap-utils
```

During the download process, I will be asked to enter the administrator password and confirm (figure 15, 16)

<center>

![](https://i.imgur.com/GijCKME.png)
Figure 15: Entering the LDAP Administrator password
![](https://i.imgur.com/WgChqwj.png)
Figure 16: Password Confirmation
</center>


Next I will need to reconfigure slapd
```
sudo dpkg-reconfigure slapd
```
There I will need to enter the domain name of my server (figure 17)

<center>

![](https://i.imgur.com/V3K2b4L.png)
Figure 17: Server domain name request during reconfiguration
</center>

Next, you will need to enter the name of the organization (I will leave it equal to the domain name)

<center>

![](https://i.imgur.com/1gQ51Px.png)
Figure 18: Entering the name of the organization
</center>

Next, I will also be asked for the administrator password - I will leave it the same as before


This completes the reconfiguration.

After that, I need to add uri and base information to the ldap.conf configuration file
```
BASE     dc=ldap-server,dc=st11,dc=sne21,dc=ru
URI      ldap://localhost
```

### Installing and configuring phpldapadmin

To install it, run the command below:
```
sudo apt install phpldapadmin
```

After installation, you must first find out my ip address, for this I will use the command `ip addr` (figure 19)

<center>

![](https://i.imgur.com/FNeKGLv.png)
Figure 19: Output of the ip addr command
</center>

Then I will proceed to setting up the phpldapadmin configuration file:
```
sudo nano /etc/phpldapadmin/config.php
```

In it, I specify the name of the server (it will be displayed on the web page), the host address and information about the base (which I set up earlier in the configuration file)
```
$servers->setValue('server','name','st11 ldap server');
$servers->setValue('server','host','192.168.122.240');
$servers->setValue('server','base',array('dc=ldap-server,dc=st11,dc=sne21,dc=ru'));
```

After changing the configuration file, you can go to the phpldapadmin web page, but before that I will specify the host information in my `/etc/hosts` file:
```
192.168.122.240 ldap-server.st11.sne21.ru
```

Now I can go to the phpldapadmin web page at:
`http://ldap-server.st11.sne21.ru/phpldapadmin` (Figure 20)

<center>

![](https://i.imgur.com/U7KvEVd.png)
Figure 20: phpldapadmin webpage
</center>

:::

:::info
>5. Access/log-in to the web dashboard of FreeIPA server. Create a test user (e.g. your name) for the next task.

To log in, I need to change the standard login DN to:
```
cn=admin,dc=ldap-server,dc=st11,dc=sne21,dc=ru
```
Where `cn` is the username
And also enter the user's password

After successfully logging in, I will be greeted by the main page (figure 21)

<center>

![](https://i.imgur.com/SExvGEb.png)
Figure 21: Home page after authorization
</center>
On it I need to open the contents on the left (figure 22) 

<center>

![](https://i.imgur.com/d7L7hDA.png)
figure 22: Content when revealing my main environment
</center>


When you click on the "Create new entry here" button, you will need to create a posixGroup (figure 24) first in the main window (figure 23) and then the user himself (figure 25)

<center>

![](https://i.imgur.com/m8xT4WV.png)
Figure 23: Main Creation window

![](https://i.imgur.com/TkgobWI.png)
Figure 24: Creating a posix group

![](https://i.imgur.com/ekUElts.png)
Figure 25: Creating a user
</center>
:::
ldap://ldap-server.st11.sne21.ru


ldapsearch -x -H "ldap://ldap-server.st11.sne21.ru" -P 3 -LLL -b "dc=ldap-server,dc=st11,dc=sne21,dc=ru"

## References:
1. [PGP public keys](https://nginx.org/en/pgp_keys.html)
2. [Building nginx from Sources](https://nginx.org/en/docs/configure.html)
3. [NGINX systemd service file](https://www.nginx.com/resources/wiki/start/topics/examples/systemd/)
4. [Install OpenLDAP with phpLDAPAdmin on ubuntu](https://medium.com/analytics-vidhya/install-openldap-with-phpldapadmin-on-ubuntu-9e56e57f741e)
5. [Configure LDAP Client on Ubuntu 20.04|18.04|16.04](https://computingforgeeks.com/how-to-configure-ubuntu-as-ldap-client/)
