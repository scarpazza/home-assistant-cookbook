# Enabling SSL encryption in Home Assistant Core 

This is work in progress.


## Goal

This is tutorial covers the steps necessary to make Home Assistant connections encrypted, for users:
* who run Home Assistant Core, and consequently have no access to the add-on store
* on Linux
* who use duckdns.org as a dynamic DNS provider
* who desire a free SSL certificate from letsencrypt.org
* whose ISP filters port 80, and consequently can not use certbot's http challenge (that is tied to port 80)                                                                                                                                                
Users with increasingly different setups will find this tutorial less and less useful. 

Reference:
* https://certbot.eff.org/docs/using.html?highlight=dns#manual
* https://letsencrypt.org/docs/challenge-types/#dns-01-challenge

## Step 1: install certbot

```
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
       
       
## Step 2: request the DNS challenge

Certbot expects you to prove control over your dns entry by place a TXT DNS record with randomized contents it specifies, under the domain name consisting of the hostname for which you want a certificate issued, prepended by `_acme-challenge`. 

Even if certbot will request a TXT record for name `_acme-challenge.your_domain.duckdns.org`, it is not necessary (and even possible) for you to create that sub-sub-domain. That's because duckdns will automatically present subdomain information to any sub-subdomain queries.


```
certbot certonly --manual -d your_domain.duckdns.org -m you@your_email_provider.com --preferred-challenge dns --agree-tos
```

Certbot will respond with:
```
Please deploy a DNS TXT record under the name
_acme-challenge.<your subdomain>.duckdns.org with the following value:

xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```


## Step 3: populate your DNS TXT record


Execute the following:
```
export DOMAIN=your_domain
export VALUE=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
export TOKEN=12345678-abcd-9876-fedc-9876543210ab
export URL="https://www.duckdns.org/update?verbose=true&domains=${DOMAIN}&token=${TOKEN}&txt=${VALUE}"
curl -v --url $URL 
```

Expect an OK answer.

## Step 4: confirm that your TXT record is active

Do that via `dig your_domain.duckdns.org TXT`

Verify that the TXT string dig fetches matches the one certbot requested.

```
; <<>> DiG 9.16.1-Ubuntu <<>> your_domain.duckdns.org TXT
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54472
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;your_domain.duckdns.org.           IN      TXT

;; ANSWER SECTION:
your_domain.duckdns.org.    60      IN      TXT     "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

;; Query time: 156 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Wed Jan 27 20:52:57 EST 2021
;; MSG SIZE  rcvd: 104
```

## Step 5: complete the challenge

```
Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your_domain.duckdns.org/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your_domain.duckdns.org/privkey.pem
   Your cert will expire on 2021-04-28. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

## Step 6: set up Home Assistant

Permission the certificates for use by the non-privileged users under which Home Assistant runs:
```
sudo chmod a+rX /etc/letsencrypt/live
sudo chmod a+rX -R /etc/letsencrypt/archive/
```

Edit your `configuration.yaml`

```
http:
  ssl_certificate: /etc/letsencrypt/live/your_domain.duckdns.org/fullchain.pem        
  ssl_key: /etc/letsencrypt/live/your_domain.duckdns.org/privkey.pem
  ip_ban_enabled: true
  login_attempts_threshold: 5  
```


## Step 7: test on your desktop browser

* Point your desktop browser to your external URL (e.g., https://your_domain.duckdns.org:8123) and ensure you can log in successfully
* Point your desktop browser to your internal URL (e.g., https://192.168.x.y:8123)
  * accept the security exception associated with the website name on the certificate not matching its address. In modern Chrome browsers, this is done by typing the string `thisisunsafe` when presented with the warning message, even if no cursor is visible. You only need to perform this once;
  * ensure you can log in successfully
 
## Step 8: reconfigure app URLs

* Change your external access URL from http://your_domain.duckdns.org:8123 to https://your_domain.duckdns.org:8123 
* Change your internal access URL from http://192.168.x.y:8123 to https://192.168.x.y:8123 and accept the security exception associated with the website name on the certificate not matching its address

* Disconnect your mobile phone from your home wi-fi and test HA functionality (external URL)
* Reconnect your mobile phone to your home wi-fi and test HA functionality (internal URL)

## Step 9: add an internet outage link

Add a sidebar link that will allow you to switch to the internal URL if you have an internet outage:
```
panel_iframe:
  internal:  
    title: 'Switch to internal'
    url: 'https://192.168.x.y:8123'
    icon: hass:close-network-outline   
```

## Step 10: certbot periodic recertification
Repeate steps 2 ... 5 when you receive a reminder from letsencrypt.org recommending a renewal.


