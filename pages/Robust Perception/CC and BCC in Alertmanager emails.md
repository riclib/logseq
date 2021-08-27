---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/cc-and-bcc-in-alertmanager-emails
author: [[Brian Brazil]] 
---
> If you look at the documentation, the Alertmanager has a to field, but nothing for CC or BCC. So how do you do those?

# CC and BCC in Alertmanager emails


If you look at the [documentation](https://prometheus.io/docs/alerting/configuration/#email_config), the Alertmanager has a `to` field, but nothing for CC or BCC. So how do you do those?

To explain this, we first need to dig into how email, and in particular the email protocol SMTP, works. There's two parts to sending an email, the envelope which consists of the commands used when talking to the SMTP server, and then the message (or data) containing the headers and body that you're used to seeing. This is similar to an postal mail, where there's what's written on the envelope and then what's written on the paper inside.

When sending an email there will be at least one `RCPT TO` command, indicating where the email is to actually get sent to. For example if part of the SMTP exchange was
```
RCPT TO:<foo@example.com>
250 OK
RCPT TO:<bar@example.com>
250 OK
```
then we'd be asking to send this email to foo@example.com and bar@example.com. The key point to understand is that at the SMTP level any `To`, `CC` ([carbon copy](https://en.wikipedia.org/wiki/Carbon_copy)), or `BCC` ([blind carbon copy](https://en.wikipedia.org/wiki/Blind_carbon_copy)) headers inside the message don't affect the `RCPT TO` which is where the email is actually going to be sent (though anti-spam systems may have opinions if they don't match).

If you're using a typical email client (a Mail User Agent or MUA if we're being precise) it'll handle things for you under the covers when it talks SMTP. Any addresses in `To` or `CC` headers will be put in a `RCPT TO`, and left in the message. Any addresses in `BCC` headers will also be put in a `RCPT TO`, however the `BCC` headers will be be removed from the message.

What the Alertmanager offers is SMTP more than email. The Alertmanager will default the `To` header to the value of the `to` configuration field, but beyond that we're on our own. So here's a configuration fragment showing all of these used normally:
```
email\_config:
to: to@example.com,cc@example.com,bcc@example.com
headers:
To: to@example.com
CC: cc@example.com
```
Here all three addresses are in the `to` and will be sent the email, but only to@example.com and cc@example.com will end up in the message headers.

The `from` field, `From` header, and `MAIL FROM` SMTP commands work similarly. All of this can be used to have an email that says it was did one thing but actually did something else. That's generally considered [abuse and/or spam](https://en.wikipedia.org/wiki/Email_spoofing) though, so don't do that.

While the Alertmanager does offer you the power to craft the exact email you want, I'd recommend keeping things simple over doing complicated templating to get things perfect.

Any email addresses you wanted to CC, instead settle for having them in `To`. So include them only in the `to` field and don't specify explicit `To` or `CC` headers, which has the same practical effect of sending to certain addresses and listing them in the headers. Instead of doing a BCC use _one of the techniques_[^1] to send a notification to a 2nd destination, which will also avoid the other recipients being aware of it.

[^1] [[Sending alert notifications to multiple destinations]]