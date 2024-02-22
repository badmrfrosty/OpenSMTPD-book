# Preface

In 2007, despite considering myself a seasoned sysadmin with a decade of managing mail services under my belt, a simple request from a user to tweak their mail setup threw me for a loop. Up until that moment, navigating the complexities of Sendmail and Postfix was a no-brainer for me, something I could do almost on autopilot.

The request, now somewhat blurred in my memory and perhaps intentionally forgotten, seemed straightforward enough. Yet, as I delved into the abyss of Sendmail's configuration labyrinth, certainty eluded me. The changes I made seemed increasingly alien, and I couldn't shake off the feeling that I was about to unleash chaos.

Eventually, I did what seemed the only logical step: I crossed my fingers and pushed the changes live[<sup>1</sup>](#1). The result? Catastrophe. The SMTP Daemon, once a reliable ally, now seemed to mock me as I hastily rolled back the changes.

Seeking solace and a distraction, I met up with a friend for what was supposed to be a casual evening of beers and coding. That night, fueled by a mix of frustration and alcohol, the first lines of a new SMTP server were born. By the end of the evening, my makeshift server was flouting every rule in the book, yet miraculously sending emails to my inbox.

Fast forward, and with the invaluable input from a dedicated community, OpenSMTPD has transformed from those humble, rule-breaking beginnings into a robust, feature-rich, and elegantly simple mail server. It's a testament to clean design and scalability, and I'm convinced it's destined for greatness.

-- Gilles Chehade

<hr />
[<sup>1</sup>](#1) Don't do that.
