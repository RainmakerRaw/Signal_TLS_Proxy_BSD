# Signal proxies for FreeBSD & OpenBSD - no Docker required!

An Nginx configuration with accompanying firewall rules and SSHGuard config, allowing use of FreeBSD and/or OpenBSD to run a Signal TLS Proxy safely, and with minimal fuss. #IranASignalProxy

## Table of contents
- [Rationale](#rationale)
- [Steps to run a Signal TLS proxy on FreeBSD](#steps-to-run-a-signal-tls-proxy-on-freebsd)
- [Steps to run a Signal TLS proxy on OpenBSD](#steps-to-run-a-signal-tls-proxy-on-openbsd)
- [Conclusion](#conclusion)

## Rationale

I have been running [Signal TLS proxies](https://signal.org/blog/help-iran-reconnect/) for users around the world, ever since the code to run them was released. Having previously run them the 'usual' way, using Docker on Linux, I wanted to move to BSD as the host OS. Why? Because awesomesauce. 

Signal advertise Docker on Linux as the only way to run a proxy. This is relatively easy (if you know Docker), and can be quite stable once running. However, it's also somewhat convoluted, and adds a further layer of abstraction and hassle that - in my opinion - is neither necessary or desired. It also leaves you at the mercy of whatever Linux distro you started with, either through choice or as a consequence of the limited offerings of your VPS provider. 

For me, over the course of a year or so in public-facing production, it was death by a thousand cuts and a source of never-ending niggling workloads. Update this, tweak that, fix the other... I kept on bumping into stupid issues, such as Ubuntu server pushing an update that hosed all my proxies at the same time (they killed the net config), or their kernel update that rebooted the machines but promptly refused to bring them back up again.

When running all my servers as cattle with the same (or similar) base OS, one issue took down the whole project. Literally. I can't tell you how stressful it is to have hundreds of users, from dozens of countries, all sending you messages at the same time pleading that the proxy is down - and you're out of options, because not only are your servers cattle, but they're all the same *kind* of cattle.

Using the BSDs as a base avoids all this drama, for several reasons: 

* The whole OS is developed in-house, as a single entity, with a focus on stability, speed and correctness. 
* Once something works, it's refined and battle-hardened over years rather than being perpetually replaced with the latest shiny toy every time the OS produces its next release. 
* Updates are released when actually really needed, rather than because 1,000 upstream projects all released feature updates/patches/security fixes for their 1,000 separate applications. Or, worse still, because the calendar said so.
* Simplicity. Nothing's overly complicated for complexity's sake - programs have one job and they do it correctly.  
* Running FreeBSD alongside OpenBSD gives similar but different-enough systems, allowing easy administration but with redundancy if one or the other OS suffers an issue. 

For those reasons and more, BSD provides a cohesive and rock solid platform. It allows a simple, Docker free, systemd free, and 'change everything, all the time, because reasons' attitude-free setup. With that in mind, I started looking for a way to move my proxies away from Docker on Linux, and onto either FreeBSD or OpenBSD. 

Unfortunately, there was next to nothing available online about it. Everyone was seemingly just running Docker on Linux, as per Signal's original blog post. Eventually, I did find and excitedly follow [Neel Chauhan's guide](https://www.neelc.org/posts/freebsd-signal-proxy/) to run a proxy using Nginx directly on FreeBSD. Unfortunately though, the resulting proxy didn't work. Upon review of the [official Signal nginx config](https://github.com/signalapp/Signal-TLS-Proxy/blob/master/data/nginx-relay/nginx.conf) (which makes up part of their Docker setup), it became apparent that the upstream URLs the Signal messenger app connects to had changed since Neel wrote his guide. 

So, I wrote an updated `nginx.conf`, along with companion IPFW and SSHGuard configs to help secure it, and the whole thing now works perfectly as of FreeBSD 13! I have contacted Neel with the relevant details, so he can update his blog/guide if he wishes. After that, it was trivial to port the config to OpenBSD (the `nginx.conf` syntax is slightly different) and add an accompanying `pf.conf` for the firewall. Done!

With these configs, you can still run a Linux VPS if you wish... and a FreeBSD VPS... and an OpenBSD VPS. Redundancy! Balancing proxies across two of the BSDs (and maybe some Linux for good measure), one bad update or pushing an incorrect config doesn't have to take down the whole infrastructure. In fact, it just can't. If a borked update or bad config push to a Linux server takes it down, the FreeBSD and OpenBSD servers remain unaffected until it's fixed, and vice versa. Now it's possible to load balance connections (and users) across a varied but highly dependable base. 

There are real, live, sometimes desperate human beings at the other end of these proxies. They are also, often in perilous situations where their privacy and security are one and the same thing. Having all your available proxies go down because $distro pushed an update that borked your whole infrastructure is obviously not good. With the configs in this repository we can keep our options open. I have uploaded the resulting files here for my own record keeping and future use, but in the hope that someone else may find them useful. 

This brief guide assumes your OS already has basic hardening tools installed (i.e. `sshguard` and `ipfw` or `pf`) as part of your installation and basic setup of the server/VPS. 

## Steps to run a Signal TLS proxy on FreeBSD

(1) Install Certbot (or similar) and obtain a certificate for your domain. That's outside the scope of this brief guide, but essentially begins with `pkg install py38-certbot` and culminates with `certbot certonly -d domain.xyz`. 

(2) Amend the enclosed `nginx.conf` to fit your requirements. Change `DOMAIN` in the `ssl_certificate` paths, to reflect where the certs you generated in step *(1)* are stored (e.g. `/etc/letsencrypt/live/domain.xyz/fullchain.pem`).

(3) Install nginx (`pkg install nginx`) and move the `nginx.conf` you just modified to replace the stock one at `/usr/local/etc/nginx/nginx.conf`. 

(4) Enable and start Nginx with `sysrc nginx_enable=YES` and `service nginx start`. 

(5) Done! The proxy is now online. 

(6) If you have the IPFW running (and you should!), you'll need to allow at least ports 80/tcp for Certbot and 443/tcp for the Nginx proxy itself. There is a basic config (block hostiles, allow established connections, allow new connections in on 80/tcp and 443/tcp) in the `FreeBSD` folder.  

Because of IPFW's syntax, the config includes ipv6 by default. It doesn't matter whether you actually have it available on your server or not. Make sure to change WAN_INTERFACE to the name of your server's WAN/external/egress interface in the config before applying it with `ipfw service restart`. See [The FreeBSD Handbook](https://docs.freebsd.org/en/books/handbook/firewalls/#firewalls-ipfw) for details on how to enable IPFW in the first place.  

## Steps to run a Signal TLS proxy on OpenBSD

(1) Install Certbot (or similar) and obtain a certificate for your domain. That's outside the scope of this brief guide, but essentially begins with `pkg_add certbot` and culminates with `certbot certonly -d domain.xyz`.

(2) Amend the enclosed `nginx.conf` to fit your requirements. Change `DOMAIN` in the `ssl_certificate` paths, to reflect where the certs you generated in step *(1)* are stored (e.g. `/etc/letsencrypt/live/domain.xyz/fullchain.pem`).

(3) Install nginx and the stream module (`pkg_add nginx nginx-stream`) and move the `nginx.conf` you just modified to replace the stock one at `/etc/nginx/nginx.conf`. 

(4) Enable and start Nginx with `rcctl enable nginx` and `rcctl start nginx`. 

(5) Done! The proxy is now online. 

(6) If you have the `pf` running (and you should!), you'll need to allow at least ports 80/tcp for Certbot and 443/tcp for the Nginx proxy itself. There is a basic config (block hostiles, allow established connections, allow new connections in on 80/tcp and 443/tcp) in the `OpenBSD` folder.  

The `pf.conf` includes rules for IPv6, and you can safely leave this in (or remove it) if you don't have IPv6 connectivity on your server. See [OpenBSD's pf documentation](https://www.openbsd.org/faq/pf/) for details on how to enable pf in the first place, but it's basically a case of copy in the config to `/etc/pf.conf`, then `pfctl -f /etc/pf.conf` to load the config, then finally `pfctl -e` to enable pf.  

## Conclusion

All being well, your proxy is now live. Congratulations! You can share it with needy users in the normal way using `https://signal.tube/#DOMAIN.xyz`, through whatever channels you have available. On Twitter, you can post using the hashtag #IRanASignalProxy to advertise to those looking for the use of one. There are also threads running to advertise proxies on Reddit, Signal's community forum and others. 

If you have any problems or comments, please feel free to open an Issue. 
