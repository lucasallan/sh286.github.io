---
layout: post
title:  "DNScrypt on Debian based distributions" 
date:   2016-01-24 11:00:00
---

<img src='/images/Encrypt_all_the_things.png'/>


DNSCrypt turns regular DNS traffic into encrypted DNS traffic that is secure from eavesdropping and man-in-the-middle attacks. For more information, visit [DNScrypt][dnscrypt]. 

### Install dependencies:

`sudo apt-get install libsodium-dev gcc`

### Compile direct from the source

First download the latest version:

`wget https://download.dnscrypt.org/dnscrypt-proxy/dnscrypt-proxy-1.6.0.tar.gz`

Extract it:

`tar -zxvf dnscrypt-proxy-1.6.0.tar.gz; cd dnscrypt-proxy-1.6.0`

and finally, compile and install it:

`./configure && make && sudo make install`

### Create SystemD service

`sudo vim /etc/systemd/system/dnscrypt.service`

<pre><code>
[Unit]
Description=A tool for securing communications between a client and a DNS resolver.
After=network.target
Wants=network.target dnscrypt-proxy-backup.service

[Service]
ExecStart=/usr/local/sbin/dnscrypt-proxy --ephemeral-keys --resolver-name=Cisco
Restart=on-abort

[Install]
WantedBy=multi-user.target 
</code></pre>

`sudo systemctl start dnscrypt.service`

### Check status

`sudo systemctl status -l dnscrypt.service`

And the last line should be something like:

`Jan 22 15:31:46 sh286 dnscrypt-proxy[17757]: [NOTICE] Proxying from 127.0.0.1:53 to 208.67.220.220:443`

### Start dnscrypt service during boot

`sudo systemctl enable dnscrypt.service`


### Update your default DNS service to use it

`sudo vim /etc/resolv.conf`

```
nameserver 127.0.0.1
```

By default, NetworkManager overrides `/etc/resolv.conf` when you connect to the internet.
We can disable it by editing `/etc/NetworkManager/NetworkManager.conf` and
change the line `dns=<something>` to `dns=none`.

Then restart NetworkManager manager:

`sudo service NetworkManager restart`


[dnscrypt]: https://www.dnscrypt.org/
