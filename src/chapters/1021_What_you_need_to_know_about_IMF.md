# What you need to know about the Internet Message Format

    Information is a source of learning. But unless it is organized, processed, and available to the right people in a format for decision making, it is a burden, not a benefit.
    -- William Pollard


## The Internet Message Format
The Internet Message Format (IMF), standardized in RFC 5322, facilitates the exchange of messages between mail exchangers during the "DATA" phase of an SMTP transaction:

```
C: DATA
S: 354 Enter mail, end with "." on a line by itself
C: Subject: proving a point
C: 
C: point made
C: .
S: 250 2.0.0: 832a2432 Message accepted for delivery
```

It's essential to recognize that while mail exchangers utilize the IMF, the primary consumers are MUA software, which rely on its structure to appropriately display messages for end-users. From the perspective of a mail exchanger, an IMF could be poorly crafted without impacting the message transfer. The use of IMF serves as a convention for interoperability.


### The basic structure
In its simplest form, the IMF comprises two main sections: the headers part and the body part. These sections are separated by an empty line:

```
some_header: foo
another_header: bar

this is a very nice body,
spanning on multiple lines,
including empty lines,
it is the first empty line that separates headers from body.
```

### The headers part
Headers serve as metadata that aids in interpreting a message. They encompass fields essential for displaying messages in an easily readable manner for the MUA (Mail User Agent), providing senders with details such as subject, name, or date, and assisting mail exchangers in troubleshooting transfers.

Each header comprises a header name and a header value, delineated by a colon and a space ":", and may extend across multiple lines, with continuation lines indicated by preceding whitespace:

```
header_name: header_value
long_header_name: value
  continues here
```

Headers adhere to certain restrictions regarding their names, value lengths, and structure. While some headers are mandatory or optional and may appear once or multiple times, others are custom and relevant only for specific applications. The book will cover pertinent details as needed, without delving into extensive technicalities.


### The body part
The body section contains the actual message content, excluding metadata. This encompasses what the original sender intends for the recipient to read.

Similar to headers, there are constraints regarding line lengths, but such details are beyond the scope of this book and can be explored in the RFC for those interested.

In a straightforward IMF, the headers part typically includes a few headers, while the body part contains a human-readable message:

```
Date: Wed, 16 Dec 2020 10:41:51 +0200
From: Gilles Chehade <gilles@poolp.org>
To: Eric Faurot <eric@faurot.net>
Subject: quick question for you

helo eric,
I was just wondering...
Any chance you could review my diff before I die of old age ?
```

In this example,
the body consists of simple ASCII message which will be displayed as is by the MUA.
The headers could also provide hints to display the message in a different format or encoding,
in HTML and UTF-8 for example,
leaving it up to the MUA to properly deal with that information:

```
Date: Wed, 16 Dec 2020 10:41:51 +0200
From: Gilles Chehade <gilles@poolp.org>
To: Eric Faurot <eric@faurot.net>
Subject: quick question for you
Content-Type: text/html; charset=utf-8

<html>
  <body>
    <h1>helo eric,</h1>
    <p>
      I was just wondering...
      Any chance you could review my diff before I die of old age ?
    </p>
  </body>
</html>
```

The example demonstrates a basic ASCII message displayed as intended by the sender. However, headers can also convey instructions for displaying the message in various formats or encodings, such as HTML and UTF-8.

The distinction between the "SMTP envelope" and the "DATA envelope" is essential. While the former includes only email addresses, the latter may incorporate additional user-friendly details like first and last names. Although they may align, discrepancies between the SMTP sender and the DATA "From" header are common.



### Multi-part messages
Modern messages often extend beyond simple ASCII text. Some include attachments or offer both HTML and plain text versions of the same content, allowing MUAs to determine the most suitable presentation.

Though this book primarily focuses on basic IMF structures, the concept of Multipurpose Internet Mail Extensions (MIME) enables the integration of multiple message components into a single message. While mail exchangers only concern themselves with the top-level structure, the nested structures facilitated by MIME offer enhanced message structuring capabilities. However, readers need not worry too much about understanding this, as the book will provide a straightforward example before moving forward:

```
Date: Wed, 16 Dec 2020 10:41:51 +0200
From: Gilles Chehade <gilles@poolp.org>
To: Eric Faurot <eric@faurot.net>
Subject: quick question for you
MIME-Version: 1.0
Content-Type: multipart/alternative;
  boundary="--==_mimepart_5b784aee4aca8_643f3fbdb76d45c04373fd";
  charset=utf-8

--==_mimepart_5b784aee4aca8_643f3fbdb76d45c04373fd
Content-Type: text/plain; charset=utf-8
    
helo eric,
I was just wondering...
Any chance you could review my diff before I die of old age ?
--==_mimepart_5b784aee4aca8_643f3fbdb76d45c04373fd
Content-Type: text/html; charset=utf-8

<html>
  <body>
    <h1>helo eric,</h1>
    <p>
      I was just wondering...
      Any chance you could review my diff before I die of old age ?
    </p>
  </body>
</html>  
--==_mimepart_5b784aee4aca8_643f3fbdb76d45c04373fd--
```

Essentially, the top structure of the email defines it as a multipart message, indicating that it contains multiple sections with different content types. These sections are separated by boundary delimiters. Each section acts as a distinct email message, complete with its own headers and body content, specifying the type of content it contains.

In our example, the email includes both HTML and plain text versions of the same message. The top-level structure's content type provides a clue to the email client about which version to display first. So, if you ever find yourself frustrated by an email displaying in HTML when you prefer plain text, it's likely because either the email wasn't multipart and lacked a plain text version, or the Content-Type header didn't provide the necessary guidance to your email client.

Multipart messages can serve various purposes. While our example aimed to provide alternative formats for the message, other parts could include attachments or encode images referenced in the HTML version. How these parts are handled depends on the headers present and the capabilities of your email client.
