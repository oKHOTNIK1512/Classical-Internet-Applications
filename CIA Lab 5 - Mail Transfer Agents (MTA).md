---
tags: Classical Internet Applications
---
:::success
# CIA Lab 5 - Mail Transfer Agents (MTA)
Name: Ivan Okhotnikov
:::


## Task 1 - DNS Insecurity

:::warning
>1. Install from source code
>>a) First make sure that your system does not contain a pre-installed version of the MTA of your choice, if so, remove it before you continue.
b) Make sure the source code is retrieved from a secure location. Use the official website for the MTA of your choice.
c) Because it is important that an MTA be correct and secure it is often signed using a digital PGP signature. If your MTA is signed then make sure you have downloaded the correct sources by checking the validity of the key and the signature.
d) There are a number of options that you will have to enter before compilation, so that the functionality can be compiled into the program. Make sure the basic install holds all the necessary functionality. Show the options you configured

>2. Most of the options for an MTA can be found in a configuration file that will be loaded when the MTA starts. It is recommended to start with an example configuration that looks
a lot like what you need for now. Show how you adapt it to your needs.

>3. Configure:
>>a) Add a local account on your experimental machine and make sure that the MTA can
deliver mail to it. Show the required configuration.
b) Add to your log an email received by this account. Do not forget the full headers!
c) Also make sure that any email intended for postmaster@st<X>.sne21.ru is
delivered to this account. Show the full email as delivered to the new account and the
required configuration.
:::

## Implementation

I will be using Exim

### 1. Install from source code
:::info
> a) First make sure that your system does not contain a pre-installed version of the MTA of your choice, if so, remove it before you continue.

Figure 1 shows evidence that I do not have exim installed
<center>

![](https://i.imgur.com/s5b0GrK.png)
Figure 1: Checking whether I have exim installed earlier
</center>
:::

:::info
> b) Make sure the source code is retrieved from a secure location. Use the official website for the MTA of your choice.

I downloaded the latest version of exim from the official website: https://downloads.exim.org/

:::

:::info
> c) Because it is important that an MTA be correct and secure it is often signed using a digital PGP signature. If your MTA is signed then make sure you have downloaded the correct sources by checking the validity of the key and the signature.

First you need to download the key and add it to gpg
```
wget https://downloads.exim.org/Exim-Maintainers-Keyring.asc
gpg --import Keyring.asc
```

Then I download the archive itself with the exim program and its signature

```
wget https://downloads.exim.org/exim4/exim-4.94.2.tar.gz
wget https://downloads.exim.org/exim4/exim-4.94.2.tar.gz.asc
```

After downloading the necessary files, I can check the file using the command: `gpg --verify signature.asc file.tar.gz`

```
gpg --verify exim-4.94.2.tar.gz.asc exim-4.94.2.tar.gz
```

<center>

![](https://i.imgur.com/pYSGLNa.png)
Figure 2: Checking the signature of the archive with the program that I downloaded
</center>

:::

:::info
> d) There are a number of options that you will have to enter before compilation, so that the functionality can be compiled into the program. Make sure the basic install holds all the necessary functionality. Show the options you configured

>2. Most of the options for an MTA can be found in a configuration file that will be loaded when the MTA starts. It is recommended to start with an example configuration that looks a lot like what you need for now. Show how you adapt it to your needs.

Before you start installing `exim`, you need to install the necessary libraries and programs:

```
sudo apt-get update -y
sudo apt-get install db-util libssl-dev gcc libpcre3-dev make db5.3 libdb-dev pkg-config mailutils -y
```

After successfully installing the necessary dependencies, I can unzip the archive with the program and start preparatory work before installing `exim`
```
tar xzf exim-4.94.2.tar.gz
cd exim-4.94.2
```

Since the Makefile contains a lot of comments by default, I will clear them with: `grep -ve "^#" -ve "^$"` and I will immediately write the data to the installation directory: `Local`

```
cat ./src/EDITME | grep -ve "^#" -ve "^$" > Local/Makefile
```

Before configuring the Makefile, I need to make sure that the exim user is created (this is important for further installation)

```
sudo groupadd -g 31 exim &&
sudo useradd -d /dev/null -c "Exim Daemon" -g exim -s /bin/false -u 31 exim
```

After creating a user, you must manually create directories for storing configuration files and logs
The following commands will allow you to do this:

```
sudo mkdir -m 0755 /usr/local/etc/exim4
sudo chown exim:exim /usr/local/etc/exim4/
sudo mkdir -m 0750 /var/log/exim4
sudo chown exim:exim /var/log/exim4
```

The preparation is completed. Next, I can start configuring the installation file
```
nano Local/Makefile
```
In this file, I set the directory for storing the binary file, the location of the configuration file, specify the name of the user (whom I created earlier) and allow the use of `OPENSSL`


Also, a very important point is that I allow authorization using `AUTH_PLAINTEXT`. Since I will continue to use Mailgun to send emails
```
BIN_DIRECTORY=/usr/local/bin
CONFIGURE_FILE=/usr/local/etc/exim4/exim.conf
EXIM_USER=exim
USE_OPENSSL=yes
USE_OPENSSL_PC=openssl
AUTH_PLAINTEXT=yes
```

The Makefile setup is complete and now I can perform the installation:
```
make 
sudo make install
```

After I have installed `exim`, I will specify the path for storing log files in the program's configuration files (since I have a long and thorny path of catching errors ahead of me, I can't do without logs)
<!-- 
```
nano /usr/local/etc/exim4/exim.conf
```
 -->
```
log_file_path = /var/log/exim4/exim_%slog
```

The basic installation is complete.
Now I will proceed to setting up the configuration file

I will specify the port for the exim daemon to work, the primary host and allow relay from other addresses:

```
daemon_smtp_ports = 25  
primary_hostname = st11.sne21.ru
hostlist   relay_from_hosts = 0.0.0.0
```

To start the daemon, I run the command: `exim -bdf`
Since the program does not start immediately in the background, I will use redirecting the file descriptor to the void: `> /dev/null&`

The full command will look like this:
```
exim -bdf > /dev/null&
```

To kill the daemon process, I will use the command below:
```
sudo pkill exim
```

:::


### 3. Configure:

:::info
> a) Add a local account on your experimental machine and make sure that the MTA can deliver mail to it. Show the required configuration.
> b) Add to your log an email received by this account. Do not forget the full headers!

Adding a user to Ubuntu is done using the command `adduser`:

```
sudo adduser st11
```
The whole process is visually shown on `figure 3`

<center>

![](https://i.imgur.com/8DzHUeb.png)
Figure 3: The process of creating a user
</center>

To send an email to this user, I will use the `exim -v` command:

```
exim -v st11@localhost
From: st11@localhost
To: ubuntu@localhost
Subject: Test

Test message
```
To view the emails of a specific user, I can look into the fileЖ `/var/mail/st11` (figure 4)

```
sudo cat /var/mail/st11
```
<center>

![](https://i.imgur.com/uRZuIV0.png)
Figure 4: A message received by the user `st11`
</center>




After sending the letter, an entry about sending letters appeared in the `exim` logs:

<center>

![](https://i.imgur.com/K8zvaGS.png)
Figure 5: An entry in the logs about sending a message
</center>
:::

:::info
> c) Also make sure that any email intended for postmaster@st<X>.sne21.ru is delivered to this account. Show the full email as delivered to the new account and the required configuration.

In order for an email from postmaster to get to st11, I need to specify this in the file `/etc/aliases` (figure 6)
```
sudo nano /etc/aliases
```
<center>

![](https://i.imgur.com/2OI6u90.png)
Figure 6: required file settings `/etc/aliases` to redirect emails from postmaster to st11
</center>

After configuring aliases, I apply these settings. To do this, I use the newaliases command:
```
sudo newaliases
```

It's time to move on to the tests. I will send an email to the post office postmaster@st11.sne21.ru (figure 7)
And I will check the contents of the mail file for st11 (`/var/mail/st11`) (figure 8)
<center>

![](https://i.imgur.com/557BijS.png)
Figure 7: Sending an email from my personal mailbox to postmaster@st11.sne21.ru

![](https://i.imgur.com/RKBlb6s.png)
Figure 8: Received an email from my personal mailbox
</center>

Since the email was successfully received, I configured exim correctly
:::

## Task 2 - Sending mail - email validation - SPF & DKIM

:::warning
For many people unsolicited commercial email, or rather SPAM, is a big problem. There are many ways to filter SPAM, each one having advantages and disadvantages. Examples include domain keys, SPF records, DNS block lists, greylisting, reverse checks, tarpitting, Bayesian filters, whitelists, etc. A lot of viruses and malware are transported over email (as well as the World Wide Web). Because viruses can cause a lot of trouble, discarding viral messages is a nice service to offer to your users.
Due to this reason cloud providers block standard SMTP port (25) for all outbound connections on their cloud instances to protect their IP address pools from having bad reputation.
Since your setup is deployed in the cloud, you are going to face that issue. To overcome this situation there are relay email servers that can accept your outgoing mail on other ports (587, 465, 2525) and forward it to the original destination server (that only works on port 25).
To provide such functionality this servers
- Provide you with credentials that should be used to submit your mail
- Require you to placing SPF/DKIM records that 2
- serves as a form of verification that you the domain owner
- used by original mail destination server to verify that relay server is eligible to
send mail from your domain
4. Write a small paragraph that highlights the advantages and disadvantages of SPF and DomainKeys Identified Mail (DKIM). What would you choose at a first glance and why?
5. Set up your mail server to use a relay server to be able to send outgoing mail for different domains (contact your TA to obtain address/credentials to the relay server and SPF/DKIM records).
6. Test how your mail is delivered to commonly known mail servers (f.e. gmail). Provide full email/MTA headers to see how SPF/DKIM were delivered.
:::

## Implementation:

:::info
> 4. Write a small paragraph that highlights the advantages and disadvantages of SPF and DomainKeys Identified Mail (DKIM). What would you choose at a first glance and why?

**SPF** allows you to avoid the need to confirm the email address. But this method has a drawback: the forwarded messages do not pass SPF authentication


**DKIM** also guarantees that the contents of emails will not be compromised, since it creates hashes for the header and the body of the email itself. This method allows you to check the integrity of the email itself.
To verify the sent message, the public key is used, which is located in the **TXT** record

Using **DKIM** is more reliable (because it implements a digital signature) and it is better to use it
:::

:::info
>5. Set up your mail server to use a relay server to be able to send outgoing mail for different domains (contact your TA to obtain address/credentials to the relay server and SPF/DKIM records).

I chose mailgun as the relay server
### Preparatory work
To configure it in exim, you need to perform preparatory work:
1. Registered in `mailgun`
1. I verified my domain name, for this I added records to my `DNS server`, which were provided by `mailgun` (figure 9):

```
@                    IN      MX      40       mxa.mailgun.org.
@                    IN      TXT     "v=spf1 include:mailgun.org ~all"
smtp._domainkey      IN      TXT     "k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDQIlf1WFdBXq+kAOfUECBc2b9pYVtGLemqGAgZLvLeGz8gUc9GCLSg4lro/61fYSj4MYRFw6LEEP+6e1oKUC8FP8Jf58kxplJuPBtG7pdMzeuBb902dRZ4/eKgJKEY7v8mF5nV/iYIVQGzQBf1dWyX6OkULO1ygBd+nF/cNaI61QIDAQAB"
```

<center>

![](https://i.imgur.com/fZ9jYLT.png)
Figure 9: `mailgun` verification page
</center>

3. After successful verification, get a login and password from the FTP server (`SMTP credentials tab`)

### Preparatory work has been completed

The next step is to configure exim to work with the relay server
(Important note, nothing will work during installation without first specifying the `AUTH_PLAINTEXT` module)

In the configuration file, I first added variables that will contain information about SMTP credentials:
```
SMARTHOST_ADDR = smtp.mailgun.org
SMARTHOST_PORT = 587
SMARTHOST_USER = postmaster@st11.sne21.ru
SMARTHOST_PASS = ***
```

After setting the variables, they must be handled correctly:

After the line: begin routers below, I added smart host initialization, indicated which transport should be used and also indicated that it is necessary to relay all domains except local

```
.ifdef SMARTHOST_ADDR
smarthost:
  driver = manualroute
  domains = ! +local_domains
  transport = smarthost_smtp
  route_list = * SMARTHOST_ADDR
  host_find_failed = defer
  no_more
.endif
```

After the line: `begin transports`, I added my own implementation, which uses my `SMARTHOST_ADDR` as well as the port that I specified earlier

```
.ifdef SMARTHOST_ADDR
smarthost_smtp:
  driver = smtp
  .ifdef SMARTHOST_PORT
  port = SMARTHOST_PORT
  .endif
  hosts_require_auth = SMARTHOST_ADDR
  hosts_require_tls = SMARTHOST_ADDR
.endif
```

After the line: `begin authenticators`, I implemented my authorization method using my username and password pair

```
.ifdef SMARTHOST_ADDR
smarthost_login:
  driver = plaintext
  public_name = LOGIN
  hide client_send =:SMARTHOST_USER:SMARTHOST_PASS
.endif
```
:::

:::info
> 6. Test how your mail is delivered to commonly known mail servers (f.e. gmail). Provide full email/MTA headers to see how SPF/DKIM were delivered.

I sent an email to my personal email account using the command: `exim -v ohotnikovivan@yandex.ru` (figure 10)
This command also allows you to see the logs when sending an email. In them, you can notice the use of `mailgun` as a relay server

<center>

![](https://i.imgur.com/xBZFs3r.png)
Figure 10: Sending mail using `exim`
</center>

On Figure 12, it is perfectly clear that the sender is confirmed and the letter contains encryption and signature (TXT records of my domain name are checked)
By going to the property of this letter (figure 12), you can see all the properties of this letter


It is not difficult to notice that there is a `DKIM-Signature` in the headers, there is a signature here, which can be checked using the `DKIM-key`, which is located in the `TXT` record of my domain (public key)
The private key is stored in mailgun

<center>

![](https://i.imgur.com/3xgXqMv.png)
Figure 11: The letter that came from `my domain mailbox`
</center>
<center>

![](https://i.imgur.com/XRBTPDd.png)
Figure 12: The property of the letter that came to my `personal mailbox`
</center>
:::




## Task 3 - Mail backup

:::warning
7. You have backup:
a) Adapt the DNS information for your domain, so that the backup MTA on your partner’s server can be found.
(...your partner configures its MTA as a backup MTA...)
b) Validate by shutting your service down and sending a message to your domain
(...your partner sees its logs and where the message is temporarily stored...)
c) Bring your service back up and wait.
8. You provide backup:
a) Make your MTA act as a backup for your partner’s domain.
b) Show the logs while doing your mate’s acceptance test and show where the message is temporarily stored.
c) Once your partner’s MTA is back online, eventually force an immediate delivery and
show your mail logs.
:::

## Implementation:

### 7. You have backup:

:::info
> a) Adapt the DNS information for your domain, so that the backup MTA on your partner’s server can be found (...your partner configures its MTA as a backup MTA...)

To do this, I added another mx record with a lower priority (in this case, it will only be used when my main server is not responding)

```
@                  IN      MX      10       mx2; MAIN
@                  IN      MX      20       st2mx; BACKUP
...
st2mx              IN      A       18.189.171.89
mx2                IN      A       3.135.210.50
```
:::

:::info
> b) Validate by shutting your service down and sending a message to your domain
(...your partner sees its logs and where the message is temporarily stored...)

I sent emails from my mailbox to `ubuntu@st11.sne21.ru`

<center>

![](https://i.imgur.com/nxp5QFs.png)
Figure 13: My partner's logs
</center>
:::

:::info

>c) Bring your service back up and wait.

Since the sending is done using mailgun, unsent emails are returned back to me (to avoid loops)
Since the partner cannot check the availability of the main host via port 25 on AWS, the partner can force emails to be sent to the queue (followed by sending emails manually)

:::

### 8. You provide backup:
:::info
> a) Make your MTA act as a backup for your partner’s domain.
> b) Show the logs while doing your mate’s acceptance test and show where the message is temporarily stored.

The same situation is with me
I can't get an email back from mailgun. But there is one solution: I can specify the partner's domain

```
queue_domains=st2.sne21.ru
```

In this case, all emails that will arrive at the partner's domain will fall into the queue. (Figure 14)

Unfortunately, you can only send them manually when I make sure that the partner's machine is available

I can check the queue using the commands (Figure 15):

```
exim -bp | exiqsumm
```

<center>

![](https://i.imgur.com/tmCwpZ5.png)
Figure 14: My logs when receiving an email to the partner's domain
</center>

<center>

![](https://i.imgur.com/uolOT7z.png)
Figure 15: List of my queues
</center>

:::

:::info
> c) Once your partner’s MTA is back online, eventually force an immediate delivery and show your mail logs.

To send all emails from the queue, I can use the command:
```
exim -q -v 
```

If I need to send only 1 email from the queue, I can use the command:
```
exim -M message_id
```
Where message_id is the ID of the message from the queue


<center>

![](https://i.imgur.com/HLHqOAD.png)
Figure 16: My logs after the immediate start of queues
</center>
:::


## Task 4 - Mailing loops

:::warning
9. Create an email loop with your partner from domain to domain using email aliases.
10. Send an email to the loop using your own email address and see what happens on your MTA.
11. Can you change the behaviour of your MTA in response to this loop?
:::
## Implementation:


:::info
> 9. Create an email loop with your partner from domain to domain using email aliases.

The contents of my file `/etc/aliases`

```
postmaster: root
root: ivan            
ivan: ifeanynyi@st2.sne21.ru
```

The contents of the `/etc/aliases` file from my partner
```
postmaster: root
root: ifeany              
ifeany: ivan@st11.sne21.ru
```

After changing the aliases, you need to apply them:
```
sudo newaliases
```
:::


:::info
> 10. Send an email to the loop using your own email address and see what happens on your MTA.

After sending an email from your personal mailbox to the mailbox `ivan@st11.sne21.ru` I see an email being forwarded to my partner (Figure 17)

<center>

![](https://i.imgur.com/KYY6e13.png)
Figure 17: The contents of my logs after sending an email to the mail `ivan@st11.sne21.ru`
</center>

Since the emails did not arrive, we decided to change the mailbox to ubuntu (for me and my partner, so the mail will be configured further in the logs ubuntu@st11.sne21.ru and ubuntu@st2.sne21.ru)

The email does not reach my partner, although it is written in the logs: `Completed` 
(Figure 18)
To deal with this, I went to the `mailgun personal account` and looked at the logs
(Figure 19)

<center>


![](https://i.imgur.com/EU561rC.png)
Figure 18: The log of successful sending of an email using `mailgun`

![](https://i.imgur.com/e4GtuNY.png)
Figure 19: The contents of the logs in mailgun
</center>


The likely problem is that mailgun prevents sending an email from me to my partner. Since in this case a loop will be called and this will disrupt the operation of this service

:::



:::info
> 11. Can you change the behaviour of your MTA in response to this loop?

Since the loop was not called, I can refer to the official exim documentation

It says that the flag: `received_headers_max` allows you to avoid a loop (since the number of "Received:" headers is counted). By default, this parameter is set to `30`. If the parameter is exceeded, the delivery will stop and the logs will display an error message.
:::


## Task 5 - Virtual Domains

:::warning
12. Create a new subdomain within your domain and add an MX entry to it.
13. Then extend your MTA configuration to handle virtual domains, and have it handle the email for the newly created domain.
14. Validate that you are now receiving emails for both domains.
:::

## Implementation:

:::info
> 12. Create a new subdomain within your domain and add an MX entry to it.

I created an `MX record` for the `virtual` subdomain and sent it to the previously existing `A` record of the `mx2` subdomain (I created this record at the beginning of the laboratory work, since I was working with `exim` on an `aws instance` separate from the `DNS server`)
```
virtual            IN      MX      10    mx2
```

:::

:::info
> 13. Then extend your MTA configuration to handle virtual domains, and have it handle the email for the newly created domain.


To use the subdomain, I changed some data in the configuration file:
```
primary_hostname = *.st11.sne21.ru
domainlist local_domains = *.st11.sne21.ru
domainlist relay_to_domains = *.st11.sne21.ru
```

With the help of wildcard, I allowed the use of my subdomains for receiving emails

:::

:::info
> 14. Validate that you are now receiving emails for both domains.


I sent an email from my personal email and was able to view it for the user `st11@virtual.st11.sne21.ru` 
(Figure 20)

After studying the contents of the letter, it is clearly visible that it came to the virtual subdomain.
It has a signature from `Yandex` (since my personal mail is supported by this company) 
(Figure 21)

<center>

![](https://i.imgur.com/evAcIjJ.png)
Figure 20: Sending an email from my personal mailbox
</center>

<center>

![](https://i.imgur.com/TmufJeJ.png)
Figure 21: The content of the email that was sent to the user `st11`
</center>

:::

## Task 6 - Transport Encryption

:::warning
15. Questions
a) Which one is better, SSL/TLS or STARTTLS, why?
b) Which one is actually in use for SMTP?
16. Task
a) Add transport encryption to your MTA.
b) Eventually force the transport to be encrypted only (refuse non encrypted transport). 
c) Proceed with validation (proof or acceptance testing), as usual.
:::

## Implementation:

### 15. Questions
:::info
> a) Which one is better, SSL/TLS or STARTTLS, why?

SMARTTLS differs from the other two in that it makes the connection secure AFTER the connection is established. This option is less confidential.

:::

:::info
> b) Which one is actually in use for SMTP?

For SMTP security, SSL/TLS is mainly used at the moment
:::

### 16. Task

During the installation, I set the parameter in the Makefile:
:::info
> a) Add transport encryption to your MTA.

To do this, I specified this parameter at the beginning of the exim installation:

```
USE_OPENSSL=yes
```
:::

:::info
> b) Eventually force the transport to be encrypted only (refuse non encrypted transport). 


To begin with, I created my own certificate (figure 22):

```
sudo openssl req -x509 -sha256 -days 9000 -nodes -newkey rsa:4096 -keyout /etc/exim.key -out /etc/exim.cert
```

Then I specified it in the settings and allowed connection from all hosts to port 465

```
tls_advertise_hosts = *
tls_on_connect_ports = 465
tls_certificate = /etc/exim.cert
tls_privatekey = /etc/exim.key
```

<center>

![](https://i.imgur.com/qeZDfmV.png)
Figure 22: The process of creating your own certificate
</center>

:::

:::info
> c) Proceed with validation (proof or acceptance testing), as usual.



Using the openssl program, I can check which certificate is used and whether encryption is used at all

```
openssl s_client -connect localhost:25 -starttls smtp
```

On figure 23 you can see that my new certificate is being used


<center>

![](https://i.imgur.com/q1ZgQEb.png)
![](https://i.imgur.com/edA6rMs.png)

Figure 23: Information about my ssl certificate

</center>
:::

## BONUS (2 points) - Task 7 - Anti-spam filters

:::warning
17. Investigate what generic anti-spam open source software packages are out there, choose one, download it (compile it if necessary) and configure your MTA to use it. Make sure that in your MTA group there are 2 different anti-spam solutions implemented!
:::

## Implementation:

В exim уже существует встронная защита от спама.
Так называемый белый и черный список отправителей
Можно также указать запрещенные доменные адреса отправителей. Этот вариант в совокупности с черным списком оправителей является более эффективным. НО данные варианты работают на уровне примитивной защиты от спама. 

Существует также програмное обеспечение, которое в совокупности с exim дает более умную защиту от спама.
Одним из таких решений является Greylisting

Greylisting - технология, которая анализирует множество параметров, начиная от заголовков письма, заканчивая содержимым письма


## References:
1. [Chapter 4 - Building and installing Exim](https://www.exim.org/exim-html-current/doc/html/spec_html/ch-building_and_installing_exim.html)
2. [Adding a certificate to exim4](https://vacadem.ru/blog/linux-unix-and-other/adding-a-certificate-to-exim4.html)
3. [Testing Exim](https://github.com/Exim/exim/wiki/TestingExim)
4. [Compile and Install Exim from Source](https://www.lisenet.com/2015/compile-and-install-exim-from-source-with-ldap-and-mysql-lookup-support-on-ubuntu-14-04/)
5. [List of useful commands to manage Exim mail server](https://www.hostpapa.es/knowledgebase/list-useful-commands-manage-exim-mail-server/)
6. [Main configuration of exim](http://www.exim.org/exim-html-3.20/doc/html/spec_11.html)