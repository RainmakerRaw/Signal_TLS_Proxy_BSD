# Signal TLS Proxies, for FreeBSD & OpenBSD. No Docker required!
An nginx.conf and other basic security configs, allowing FreeBSD and OpenBSD to run a Signal TLS Proxy (#IranASignalProxy)

## Rationale
Having run Signal proxies using Docker on Linux since their inception, I wanted to move to FreeBSD as the host OS. Signal advertise only Docker as the way to run a proxy. This is relatively easy (if you know Docker), and can be stable once running. It's also somewhat convoluted, and requires the user to install Docker if the host OS doesn't include it by default (or in its repos). It also adds a further layer of abstraction and hassle (eg firewalling, routing) and overhead, that in my opinion is neither necessary or desired.  

Finally, I kept bumping into stupid issues such as Ubuntu pushing an update that hosed all my proxies at the same time (killed the net config), or a kernel update rebooting the machine and having it not come back up again. Runnning one of the BSDs avoids all this drama. It allows a simple, Docker free, systemd free, and 'change everything, all the time, because reasons' attitude-free setup, balanced across multiple OS so that one bad update/config/server doesn't take down your whole infrastructure.  

There are real, live, sometimes desperate human beings at the other end of these proxies. They are also, often in perilous situations where their privacy and security are one and the same thing. Having all your available proxies go down because $distro pushed an update that borked your whole infrastructure is obviously not good. Now, with these configs, you can run a Linux VPS, and a FreeBSD VPS (on QUARTERLY or CURRENT package updates, or both) and an OpenBSD VPS.  

If one breaks or goes down or has a bad update, you have options. Redundancy! I can't tell you how stressful it is to have hundreds of users, from  dozens of countries, all sending you messages at the same time pleading that the proxy is down... and you're out of options, because your servers are cattle, but all the same *kind* of cattle. This way, you can say 'Sorry about that, here's two more that **do** work!'. Peace of mind is priceless, for you and your users; but unfortunately VPS are not! I'm spending a fortune - who wants to sponsor me some slots? Netcup? Vultr? Linode? DO? Anyone??...

Because they're stable, sane and nice to work with, I prefer the BSDs. Having followed [Neel Chauhan's guide here](https://www.neelc.org/posts/freebsd-signal-proxy/), the proxy didn't work. Upon review of the [official Signal nginx config](https://github.com/signalapp/Signal-TLS-Proxy/blob/master/data/nginx-relay/nginx.conf) (which makes up part of their Docker setup), it became apparent that the upstream URLs the Signal messenger app connects to had changed.  

The FreeBSD portion of this repo contains my nginx.conf as updated to reflect those upstream changes, and allows the user to run a Signal proxy on FreeBSD. I then further refined the config to work on OpenBSD 7.0 (which was trivial). I supply those configs and related security files (firewall configs, sshguard configs) for my own use, and in the hope that they help someone else likewise wishing to host Signal TLS proxies on the BSDs.  

This guide assumes your server already has basic hardening (`sshguard` and `ipfw` enabled etc).  

## Steps to run a Signal TLS proxy on FreeBSD

(1) Install acme.sh or certbot (or similar) and obtain a cert for your domain. That's outside of the scope of this brief guide, but essentially culminates with `certbot certonly -d domain.xyz`. 

((2) Amend the enclosed nginx.conf to fit your requirements. Change `DOMAIN` accordinly in the ssl_certificate paths, which should reflect where your certs from step are stored (e.g. `/etc/letsencrypt/live/domain.xyz/fullchain.pem`).

(3) Install nginx (`pkg install nginx`) and move this nginx.conf to `/usr/local/etc/nginx/nginx.conf`. Don't forget your changes from step *(2)*!

(4) Enable and start Nginx with `sysrc nginx_enable=YES` and `service nginx start`. 

(5) Done! The proxy is now online. 

(6) If you have the IPFW running (and you should!), you'll need to allow at least ports 80/tcp and 443/tcp. You can drop port 80 once you have your certs if required, the proxy runs on 443/tcp. There is a basic config (block hostiles, allow 80/tcp and 443/tcp) in the `FreeBSD` folder.  
  
Because of IPFW's syntax, the config includes ipv6 by default. It doesn't matter whether you actually have it available on your server or not. Make sure to change WAN_INTERFACE to the name of your server's WAN/external/egress interface in the config before applying it with `ipfw service restart`. See [The FreeBSD Handbook](https://docs.freebsd.org/en/books/handbook/firewalls/#firewalls-ipfw) for details on how to enable IPFW in the first place.  

## Steps to run a Signal TLS proxy on OpenBSD

(1) Install one of the acme* packages or certbot (or similar), and obtain a cert for your domain. That's outside of the scope of this brief guide, but essentially culminates with `certbot certonly -d domain.xyz`. 

(2) Amend the enclosed nginx.conf to fit your requirements. Change `DOMAIN` accordinly in the ssl_certificate paths, which should reflect where your certs from step are stored (e.g. `/etc/letsencrypt/live/domain.xyz/fullchain.pem`).

(3) Install nginx (`pkg_add nginx nginx-stream`) and move this nginx.conf to `/etc/nginx/nginx.conf`. Don't forget your changes from step *(2)*!

(4) Enable and start Nginx with `rcctl enable nginx` and `rcctl start nginx`. 

(5) Done! The proxy is now online. 

(6) If you have the `pf` running (and you should!), you'll need to allow at least ports 80/tcp and 443/tcp. You can drop port 80 once you have your certs if required, the proxy runs on 443/tcp. There is a basic config (block hostiles, allow 80/tcp and 443/tcp) in the `OpenBSD` folder.  
  
The config includes ipv6, and you can safely leave this in (or remove it) if you don't have IPv6 connectivity on your server. See [OpenBSD's pf documentation](https://www.openbsd.org/faq/pf/) for details on how to enable pf in the first place, but it's basically a case of copy in the config to `/etc/pf.conf`, then `pfctl -f /etc/pf.conf` to load the config, then finally `pfctl -e` to enable pf.  

##
## All being well, your proxy is now live and you can share it in the normal way using `https://signal.tube/#domain.xyz`. Congratulations! If you have any problems please open an Issue.
