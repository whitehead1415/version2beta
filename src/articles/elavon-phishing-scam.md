---
pagetitle: Elavon Phishing Scam
longtitle: I didn't find anything useful on Google regarding phishing scam, so I figured I'd put something out here.
author: Rob Martin
published: 2011-11-28 11:20
snippet: A customer received 40,000 mailer-daemon messages yesterday. I bet that was fun.
---

A customer received 40,000 mailer-daemon messages yesterday. I bet that was fun.

I've seen only four or five of the emails, but it was enough to demonstrate that someone had credentials to use the SMTP server to send phishing messages pretending to be from Elavon Merchant Services.

The text of the message goes something like this:

> Elavon Virtual Merchant Account Alert -
>
> Your RETAIL account has expired. If you intend to use our services in the future you have to update your online info now.
>
> - Download the attached login page in this email to continue. -
>
> *** Please note : Only Retail accounts are locked. (Example : Market Segment : Retail )
>
> Thank you - Elavon Merchant Services Team.

The attached HTML file looks like a login form for Elavon, but sends the data to a Japanese website http://depot-abc.com, deep linked into a Wordpress installation, probably hacked.

While it looked like the client's computer might have been compromised, it turned out the messages were being sent from colocated or shared servers, at least the few I tracked down. Most of the messages I looked at were sent from The Planet in Texas, which I believe is still the world's largest privately held hosting facility (hosting services, not like Facebook or Google which undoubtedly have bigger facilities.) Some of the other messages came from a machine hosted at Netelligent.ca, providing hosting services in and around Montreal. In this case, the machine seems to be in Laval, QC.
