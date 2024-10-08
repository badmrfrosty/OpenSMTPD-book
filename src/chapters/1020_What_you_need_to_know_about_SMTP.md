# What you need to know about SMTP

    Risk comes from not knowing what you're doing.
    -- Warren Buffett

    We are all born ignorant, but one must work hard to remain stupid.
    -- Benjamin Franklin


## What is SMTP ?
SMTP is the Simple Mail Transfer Protocol. It's a protocol standardized in RFC5321 and used by mail systems to communicate with each other. The SMTP protocol doesn't manage the entire email process but has a very specific goal: reliably transferring mail from point A to point B. It's like the electronic version of a mailman, more reliable though...

## The SMTP protocol
SMTP operates as a plaintext protocol, facilitating a line-based, human-readable dialogue between a client and a server. Even a human, including myself, can manually engage in an SMTP session by connecting to an SMTP server:

```
$ nc localhost 25 
S: 220 poolp.org ESMTP OpenSMTPD
C: HELO localhost
S: 250 poolp.org Hello localhost [127.0.0.1], pleased to meet you
C: MAIL FROM:<gilles>
S: 250 2.0.0: Ok
C: RCPT TO:<gilles>
S: 250 2.1.5 Destination address valid: Recipient ok
C: DATA
S: 354 Enter mail, end with "." on a line by itself
C: Subject: proving a point
C: 
C: point made
C: .
S: 250 2.0.0: 832a2432 Message accepted for delivery
C: QUIT
S: 221 2.0.0: Bye
```

It's important to note that it doesn't mean a transfer is necessarily readable in the wire.
The plaintext dialogue may very well take place within an encrypted transport layer, such as TLS, making it look gibberish on the outside.
However,
no matter how it looks on the outside,
it is beautiful on the inside.


### The SMTP session
The SMTP session begins when an SMTP client establishes a connection with an SMTP server and concludes upon the client's disconnection or being disconnected. A session encapsulates "everything that transpires during a single connection."

Each SMTP session initiates with an introductory phase between the server and the client:
```
S: 220 poolp.org ESMTP OpenSMTPD
C: HELO localhost
S: 250 poolp.org Hello localhost [127.0.0.1], pleased to meet you
```

Depending on the client's method of introduction, utilizing either the HELO or EHLO verb for an extended HELO, the server may disseminate information regarding its capabilities, supported extensions, and limitations.

```
S: 220 poolp.org ESMTP OpenSMTPD
C: HELO localhost
S: 250 poolp.org Hello localhost [127.0.0.1], pleased to meet you

S: 220 poolp.org ESMTP OpenSMTPD
C: EHLO localhost
S: 250-poolp.org Hello localhost [127.0.0.1], pleased to meet you
S: 250-8BITMIME
S: 250-ENHANCEDSTATUSCODES
S: 250-SIZE 36700160
S: 250-DSN
S: 250 HELP
```

Subsequently, the client may begin a transaction to dispatch a message:

```
C: MAIL FROM:<gilles>
S: 250 2.0.0: Ok
C: RCPT TO:<gilles>
S: 250 2.1.5 Destination address valid: Recipient ok
C: DATA
S: 354 Enter mail, end with "." on a line by itself
C: Subject: proving a point
C: 
C: point made
C: .
S: 250 2.0.0: 832a2432 Message accepted for delivery
```

The session concludes either due to disconnection from either side or upon the client's explicit request to QUIT:

```
C: QUIT
S: 221 2.0.0: Bye
```

Congratulations!

You're now adept at conversing in SMTP and conducting sessions manually. Proceed to the next subsection to delve deeper into the essence of SMTP: transactions.


### The SMTP transaction
During an SMTP session, a client can send multiple messages.
Each message is handled within its own transaction,
which includes a single sender, a single message and one or many recipients.

The transaction begins when the client submits a new sender with the MAIL FROM command:

```
C: MAIL FROM:<gilles>
S: 250 2.0.0: Ok
```

The client then submits one or many recipients with the RCPT TO command:

```
C: RCPT TO:<gilles>
S: 250 2.1.5 Destination address valid: Recipient ok
```

Finally, client submits message with the DATA command and ends it with a single dot, '.', on a line by itself.

```
C: DATA
S: 354 Enter mail, end with "." on a line by itself
C: Subject: proving a point
C: 
C: point made
C: .
S: 250 2.0.0: 832a2432 Message accepted for delivery
```

The single dot on a line indicates the transaction's conclusion: it is a transaction commit request.
The client basically asks for the server to acknowledge that it will become responsible for that message being delivered to all recipients.

The server's positive response, indicated by the '250 2.0.0' code, signifies that the message has been accepted for delivery. This acknowledgment assures the client that the server will make every effort to deliver the message to all intended recipients. Additionally, the server commits to notifying the sender of any errors encountered during the delivery process. Therefore, upon receiving this response, the client can proceed with confidence, knowing that the message is in the hands of the server and will be delivered as intended: the message will not silently vanish.


### The SMTP network
The SMTP protocol operates within a network comprised of nodes commonly referred to as Mail eXchangers, or MX for short, all of which communicate using SMTP. These nodes serve the purpose of advancing a message one step closer to its destination, typically achieved by querying DNS to determine the next MX responsible for the destination. Within the network, nodes function as either relays, which accept and forward messages to other nodes, or destinations, which accept and deliver messages to recipients. From a node's perspective, the next node is always the destination; otherwise, it would be more efficient to communicate directly with the destination. Nodes rely on an envelope containing sender and recipient details, among other information, to determine whether to function as a relay or destination. In some instances, a node may act as a relay for certain domains and a destination for others.

The concepts of relay and destination are pivotal and will recur throughout this book. At this juncture, it's crucial to grasp that, from a user standpoint, the SMTP protocol solely pertains to sending mail, not retrieving it. Retrieval is managed either through direct access to the mailbox or via alternative protocols such as POP or IMAP.

The role of the SMTP server holds particular significance. A server outage results in the affected domains no longer receiving mail. Users expect their mail systems to be dependable and typically have low tolerance for disruptions. To address this, the SMTP protocol incorporates mechanisms and imposes various constraints to mitigate the impact of downtimes.
