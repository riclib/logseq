---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/sending-email-with-the-alertmanager-via-gmail
author: [[Brian Brazil]] 
---
# Sending email with the Alertmanager via Gmail

> A blog on monitoring, scale and operational Sanity

---
A blog on monitoring, scale and operational Sanity

We've already looked at how the [Prometheus](https://prometheus.io/) Alertmanager can talk to [Pagerduty](http://www.robustperception.io/using-pagerduty-with-the-alertmanager/) and [Slack](http://www.robustperception.io/using-slack-with-the-alertmanager/). It also works with SMTP, more commonly known as email. Let's see how to get it working via a Gmail account.

Let's say we want to deliver email from the Alertmanager out to somewhere on the internet. The Alertmanager isn't a full SMTP server itself, however it can pass on emails to something like Gmail which can send them on on your behalf. While this is possibly not the best of ideas for a production setup, it's fine for personal use.

For security you shouldn't use your main Gmail password. Instead rather [generate an app password](https://support.google.com/accounts/answer/185833?hl=en). Once that's done run the following in a shell, substituting in your Gmail address and app password:

```shell
GMAIL_ACCOUNT=me@example.com # Substitute in your full gmail address here.
GMAIL_AUTH_TOKEN=XXXX        # Substitute in your app password
wget https://github.com/prometheus/alertmanager/releases/download/v0.8.0/alertmanager-0.8.0.linux-amd64.tar.gz
tar -xzf alertmanager-0.8.0.linux-amd64.tar.gz
cd alertmanager-*

cat <<EOF > alertmanager.yml
route:
  group_by: [Alertname]
  # Send all notifications to me.
  receiver: email-me

receivers:
- name: email-me
  email_configs:
  - to: $GMAIL_ACCOUNT
    from: $GMAIL_ACCOUNT
    smarthost: smtp.gmail.com:587
    auth_username: "$GMAIL_ACCOUNT"
    auth_identity: "$GMAIL_ACCOUNT"
    auth_password: "$GMAIL_AUTH_TOKEN"
EOF
./alertmanager &
```
If you send an alert, you'll see a notification in your Gmail account:

[![](http://www.robustperception.io/wp-content/uploads/2016/04/Screenshot-from-2016-04-13-124018.png)](http://www.robustperception.io/wp-content/uploads/2016/04/Screenshot-from-2016-04-13-124018.png)
