# Signal TLS Proxy for FreeBSD & OpenBSD
An nginx.conf and other basic security configs, allowing FreeBSD and OpenBSD to run a Signal TLS Proxy (#IranASignalProxy)

## Rationale
Having run Signal proxies using Docker on Linux since their inception, I wanted to move to FreeBSD as the host OS. Signal advertise only Docker as the way to run a proxy. This is relatively easy (if you know Docker), and can be stable once running. It's also somewhat convoluted, and requires the user to install Docker if the host OS doesn't include it by default (or in its repos). It also adds a further layer of abstraction and hassle (eg firewalling, routing) and overhead, that in my opinion is neither necessary or desired. 

Because they're stable, sane and nice to work with, I prefer the BSDs. Having followed [Neel Chauhan's guide here](https://www.neelc.org/posts/freebsd-signal-proxy/), the proxy didn't work. Upon review of the [official Signal nginx config](https://github.com/signalapp/Signal-TLS-Proxy/blob/master/data/nginx-relay/nginx.conf) (which makes up part of their Docker setup), it became apparent that the upstream URLs the Signal messenger app connects to had changed.  

The FreeBSD portion of this repo contains my nginx.conf as updated to reflect those upstream changes, and allows the user to run a Signal proxy on FreeBSD. I then further refined the config to work on OpenBSD 7.0 (which was trivial). I supply those configs and related security files (firewall configs, sshguard configs) for my own use, and in the hope that they help someone else likewise wishing to host Signal TLS proxies on the BSDs. 

## Steps to run a Signal TLS proxy on FreeBSD

(1) Install acme.sh or certbot (or similar) and obtain a cert for your domain. That's outside of the scope of this brief guide, but essentially culminates with `certbot --certonly -D domain.xyz`. 

(2) Amend the enclosed nginx.conf to fit your requirements. Change `PATH` to the location in which your certs from step (1) are stored (e.g. `/usr/local/etc/letsencrypt/live/domain.xyz/fullchain.pem`).

(3) Install nginx (`pkg install nginx`) and move this nginx.conf to `/usr/local/etc/nginx/nginx.conf`. Don't forget your changes from step *(2)*!

(4) Enable and start Nginx with `sysrc nginx_enable=YES` and `service nginx start`. 

(5) Done! The proxy is now online. 

(6) If you have the IPFW running (and you should!), you'll need to allow at least ports 80/tcp and 443/tcp. You can drop port 80 once you have your certs if required, the proxy runs on 443/tcp. There is a basic config (block hostiles, allow 80/tcp and 443/tcp) in the `FreeBSD` folder.  

## Steps to run a Signal TLS proxy on OpenBSD

(1) Install one of the acme* packages or certbot (or similar), and obtain a cert for your domain. That's outside of the scope of this brief guide, but essentially culminates with `certbot --certonly -D domain.xyz`. 

(2) Amend the enclosed nginx.conf to fit your requirements. Change `PATH` to the location in which your certs from step are stored (e.g. `/etc/letsencrypt/live/tahm.xyz/fullchain.pem`).

(3) Install nginx (`pkg_add nginx nginx-stream`) and move this nginx.conf to `/etc/nginx/nginx.conf`. Don't forget your changes from step *(2)*!

(4) Enable and start Nginx with `rcctl enable nginx` and `rcctl start nginx`. 

(5) Done! The proxy is now online. 

(6) If you have the `pf` running (and you should!), you'll need to allow at least ports 80/tcp and 443/tcp. You can drop port 80 once you have your certs if required, the proxy runs on 443/tcp. There is a basic config (block hostiles, allow 80/tcp and 443/tcp) in the `OpenBSD` folder.

##
## All being well, your proxy is now live and you can share it in the normal way using `https://signal.tube/#domain.xyz`. Congratulations! If you have any problems please open an Issue.
