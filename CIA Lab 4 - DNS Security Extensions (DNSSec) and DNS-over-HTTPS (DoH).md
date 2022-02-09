---
tags: Classical Internet Applications
---
:::success
# CIA Lab 4 - DNS Security Extensions (DNSSec) and DNS-over-HTTPS (DoH)
Name: Ivan Okhotnikov
:::


## Task 1 - DNS Insecurity
:::warning
● In what context was the DNS cache poisoning attack discovered? (Who, when, why)
● How does this attack work and how to prevent it?
● What is DNS spoofing and what tools can be used to integrate this attack?
● What does a validating resolver do? What is the difference between island-of-trust versus full-chain-of-trust?
● Does DoH substitute DNSSec or complement it? What are the differences between them?
● What do you think about DoH vs DoT vs DNSCrypt?
:::

## Implementation:

:::info
> In what context was the DNS cache poisoning attack discovered? (Who, when, why)


The DNS appeared at the dawn of the Internet and since it was mainly used by universities, there was no point in thinking about security.

The attack was accidentally discovered in 2008 by Dan Kaminsky at the time of research on the fastest data placement on the Internet
Kaminsky sent a different response to different hosts at the same time (to identify them).
While using random domain names, he noticed that they are overwritten in the DNS cache (although they should not be at all)

:::

:::info
> How does this attack work and how to prevent it?

The attacker will send the `wrong IP` to the client (you need to do this quickly so that the response from the real DNS resolver does not have time to reach it) and he will cache the response at the time specified in the `TTL`.
For substitution, the `TXID` is required (the attacker will have to pick it up in a short time, which makes the attack somewhat more difficult)

Basically, the algorithm of the `birthday paradox` is used for selection, which is based on the theory of verification and makes it easier to find the desired identifier)

**I can prevent the attack by:**
- implementing verification and authentication (DNSSEC)
- use of verified dns servers

**I can also use additional methods:**
- using a VPN connection (if I work from home)
- changing the WiFi password to a more complex one (if I work from home)

The very first solution to combat the attack after its detection in `2008` was to increase the `TXID` to 32.
This solution prevented instant attacks, but does not eliminate the attack itself.

:::

:::info
> What is DNS spoofing and what tools can be used to integrate this attack?

This is the same as DNS cache poisoning, but only in a more general sense.
This attack allows the user to get the wrong ip address for the domain

You can perform this attack by using some utilities:

- **arpspoof**
>The program allows you to redirect packets from the target host to the LAN that are intended for another host by faking ARP responses
For example: when a user connects, he searches for a host. At this point, we can substitute the answer and it will connect to us
**Basic parameters:**
`-i` indicates the network interface used
`-t <host>?` indicates the target to which we will send packets (if not specified, all will be selected)

- **dnsspoof**
>The program allows you to fake responses from a DNS server in a local network and allows you to implement "attacker-in-the-middle" attacks 
**Basic parameters:**
`-i` indicates the network interface used
`-f` stands for host file

:::

:::info
> What does a validating resolver do? What is the difference between island-of-trust versus full-chain-of-trust?

Its function is that repeated requests are sent via DNSSEC, and the response is checked in order to verify the sender

For this purpose, encryption and two types of keys are used:
**Private** - it is used to encrypt the signature
**Public** - it is available to all users and is used to decrypt and verify the signature

---

**island-of-trust:**
> Comparison of the DNSKEY hash (private key), which is stored in the DS record with the public key.
If they match, then the zone can be trusted

**full-chain-of-trust:**
> A chain of interconnected trusted zones (which are checked using the methods described above)
It also includes trust in the root zone (the security of which is strict enough that we trust it unconditionally)

:::

:::info
> Does DoH substitute DNSSec or complement it? What are the differences between them?

DoH complements DNSSec, since DoH (DNS over HTTPS) and DNSSec were developed at different times, and each of them has its own documentation

The main goal of DoH is to increase user privacy and ensure confidentiality. This is achieved by the fact that all transmitted traffic is masked in the general HTTPS traffic

The main purpose of DNSSec is to confirm the authenticity of a DNS record.

**Difference:**

- HTTPS works via TLS, and DNSSec works via UDP (however, if configured correctly, DNSSec can also work via TLS)
- They are aimed at different aspects of security (since the methods do not conflict, they can be used together)

An example of DNSSec and DoH working together is the implementation from CloudFlare (ODoH)
:::

:::info
> What do you think about DoH vs DoT vs DNSCrypt?

`DoH` provides greater confidentiality than `DoT` because it is hidden in the general `HTTPS` traffic (used at the application level). Implements full encryption of the `DNS protocol`. New versions are being developed. Uses port `443`.

`DoT`, in turn, is more preferred by system administrators for simple tracking and blocking `DNS traffic` (used at the transport level). Uses port `853`. The initial connection is slow because the handshake is long

`DNSCrypt` is a protocol for authenticating and encrypting traffic (It is better than DoH in terms of confidentiality)
It can work with `DoH` on a single port. Allows you to hide the IP addresses of clients from the servers. 

Because of the greater privacy, `DNSCrypt` is more preferable to me

:::

## Task 2 - Validating Resolver

:::warning
> Setup
● Enable DNSSec to your Unbound and verify the root key is used as a trusted source.
● Use dig or drill to verify the validity of DNS records for isc.org, sne21.ru 


> Validate
● How does dig or drill show whether DNSSec full-chain-of-trust validation was
successful or not? Hint: it’s about a flag

> Counter-validate (breaking the chain of trust)
● Where does Unbound store the DNSSec root key?
● How do managed keys differ from trusted keys? Which RFC describes the mechanisms for managed keys?
● Modify the DNSSec root key and test your validating resolver.
● How did your server react?
:::

## Imlementation

### Setup

:::info
> Enable DNSSec to your Unbound and verify the root key is used as a trusted source.

To enable DNSSec on unbound, I need to generate a root key file `root.key`

To start, I will go to the unbound directory
`cd /usr/local/etc/unbound`

And from under the unbound user (since otherwise there may be problems with access rights), we will create root. key c using the unbound-anchor command
```
sudo chown -R unbound:unbound /usr/local/etc/unbound/
sudo -u unbound unbound-anchor 
```

Then I need to fix the unbound.conf configuration file by adding `root.key` support to it

```
auto-trust-anchor-file: "root.key"
```
<center>

![](https://i.imgur.com/pPjPphL.png)
Figure 1: Configuration example
</center>
:::

:::info
> Use dig or drill to verify the validity of DNS records for isc.org, sne21.ru

Using the `dig command x + dnssec`, where x is the domain name I can determine that DNS resolver uses dnssec (as in the figures below)

<center>

![](https://i.imgur.com/qLjuncI.png)
Figure 2: Without using DNSSEC part 1
![](https://i.imgur.com/y2bGtpC.png)
Figure 3: Without using DNSSEC part 2
    
</center>

---

<center>

![](https://i.imgur.com/9HSya7T.png)
Figure 4: Using DNSSEC part 1
![](https://i.imgur.com/w27JgMM.png)
Figure 5: Using DNSSEC part 2
</center>
:::

### Validate

:::info
> How does dig or drill show whether DNSSec full-chain-of-trust validation was
successful or not? Hint: it’s about a flag

The `-ad` flag is responsible for the fact that our dns resolver uses dnssec (as shown in the figures above)
:::

### Counter-validate (breaking the chain of trust)

:::info
> Where does Unbound store the DNSSec root key?

In my case, in the directory `/usr/local/etc/unbound`,
but you can go to any other dirrectory. It is important that the unbound user has rights and knows where the root. key is located (you need to specify this in the configuration file)

:::

:::info
> How do managed keys differ from trusted keys? Which RFC describes the mechanisms for managed keys?

- `trusted-keys` are always trusted until they are removed from named. conf;
- `managed-keys` are trusted only once. It implements the loading of managed keys and starts the key maintenance process.

To support managed keys, use `RFC5011`

:::

:::info
> Modify the DNSSec root key and test your validating resolver.

I changed the key and then restarted the server
An entry appeared in the logs

```
[1631284565] unbound[17935:0] info: failed to prime trust anchor -- DNSKEY rrset is not secure . DNSKEY IN
[1631284565] unbound[17935:0] info: validate keys with anchor(DS): sec_status_bogus
```
:::

:::info
> How did your server react?
> 
Unbound, although running, does not work as a dns resolver
The successful validation flag has disappeared and dnssec also does not work

:::

## Task 3 - Secure Zone

:::warning
> Island of trust
● Look up which cryptographic algorithms are available for use in DNSSec. Which one do you prefer, and why?
● What is the difference between Key-Signing Key (KSK) and Zone-Signing Key (ZSK)?
● Why are those separated and why do those use different algorithms, key sizes and lifetimes?
● Use NSD tools (ldns−keygen & ldns−signzone) to secure your zone. Show the configuration files involved and validate your setup (https://dnssec-analyzer.verisignlabs.com, https://dnsviz.net).

> Full chain of trust
● Does your parent zone offer secure delegation?
● Describe the DS and DNSKEY records from sne21.ru that are important for your domain. Which keys are used to sign them?
:::

## Imlementation
### Island of trust
:::info
> Look up which cryptographic algorithms are available for use in DNSSec. Which one do you prefer, and why?

Based on RFC8624, it is possible to use various encryption algorithms:
- RSAMD5
- DSA
- RSASHA1
- DSA-NSEC3-SHA1
- RSASHA1-NSEC3-SHA1
- RSASHA256
- RSASHA512
- ECCGOST
- ECDSAP256SHA256
- ECDSAP384SHA384
- ED25519
- ED448

Each of them provides its own level of security.
Cryptographically weak algorithms in the signature can cause serious problems.
Due to the low level of security, it is better to abandon the use of algorithms such as `RSAMD5` and `DSA-NSEC3-SHA1`
In my opinion, it is better to use new algorithms like `ED25519` and `ED448` (since they are cryptographically more resistant and help to maintain security at a high level

:::

:::info
> What is the difference between Key-Signing Key (KSK) and Zone-Signing Key (ZSK)?

**ZSK** - `zone-signing keys`.
Each zone has a pair of keys: `public` and `private`. The private part signs each RRSET in the zone, and the public part checks the signature
In order for DNS resolver to understand that it is necessary to check the record, it is necessary to make the ZSK public key available


**KSK** - `key-signing key`.
It is larger in size than ZSK because it is used to encrypt small data. Since it is larger than ZSK, its lifetime is longer and changes less often.
It is a key for signing `DNSKEY ZSK`.

:::

:::info
> Why are those separated and why do those use different algorithms, key sizes and lifetimes?

Comparing `ZSK` and `KSK` in size, `ZSK` will be smaller. This gives great performance. But there is also a downside. The smaller the key, the less time it takes to crack it. Therefore, `ZSK` needs to be changed more often.

To change the `KSK`, you will need to interact with the parent zone. This may be more difficult than deploying `ZSK`. Also, since its size is larger and used on smaller data, its lifetime is longer and it needs to be updated less often

Depending on the deployment, they may have different encryption (the list is specified in RFC6840)

:::

:::info
> Use NSD tools (ldns−keygen & ldns−signzone) to secure your zone. Show the configuration files involved and validate your setup.

The first thing you need to do is install `ldnsutils`
`sudo apt-get install ldnsutils`

To generate the key, use the ldns-keygen command
I will do this for the `ZSK` and `KSK` keys

```
export ZSK=`ldns-keygen -a RSASHA256 -b 1024 st11.sne21.ru -r /dev/urandom`
export KSK=`ldns-keygen -k -a RSASHA256 -b 2048 st11.sne21.ru -r /dev/urandom`
```

>`$ZSK` and `$KSK` contains the response of the `ldns-keygen` program, which points to the created keys (in my case, `KSK` and `ZSK`)

`-a` denotes the encryption algorithm (I have `RSASHA256`)
`-b` is the key length
`-r` is a random number generator (I changed it because I had problems with it on AWS)

After generating the keys, I signed the zones using the `ldns-signzone` command

`ldns-signzone st11.sne21.zone $ZSK $KSK`

After successful signing, we change the name of the zone file in the configuration file (since the signed zone was created)

`sudo nano /usr/local/etc/nsd/nsd.conf`

<center>

![](https://i.imgur.com/wpzp2wS.png)
Figure 6: Part from the NSD configuration file
</center>

After the manipulations, I made sure that nothing was broken and rebooted the `nsd`

`nsd-checkconf /usr/local/etc/nsd/nsd.conf`

`nsd-control reload`

<center>

![](https://i.imgur.com/dQ5Okbz.png)
Figure 7: Response from the dig command (show DNSKEY record for my local domain)
</center>

:::

### Full chain of trust
:::info
> Does your parent zone offer secure delegation?

Yes offers
Since to build this zone, I passed its DS record and public IP address to the parent zone
:::

:::info
> Describe the DS and DNSKEY records from sne21.ru that are important for your domain. Which keys are used to sign them?

DS key was formed from DNSKEY KSK
I used the ZSK and KSK keys to sign the record

<center>

![](https://i.imgur.com/NyZgBdf.png)
Figure 8: Response from the dig command (show DNSKEY record)
</center>

<center>

![](https://i.imgur.com/7gTbQuf.png)
Figure 9: response from the dig command (show DS record)
</center>

:::

## Task 4 - Key Rollovers

:::warning
> **Zone-Signing Key rollover**
● Study RFC 6781 or affiliated online resources
● What are the options for doing a ZSK rollover? Choose one procedure and motivate your choice
● How would you integrate this procedure with the tools for signing your zone?
Which timers are important?
● Bonus (1 point): integrate the procedure and use a DNSSec debugger to verify each step

> **Key-Signing Key rollover**
● Can you use the same procedure for a KSK rollover?
● What does this depend on?
:::

## Imlementation

### Zone-Signing Key rollover

:::info
> What are the options for doing a ZSK rollover? Choose one procedure and motivate your choice

- Pre-Publish key rollover
>It does not require a double signature of data inside the zone. This is achieved by the fact that during the transfer, a new key is published in the key set, which makes it possible to perform an attack using cryptanalysis
It requires four stages

- Double-Signature ZSK rollover
>During the work, the number of signatures will double
If there are many large zones, this may be prohibitive
It requires three stages

:::

:::info
> How would you integrate this procedure with the tools for signing your zone?
Which timers are important?

I would do it like this:
- A new key pair was generated using a single algorithm (in my case, RSASHA256)
- New public keys I will add are added to the zone file for pre-publication. The zone is signed by the active ZSK and KSK and published
- When the ZSK period comes to an end, I will sign the zone with a new ZSK key and an active KSK

LDNS does not allow you to enter timers, but I can solve the issue of setting the working time in two ways:

- Use the minimum TTL for for writing
- Specify the minimum prolongation interval for the SOA record


However, the second option is more preferable, since it allows you to minimize the risks

The TTL time is set in seconds, so it is important to convert the value to seconds in advance and use them

:::

:::info
> Bonus (1 point): integrate the procedure and use a DNSSec debugger to verify each step

:::

### Key-Signing Key rollover

:::info
> Can you use the same procedure for a KSK rollover?
> What does this depend on?
    
Yes, theoretically I can, but do not forget that these rollovers are different.
To update the KSK, you need to interact with the root zone.
This depends on the fact that the zone data and public keys are cached independently by DNS resolvers.
Because of this, I can get an error when checking DNSSEC if a converter that does not already have the old key will try to check records signed with the old key. Well, or vice versa.

:::


## Task 5 - DNS-over-HTTPS

:::warning
Implementing DNS-over-HTTPS on your server.
● For this step you probably will need to recompile and reinstall your server (unbound requires --with-libnghttp2 flag).
● Also, you will need a certificate for your domain, you can use certbot to obtain it.
● Show configuration options that have to be tuned.
● Verify that DNS-over-HTTPS is working (Chrome and Firefox has support for it) by inspecting server incoming queries in the log.
:::

## Imlementation

### Implementing DNS-over-HTTPS on your server.

:::info
> For this step you probably will need to recompile and reinstall your server (unbound requires `--with-libnghttp2` flag).

Before starting the reconfiguration, I need to install the `libnghttp2` library

`sudo apt-get install libnghttp2-dev`

Next, I need to reconfigure and install `unbound`

```
sudo ./configure --prefix=/usr/local \
--sysconfdir=/usr/local/etc \
--with-pidfile=/var/run/unbound.pid \
--with-libnghttp2

make
sudo make install
```
:::

:::info
> Also, you will need a certificate for your domain, you can use certbot to obtain it.

To get a certificate, I will first install certbot
`sudo apt-get install certbot`

Then I will use the command below to get a certificate for my domain:

`certbot -d st11.sne21.ru`

<center>

![](https://i.imgur.com/6RnthIt.png)
Figure 10: The window for successfully creating the certbot certificate
</center>

After receiving the certificate, I installed NGINX and added support for my subdomain and certbot certificate files to its configuration

`sudo nano /etc/nginx/sites-available/default`

<center>

![](https://i.imgur.com/PtIywHK.png)
Figure 11: Part of the nginx configuration file
</center>

:::

:::info
> Show configuration options that have to be tuned.

I allowed `setrd` in `access_control` for unbound
You can use it to disable recursion and then everything will work

<center>

![](https://i.imgur.com/7yAqpzA.png)
Figure 12: Part of the unbound configuration (now access_control allows setrd)
</center>

:::

:::info
> Verify that DNS-over-HTTPS is working (Chrome and Firefox has support for it) by inspecting server incoming queries in the log.

Opening the page https://st11.sne21.ru in the browser, I can make sure to the left of the address bar that the certificate was installed correctly
<center>

![](https://i.imgur.com/SndbpR6.png)
Figure 13: Successful result of checking the health of the ssl certificate from Let's Encrypt
</center>
:::


## Bonus - Secure Zone Delegation (2 points)
:::warning
● Securely delegate a subzone to another student - st<X>.st<Y>.sne21.ru,
where X - the number of your machine, Y - the number of your partner’s machine.
● How to communicate the necessary record to your buddy? (Integrity matters here)
● What about Confidentiality, does it matter here?
:::

## Implementation:

:::info
> Securely delegate a subzone to another student - st<X>.st<Y>.sne21.ru,
where X - the number of your machine, Y - the number of your partner’s machine.

To enable this, it was necessary to add I need to create a zone `st11.st8.sn21.zone` with the following contents:

```
$TTL 1800
$ORIGIN st11.st8.sne21.ru

@ IN SOA ns ns5 (
   20 ;Serial
   12h ;Refresh
   15m ;Retry
   3w ;Expire
   2h ;Minimum
 )

@ IN A 3.19.79.92

@ IN NS ns5
ns IN A 3.19.79.92
```

After creating a zone, you need to sign it and include it in the `NSD`

<center>

![](https://i.imgur.com/mTV1BDu.png)
Figure 14: Added lines to support the zone file
</center>


And at a friend's place, make notes in the main zone:

```
st11              IN      NS      ns.st11
ns.st11           IN      A       3.19.79.92
st11              IN      DS      ...
```

where DS is my transferred public key KSK (with which I signed the zone)

---

<center>

![](https://i.imgur.com/E9hY6HY.png)
Figure 15: Result of successful delegation of my subdomain
</center>

**Unfortunately, unbound refused to work in this mode and I turned it off, after that I turned on nsd on port 53 and everything worked**

:::

:::info
> How to communicate the necessary record to your buddy? (Integrity matters here)

I have to create entries for it in my master record:

```
st8                IN      NS      ns.st8
ns.st8             IN      A       3.131.159.226
st8                IN      DS      .....
```
where DS is the KSK public key transmitted by a friend (with which he signed the zone)

Since it works with bind, its zone file looks like this:
```
$TTL 1800
$ORIGIN st8.st11.sne21.ru.

@ IN SOA ns admin (
   20 ;Serial
   12h ;Refresh
   15m ;Retry
   3w ;Expire
   2h ;Minimum
 )

@ IN A 3.131.159.226

@ IN NS ns
ns IN a 3.131.159.226

@ IN MX 10 mail
mail IN A 3.131.159.226

www IN A 3.131.159.226


$INCLUDE "/etc/bind/keys/Kst8.st11.sne21.ru.+008+38152.key" ;KSK
$INCLUDE "/etc/bind/keys/Kst8.st11.sne21.ru.+008+56853.key" ;ZSK
```

:::

:::info
> What about Confidentiality, does it matter here?

Since DNSSEC was used, there is no need to talk about confidentiality
:::


## Bonus - NSEC vs NSEC3 (1 point)

:::warning
NSEC is used to prove that a certain resource record does not exist. A nasty side effect of
NSEC is that it enables zone traversal.
Using NSEC3, a DNS server can prove the non-existence of a resource record without
facilitating zone traversal.
● What are the main differences between NSEC and NSEC3?
● How to select one or the other during zone-signing?
● Do the root-servers use NSEC or NSEC3? What about the .org domain?
● How to take advantage of zone traversal as an attacker, how does it work?
:::
## Implementation:

:::info
> What are the main differences between NSEC and NSEC3?

`NSEC3` is an improved version of `NSEC`.
They are necessary to create an authenticated denial of the existence of records.

**For NSEC:**
For each record, an NSEC-signed record corresponds. This record has a cryptographic signature and allows you to create a list of all records in the zone.
Thus, if I request a non-existent record, the DNS server will be able to answer with full confidence that it does not exist. 

**For NSEC3:**
NSEC3 solves the problem of using NSEC responses to create a list of all records in the zone.
The solution is to add a caching mechanism. Because of this, the zone cannot be traversed as easily as in NSEC

**To summarize:**
NSEC is easier and I should use it if it's all the same for domain scanners.

If it is important to me that users cannot move freely throughout the entire zone, then `NSEC3` should be used
:::

:::info
> How to select one or the other during zone-signing?

In my case, when I use dns-signzone, the choice between `NSEC3` and `NSEC` is made by specifying the `-n` flag
For example:
`ldns-signzone -n ...` will use NSEC3
:::

:::info
> Do the root-servers use NSEC or NSEC3? What about the .org domain?

The check for the presence of `NSEC` in the root zone can be checked
with the command:
`dig . NSEC +dnssec`
The output result indicates that `NSEC` is used

In the case of the zone .I will perform the same check for the zone `.org`
`NSEC3` is used for it

<center>

![](https://i.imgur.com/5dTe4NO.png)
Figure 16: Result of checking the root zone

![](https://i.imgur.com/B7tSKe9.png)
Figure 17: The result of checking the zone `.org`
</center>

:::

:::info
> How to take advantage of zone traversal as an attacker, how does it work?

To use the zone bypass, you will need to get the NSEC content, which contains the following domain name in the zone. After going through this entire chain, you will get a list of all available records

:::

## References:
1. [DNS cache poisoning (CloudFlare)](https://www.cloudflare.com/learning/dns/dns-cache-poisoning/)
2. [DNS cache poisoning (Varonis)](https://www.varonis.com/blog/dns-cache-poisoning/)
3. [What are DNS Spoofing, DNS Hijacking, and DNS Cache Poisoning?](https://www.infoblox.com/dns-security-resource-center/what-are-dns-spoofing-dns-hijacking-dns-cache-poisoning/)
3. [DNS cache poisoning. Description of the technology, examples and method of protection](https://www.securitylab.ru/analytics/499199.php)
3. [Validating Resolver](https://clouddocs.f5.com/training/community/dns/html/class2/module5/module5.html)
3. [DNS over TLS vs. DNS over HTTPS | Secure DNS](https://www.cloudflare.com/learning/dns/dns-over-tls/)
1. [Frequently Asked Questions DNSCrypt](https://dnscrypt.info/faq/)
1. [Howto enable DNSSEC](https://nlnetlabs.nl/documentation/unbound/howto-anchor/)
1. [DNSSEC Guide : Chapter 3. Validation](https://dnsinstitute.com/documentation/dnssec-guide/ch03s04.html)
1. [DNSSEC:NSEC vs. NSEC3](https://www.internetsociety.org/resources/deploy360/2014/dnssecnsec-vs-nsec3/)
1. [DNSSEC veelgestelde vragen (FAQ)](https://archief.dnssec.nl/wat-is-dnssec/faq/index.html)
1. [Dnssec howto with NSD and ldns](https://www.whyscream.net/wiki/Dnssec_howto_with_NSD_and_ldns.md)
1. [How To Set Up DNSSEC on an NSD Nameserver on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-dnssec-on-an-nsd-nameserver-on-ubuntu-14-04)