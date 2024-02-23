# What you need to know about DNS

    Letitia!
    What a name.
    Halfway between a salad and a sneeze.
    -- Terry Pratchett


## The DNS protocol
DNS, or the Domain Name Service, is an indispensable element of modern networks. Imagine a world where you had to recall IP addresses like 88.190.237.114 or 2a01:e0b:1000:23:be30:5bff:fed7:9af instead of domain names. DNS simplifies our digital lives by translating human-readable domain names into machine-readable IP addresses.

However, DNS isn't just about name resolution. It plays a crucial role in the functioning of SMTP networks. Understanding DNS is essential for grasping SMTP mechanics.

In fact, many SMTP failures stem from issues at the DNS level. A faulty DNS setup inevitably leads to disrupted SMTP exchanges.

Let's delve into how DNS operates and its interaction with SMTP, focusing solely on the basics for now. We'll save advanced topics like DKIM, SPF, DMARC, and DANE for later discussions. Additionally, for simplicity's sake, we'll ignore IPv6 for the time being.


### DNS zones
DNS servers organize domain information into zones. Each DNS server manages one or more zones, containing records pertinent to the associated domains. For example, the zone file for "opensmtpd.org" may include details like:

DNS servers organize domain information into zones.
Each DNS server manages one or more zones, containing records pertinent to the associated domains.
For example,
the _ns-master.poolp.org_ nameserver is in charge of the domain _opensmtpd.org_ and has a zone file which includes details like:

```
                NS      ns-master.poolp.org.
                NS      ns-backup.poolp.org.
                A       82.65.169.200
www             A       82.65.169.200
opensmtpd.org.  MX 0    mx-in.poolp.org.
opensmtpd.org.  MX 50   mx-backup.poolp.org.
```

The opensmtpd.org zone is configured with two nameservers: ns-master.poolp.org and ns-backup.poolp.org. Additionally, it specifies that both opensmtpd.org and www.opensmtpd.org resolve to the IP address 82.65.169.200. Of particular interest to us postmasters, it includes two mail exchanger records for opensmtpd.org: mx-in.poolp.org with a preference of 0 and mx-backup.poolp.org with a preference of 50. These mail exchanger records are managed using MX records.

It's worth noting that records can have either relative labels, such as www relative to the zone domain opensmtpd.org, or absolute labels, indicated by a trailing dot.


### MX records
MX records play a crucial role in designating mail exchangers responsible for handling emails for a specific domain or subdomain. While a domain must have at least one MX record (to receive emails), it can list multiple for redundancy. For example:

```
opensmtpd.org.          MX 0    mx-in.poolp.org.
opensmtpd.org.          MX 10   mx-backup.poolp.org.
subnet1.opensmtpd.org.  MX 0    mx1.example.org.
subnet2.opensmtpd.org.  MX 0    mx1.example.com.
```

Here, "opensmtpd.org" has two mail exchangers, while subdomains like "subnet1.opensmtpd.org" and "subnet2.opensmtpd.org" have their own designated mail exchangers.


### Resolver
XXX


### MX lookup
SMTP servers make use of DNS for many purposes,
like looking up the hostname matching the IP address of a client for example,
but the most important use upon which the entire SMTP protocol is built is _MX lookup_.
If you read this far,
you probably understood that this boils down to finding which MX are responsible for the destination domain of a message.

Imagine I'm exchanging emails with Eric Faurot. I, with the email address gilles@poolp.org, want to send an email to Eric Faurot at eric@faurot.net. I input his email address into my Mail User Agent (MUA), such as mutt, and send the message.

Behind the scenes, my MUA somehow forwards the message to a configured SMTP server. The SMTP server agrees to handle the email, queues it for relaying, and provides my MUA with a transaction identifier indicating the message's status as OK. Since this is a standard scenario, my MUA doesn't provide any additional feedback, following the Unix philosophy of staying quiet unless there's an issue.

Now, the magic unfolds on the server's side: my SMTP server conducts an MX lookup. It instructs its configured nameserver to retrieve the MX records for the domain faurot.net. The nameserver either retrieves this information from its cache or locates the authoritative nameservers for faurot.net:

```
$ dig +nocomments -t NS faurot.net |grep NS 
; <<>> DiG 9.4.2-P2 <<>> +nocomments -t NS faurot.net
;faurot.net.                    IN      NS
faurot.net.             10757   IN      NS      c.dns.gandi.net.
faurot.net.             10757   IN      NS      a.dns.gandi.net.
faurot.net.             10757   IN      NS      b.dns.gandi.net.
$
```

Then requests either on of them to return the MX entries for the domain faurot.net:

```
$ dig +nocomments -t MX faurot.net |grep MX 
; <<>> DiG 9.4.2-P2 <<>> +nocomments -t MX faurot.net
;faurot.net.                    IN      MX
faurot.net.             10496   IN      MX      50 fb.mail.gandi.net.
faurot.net.             10496   IN      MX      10 spool.mail.gandi.net.
$ 
```

Upon obtaining this information, my resolver returns the MX records for faurot.net to my SMTP server. For instance, it may provide fb.mail.gandi.net with a preference of 50 and spool.mail.gandi.net with a preference of 10. My SMTP server prioritizes these mail exchangers based on their preference numbers, placing those with lower values at the top:

```
faurot.net.             10496   IN      MX      10 spool.mail.gandi.net.
faurot.net.             10496   IN      MX      50 fb.mail.gandi.net.
```

Next, my SMTP server conducts a DNS A lookup on the first mail exchanger to translate the hostname to an IP address:

```
$ dig +nocomments -t A spool.mail.gandi.net | grep A 
; <<>> dig 9.10.8-P1 <<>> +nocomments -t A spool.mail.gandi.net
;spool.mail.gandi.net.          IN      A
spool.mail.gandi.net.   487     IN      A       217.70.178.1
$  
```

It then attempts to establish a connection to that IP to start an SMTP session and deliver the message in an SMTP transaction.

Typically, this marks the conclusion of my SMTP server's involvement. It's fulfilled its responsibility by delivering the message to its intended destination.


### Backup MX
Due to the quantic nature of our universe,
an MX may be in two overlapping states:
it may be willing to accept a message but it may also be unable to accept it at the same time.

The MX may have a dying drive failing disk writes at random,
it may be lacking disk space all of a sudden due to a surge of messages with large attachements,
it may even work strangely due to a broken DNS setup<sup>[1](#1)</sup>.
A variety of reasons can prevent it from committing to deliver a message.

That's if its even reachable at all.

The MX could suffer a power outage,
a network outage,
someone tripping on the cable,
or even an exhausted sysadmin accidentally typing `halt` in the wrong terminal<sup>[2](#2)</sup>.

This is where preferences come into play.
Each MX server for a domain has a preference value associated to its MX record in the zone.
The preference is a numerical value which determines in what order MX servers should be attempted by a remote MX when trying to transfer messages for the domain.
The lower the preference value is, the highest priority the MX gets.
This mechanism can be use to distribute load over pools of machines,
to prioritize faster servers,
or to create... backup MX.

```
faurot.net.   10496   IN      MX      10 spool.mail.gandi.net.
faurot.net.   10496   IN      MX      50 fb.mail.gandi.net.
```

From my MX point of view,
there is no difference between these two MX servers except for the requirement to attempt one before the other.
It considers _spool.mail.gandi.net_ as closer to the destination than _fb.mail.gandi.net_,
though from a purely technical perspective it could send to either one and be discharged of its responsability with regard to the mail as they both are declared as destination MX for the domain in the zone.

Any of the MX for a domain may accept messages for that domain,
regardless of their preference value,
but since an MX commits to bring a message closer to its destination,
it implies that an MX that's not the destination will only attempt to forward a message to another MX with a lower preference value.
These MX servers are called _backup MX_ because they provide a backup service to the _primary MX_ servers in case they are unreachable,
and commit to either attempt passing them the messages until they succeed or acknowledge the original sender that an error occurred.

Note that the preference value is only here to create a priority-sorted list of MX servers to contact.
There is no requirement that a priority 10 exists,
there is no rule preventing a zone from using preferences ranging from 100 to 200 with 100 being its _primary MX_.
Basically,
nothing prevent postmasters to give the preference they want to the MX they want and the sender MX should not think in terms of primary or backup MX,
but in terms of what's the MX with the lowest value that it has not tried yet.

Let's get back to my mail exchange and pretend that _spool.mail.gandi.net_ is temporarily unavailable.
My MX fails to connect and looks at the next entry in the list which is _fb.mail.gandi.net_.
It resolves the name to an address,
then attempts to connect and passe over the message.

At this point there are three possible outcomes:
- I'm being unlucky, the backup MX is also temporarily unable to handle the mail.
    Since the zone does not provide a third MX to contact,
    my MX is stuck with the mail and will have to reattempt a transfer later.

- The MX has explicitely rejected the mail permanently.
  There is no need to try again later in hope that another MX would accept it,
  a backup MX is not allowed to reject a message that could be accepted by a primary MX,
  so all destination MX can be considered as authoritative in their rejections.

- The MX has accepted the message.
  As far as my MX is concerned,
  another MX is now responsible for not letting the message vanish,
  work is over.


<hr />

[<sup>1</sup>](#1) By now you know this is the first thing to rule out.

[<sup>2</sup>](#2) This is a work of fiction. Names, characters, businesses, places, events, locales, and incidents are either the products of the author's imagination or used in a fictitious manner. Any resemblance to actual persons, living or dead, or actual events is purely coincidental.
