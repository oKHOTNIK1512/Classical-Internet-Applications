---
tags: Classical Internet Applications
---
:::success
# CIA Lab 3 - Domain Name System (DNS)
**Name: Ivan Okhotnikov**
:::


## Task 1 - Downloading and Installing a Caching Name Server

### 1.1. - Validating the Download
:::info
The https://www.isc.org website provides signature files in addition to the BIND tarball.
These can be used to check if you have downloaded the version they intended to distribute.
The Unbound website uses a different mechanism, in particular, a modification detection
code, better known as a cryptographic hash.
➢ Why is it wise to verify your download?
➢ Download the BIND tarball (also if you are doing the Unbound+NSD part) and
check its validity using one of the signatures.
➢ Which mechanism is the best one to use (signatures or hashes)? Why?
:::

### Implementation:

:::info
> Why is it wise to verify your download?

 We need to check the file for several reasons:
 1. check the integrity of the downloaded file
 1. helps to make sure that the downloaded file has not been compromised
:::


:::info
> Download the BIND tarball (also if you are doing the Unbound+NSD part) and check its validity using one of the signatures.
1. Download public key from [this link](https://www.isc.org/202122pgpkey/)
2. Import public key to gpg use command (Figure 1) `gpg --import 202122pgpkey`
3. Verify download file use signature (Figure 2) `gpg --verify signature.asc file` 

<center>

![](https://i.imgur.com/r6yci5a.png)
Figure 1: Import public key to gpg

![](https://i.imgur.com/laTrHrt.png)
Figure 2: Verify tar.xz file use signature file
</center>
:::

:::info
> Which mechanism is the best one to use (signatures or hashes)? Why?

It is better to use a signature, because it allows you to verify the authenticity of the sender using a key pair (public and private)
In the case of a hash - it simply shows that the message was delivered completely
:::

### 1.2 - Documentation & Compiling
:::info
Most things about DNS are described and standardized in so-called “Request For Comments
(RFCs)”, created and published by the Internet Engineering Task Force (IETF). A good DNS
RFC to start with is RFC 1034: Domain Names - Concepts and Facilities.
Compiling and installing BIND, Unbound and NSD servers is simple and consists of the
usual sequence of commands on most systems: ./configure; make and make install
First, make sure your installation does not contain a previous version of the servers, as that
can really mess things up (show how to check).
Next, configure, compile and then install the servers in the directory /usr/local/.
Make sure each server will look for its configuration files in
- BIND - /usr/local/etc/bind
- Unbound - /usr/local/etc/unbound
Let the server write its state information, such as the named.pid or unbound.pid file, in
/var/run.
Information about which configuration options to use to achieve the result you can find in
the README files.
➢ What is the difference between /etc, /usr/etc, /usr/local/etc
:::

## Implementation:


### Preliminary works

Before installation, you need to install the necessary programs and libraries to work

```
sudo apt-get update -y
sudo apt-get install libunbound-dev openssl net-tools libudev-dev libssl-dev libevent-dev libexpat1-dev make gcc -y
```


I will also download `UNBOUND` and `NSD`

```
wget https://nlnetlabs.nl/downloads/unbound/unbound-1.13.2.tar.gz
wget https://nlnetlabs.nl/downloads/nsd/nsd-4.3.7.tar.gz
```

I will unpack the downloaded archives
```
tar xzf unbound-1.13.2.tar.gz
tar xzf nsd-4.3.7.tar.gz
```


### Installing UNBOUND

I will go to the unpacked folder for unbound

```
cd unbound-1.13.2
```

First I need to configure the installation

```
sudo ./configure --prefix=/usr/local --sysconfdir=/usr/local/etc --with-pidfile=/var/run/unbound.pid --with-libnghttp2
```

`--prefix` - the prefix for the folder in which we will perform the installation
`--sysconfdir`- the folder where the configuration will be located
`--with-pidfile` - to specify the location of the pid file

Then it remains only to install

```
make
sudo make install
```

I will create an unbound user and give him the rights

```
sudo groupadd -g 499 unbound
sudo useradd --system -M -d /var/chroot/unbound -s /sbin/nologin -g unbound -u 499 unbound
```

Next, I need to create a file for recording logs
```
sudo touch /usr/local/etc/unbound/unbound.log
sudo chown unbound:unbound /usr/local/etc/unbound/unbound.log
sudo chmod 640 /usr/local/etc/unbound/unbound.log
```

Completing the installation of unbound

```
sudo unbound-control-setup
```

I can check the version of the installed unbound package using the command

`unbound -V`
<center>

![](https://i.imgur.com/80F5quo.png)
Figure 3: Result of checking the unbound version
</center>

:::info
> What is the difference between /etc, /usr/etc, /usr/local/etc
 
`/etc/` - required for system program configuration files
`/usr/etc/` - required for program configuration files
`/usr/local/etc/` - required for program configuration files that the user has installed manually
:::

## Task 2 - Configuring Caching Name Server

:::info
Compiling and installing a server is relatively simple, but configuring a DNS server is not
trivial. To keep things simple, we will start with BIND and Unbound running as a
caching-only name server. This type of name server does not control any zone data.
:::


:::info
> Why are caching-only name servers still useful?

Because they allow you to speed up the response time from the original dns servers and reduce the load on them
:::

The first thing I do when setting up unbound is to make a backup copy of the configuration file

```
sudo cp /usr/local/etc/unbound/unbound.conf /usr/local/etc/unbound/unbound.conf.dist
sudo touch /usr/local/etc/unbound/unbound.conf
```

### Access control

Now you can start changing the configuration file.
I filled it with the following contents:
```
server:
    verbosity: 2 #logging level
    interface: 0.0.0.0 #listening interface 
    chroot: "/usr/local/etc/unbound" #parent folder
    directory: "/" #working directory
    logfile: "/unbound.log" #log file 
    do-not-query-localhost: no #so that you can work with nsd
    access-control: 10.1.1.1/24 allow #access control
    access-control: 127.0.0.1/0 allow #access control
```

In my setup, I allowed connection for my local network, as well as for external connections from the `10.1.1.1` subnet (you will need it in the following tasks)

:::info
> Why access control is important for recursive server?

Using access control, you can specify a range of addresses that can connect to our server
:::

### Testing

After changing the configuration file, I need to check whether it is filled in correctly and whether the syntax is not broken

I can do this using the command below:
```
sudo unbound-checkconf
```
<center>

![](https://i.imgur.com/QBb0Ru3.png)
Figure 4: Result of checking the configuration file
</center>

:::info
> Why do the programs return a result value?
> 
The program outputs a value so that you can check the syntax of the configuration file before restarting the server.
If successful, a confirmation will be displayed that everything is fine
In case of problems, I will find out about them in advance
:::

## Task 3 - Running Caching Name Server

:::info
You can now start the DNS server by hand using named -g -d2 , or unbound -d -vv , respectively (it will start the daemon with debug level 2 - read the manual page for more information). 
But it is better to use a “name server control” tool to start and stop the server. For BIND this tool is called rndc , and for Unbound, it is called unbound-control . You’ll have to adapt the configuration to enable it. Also, it would be better to configure the name server to write
debug information to a log file. This is described in Chapter 3 of the BIND ARM, or the unbound-control manual page, respectively. 

1. Show the changes you made to your configuration to allow remote control 
2. What other commands/functions does rndc/unbound-control provide? What 
3. What is the difference between stop -> start and reload? 

To use your own server name you will need to adapt resolv.conf. Take into account that on recent distributions this file is automatically generated. 

What do you need to put in resolv.conf (and/or other files) to use your own name
server? 

Now use the tools and scripts provided with your distribution to test your name server
( Hint: dig, drill - study these tools to understand how to correctly q uery your server or f.e. google’s or
cloudflare’s dns). 

Show that your queries are successfully resolved and cached by also inspecting the server’s log file (  Hint: configure log verbosity level) . 
:::

## Implementation:

:::info
You can now start the DNS server by hand using named -g -d2 , or unbound -d -vv , respectively (it will start the daemon with debug level 2 - read the manual page for more information). 
But it is better to use a “name server control” tool to start and stop the server. For BIND this tool is called rndc , and for Unbound, it is called unbound-control . You’ll have to adapt the configuration to enable it. Also, it would be better to configure the name server to write
debug information to a log file. This is described in Chapter 3 of the BIND ARM, or the unbound-control manual page, respectively. 
:::

`Unbound` has its own startup management utility
To enable it, you need to specify `remote-control` in the config file


:::info
> Show the changes you made to your configuration to allow remote control

To allow the use of `remote-control`, you must enter the following contents at the end of the configuration file: 
```
remote-control:
        control-enable: yes
```

It will enable `unbound-control`
:::

:::info
> What other commands/functions does rndc/unbound-control provide? What

I can view the main commands for unbound-control using the command:
`sudo unbound-control -h`

Here are the main ones:

```
start - start the server
stop - stop the server
reload - restart the server
flush - to clear the cache
stat - for viewing statistics
dump_cache - to output the contents of the cache
load_cache - for loading information to the cache
lookup - for viewing nameservers for name
verbosity - to change the caching mode
```
:::

:::info
>What is the difference between stop -> start and reload?

**Stop** - to stop the unbound server
**Start** - to start the unbound server
**Reload** - to restart the unbound server
:::

:::info
>What do you need to put in resolv.conf (and/or other files) to use your own name server?

To use my server, I need to insert the line:
`nameserver 127.0.0.1`
:::

:::info
> Show that your queries are successfully resolved and cached by also inspecting the server’s log file

For example, let's try to ping the domain `vk.com`
<center>

![](https://i.imgur.com/0KzCdI3.png)
Figure 5: Result of the ping command
</center>
After checking the log file, we can verify the operation of our server
<center>

![](https://i.imgur.com/B8tXD4A.png)
Figure 6: Part of the logs of the `unbound` program
</center>

You can check the caching function using the command: 
`sudo unbound-control dump_cache`
<center>

![](https://i.imgur.com/FHNpDDz.png)
Figure 7:  Output of cached information
</center>


:::

## Task 4 - Authoritative Name Server

:::info
If you are using Unbound+NSD, you should now install and test the NSD server before
going to the next step. You can get it from https://nlnetlabs.nl/projects/nsd/about/. Follow
the same steps as above for Unbound to configure, make and install it, also with the
/usr/local prefix.
*Note: in order to run Unbound and NSD together you will need to decide how they should share the machine's network interfaces and ports. Again, think of use cases for authoritative and recursive name servers and make a decision.*


Now that you have checked that your configuration works correctly, you can set it up to serve your own subdomain - stX.sne21.ru 
But before, please answer: 
➢  What is a private DNS zone? Is stX.sne21.ru private? 
➢  What information was needed by TAs so they can implement the delegation? 
The zone file format is standardized, so it is the same for both BIND and NSD.

The subdomain stX.sne21.ru is delegated to your experimentation server - ns.stX.sne21.ru . 
Create a forward mapping zone file for your domain. It must contain the following resource records: 
➢  2 MX records. Make sure that mail for your domain is delivered to your own
public IP. We will use the MX records later on. 
➢  4 A or AAAA records. Use your imagination. 
➢  2 CNAME records. 
Then: 
➢  Show the resulting zone file. 
➢  Add a reference to your zone file to named.conf or nsd.conf, respectively. 
➢  Restart or reload the name server and test your configuration using the provided tools (  Hint: dig, drill) . 
:::

## Impementation
:::info
If you are using Unbound+NSD, you should now install and test the NSD server before
going to the next step. You can get it from https://nlnetlabs.nl/projects/nsd/about/. Follow
the same steps as above for Unbound to configure, make and install it, also with the
/usr/local prefix.
Note: in order to run Unbound and NSD together you will need to decide how they should share the machine's network interfaces and ports. Again, think of use cases for authoritative and recursive name servers and make a decision.
:::

### Installing NSD

I will go to the unpacked folder for nsd

```
cd nsd-4.3.7
```
First I need to configure the installation

```
sudo ./configure --prefix=/usr/local --sysconfdir=/usr/local/etc --with-pidfile=/var/run/nsd.pid
make
sudo make install
```

We create an nsd user and also give him rights
```
sudo groupadd -g 500 nsd
sudo useradd --system -M -d /var/chroot/nsd -s /sbin/nologin -g nsd -u 500 nsd
```

Completing the installation of nsd
```
sudo nsd-control-setup
```

I create a file for logs and give the rights to the group you

```
sudo touch /usr/local/etc/nsd/nsd.log
sudo chown nsd:nsd /usr/local/etc/nsd/nsd.log
sudo chmod 640 /usr/local/etc/nsd/nsd.log
```

Creating a directory for storing local zones
`sudo mkdir /usr/local/etc/nsd/zones`

Next, I need to give the rights to the folder `/usr/local/var/db/nsd/` to the nsd user, since this folder will be needed for the operation of the nsd server

`sudo chown nsd:nsd /usr/local/var/db/nsd`


It remains only to fill in the configuration file for nsd
To get rid of compatibility problems with unbound and nsd, I will change the working port to 5358 in the configuration

```
server:
port: 5358
zonesdir: "/usr/local/etc/nsd/zones" #folder for zones
logfile: "/usr/local/etc/nsd/nsd.log" #the path to the logs
remote-control:
        control-enable: yes
```

To check the correctness of the syntax of the filled configuration file, you will need to make a check
```
sudo nsd-checkconf /usr/local/etc/nsd/nsd.conf
```


:::info
>What is a private DNS zone? Is stX.sne21.ru private? 

A private DNS zone is a zone that is not publicly accessible.
`stX.sne21.ru` it is private, because it is not publicly available
:::

:::info
> What information was needed by TAs so they can implement the delegation?

I need to transmit information about the ip address of my dns server
:::

:::info
The zone file format is standardized, so it is the same for both BIND and NSD.
The subdomain stX.sne21.ru is delegated to your experimentation server - ns.stX.sne21.ru.
Create a forward mapping zone file for your domain. It must contain the following resource
records:
➢ 2 MX records. Make sure that mail for your domain is delivered to your own
public IP. We will use the MX records later on.
➢ 4 A or AAAA records. Use your imagination.
➢ 2 CNAME records.
Then:
➢ Show the resulting zone file.
➢ Add a reference to your zone file to named.conf or nsd.conf, respectively.
➢ Restart or reload the name server and test your configuration using the provided
tools (Hint: dig, drill).
:::

To begin with, I will need to delegate the zone from the `unbound` server to the `nsd` server
To do this, I need to write the following contents at the end of the `unbound` configuration file:

```
...
stub-zone:
  name: "st11.sne21.ru"
  stub-addr: 127.0.0.1@5358 # address and port of the nzd host and its port

```


Next, we need to allow support `st11.sne21.ru` in the `nsd` configuration

```
...
zone:
    name: "st11.sne21.ru"
    zonefile: "st11.sne21.zone"

```

and create a file for this zone

`sudo touch /usr/local/etc/nsd/zones/st11.sne21.zone`

Сontents of the zone file:

```
$ORIGIN st11.sne21.ru.
$TTL 1800
@       IN      SOA     ns.st11.sne21.ru. st11.sne21.ru.  (
                        2014070201              ; serial number
                        3600                    ; refresh
                        900                     ; retry
                        1209600                 ; expire
                        1800                    ; ttl
                       )
                   IN      NS      st11.sne21.ru.
                   IN      NS      ns.st11.sne21.ru.

st12               IN      A       10.1.1.212 ;st12.st11.sne21.ru -> 10.1.1.212
cname              IN      CNAME   st11       ;cname.st11.sne21.ru -> 10.1.1.212
cname2             IN      CNAME   st11
mx1                IN      MX      10      cname 
mx2                IN      MX      20      cname2

host1              IN      A       10.1.1.211
host2              IN      A       10.1.1.211
host3              IN      A       10.1.1.211
@                  IN      A       10.1.1.211

```

It remains only to restart `unbound` and `nsd`

```
sudo unbound-control reload
sudo nsd-control reload
```


We check that everything works

<center>


![](https://i.imgur.com/7oBvtul.png)
Figure 8: Output of information about the ns zone for my subdomain

![](https://i.imgur.com/etg8aeQ.png)
Figure 9: Checking the MX record for a subdomain

![](https://i.imgur.com/y0zi8FY.png)
Figure 10: Checking the CNAME record for a subdomain
</center>



It would also be possible to register local zones in the unbound config

```
server:
    local-zone: "st11.sne21.ru." transparent
    local-data: "st11.sne21.ru. IN NS ns.st11.sne.ru"

    local-zone: "ns.st11.sne21.ru." static
    local-data: "ns.st11.sne21.ru. IN A 192.168.122.92"
    local-data: "ns.st11.sne21.ru. IN MX 10 ns.st11.sne21.ru"
    local-data: "ns.st11.sne21.ru IN CNAME 192.168.122.92"
    local-data: "ns.st11.sne21.ru. IN NS st11.sne21.ru"
```


## BONUS - Task 5 - Delegating Your Own Zone (2 points)

:::info
Work together with one of your fellow students. Each of you will create a subdomain of
your own domain and delegate the authority for that subdomain to your partner. So in the
end there should be two additional zones:
- st<X>.st<Y>.sne21.ru and
- st<Y>.st<X>.sne21.ru
where X - the number of your machine, Y - the number of your partner’s machine.
How did you set up the subdomains and their delegation:
➢ How did you update st<X>.sne21.ru zone file to delegate subdomain to
your partner?
➢ Show content of st<X>.st<Y>.sne21.ru zone file that you have
delegation for.
➢ What named.conf/nsd.conf options did you add or change?
➢ Show the results of the tests that you performed.
:::

## Implementation

:::info
> How did you update st<X>.sne21.ru zone file to delegate subdomain to
your partner?

To transfer a subdomain, I added an entry of the form to my zone

```
st12      NS        ns2
ns2       A         10.1.1.212
```

:::

:::info
> Show content of st<X>.st<Y>.sne21.ru zone file that you have
delegation for.

```                                               
$ORIGIN st9.st11.sne21.ru.
$TTL 1800

@       IN      SOA     ns1.st9.st11.sne21.ru.  st9.st11.sne21.ru. (
                        2014070201              ; serial number
                        3600                    ; refresh
                        900                     ; retry
                        1209600                 ; expire
                        1800                    ; ttl
)

IN      NS      st9.st11.sne21.ru.

test    IN      A       10.1.1.209

@       IN      A       10.1.1.209

```
:::

:::info
> What named.conf/nsd.conf options did you add or change?

I need to add support for partner domain names
```
stub-zone:
  name: "st9.sne21.ru"
  stub-addr: 10.1.1.209
```

:::

:::info
➢ Show the results of the tests that you performed.
![](https://i.imgur.com/0gyylrl.png)

![](https://i.imgur.com/UC6BKXZ.png)


:::

## References:
1. [Next start to unbound](https://www.nlnetlabs.nl/documentation/unbound/howto-setup/)
2. [Setting up Unbound from scratch](https://pub.nethence.com/daemons/unbound)
3. [Configuring Unbound as a simple forwarding DNS server](https://www.redhat.com/sysadmin/forwarding-dns-2)
4. [Unbound not accepting a stub or forward pointing to a loopback interface.](https://nsd-users.nlnetlabs.narkive.com/SVTmRQKB/unbound-not-accepting-a-stub-or-forward-pointing-to-a-loopback-interface)
5. [Unbound DNS Server Cache Control](https://abridge2devnull.com/posts/2016/03/unbound-dns-server-cache-control/)




