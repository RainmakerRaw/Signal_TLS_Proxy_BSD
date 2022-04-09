# Signal TLS Proxy for FreeBSD
An nginx.conf allowing FreeBSD to run a Signal TLS Proxy (#IranASignanProxy)

### Rationale
Having run Signal proxies since their inception, I wanted to move to FreeBSD as the host OS. Signal advertise only Docker as the way to run a proxy. While this is somewhat easy (if you know Docker) and can be stable once running, it's also somewhat convoluted and requires the user to install Docker if the host OS doesn't include it by default.  

Because they're stable, sane and nice to work with, I prefer the BSDs. Having followed [Neel Chauhan's guide here](https://www.neelc.org/posts/freebsd-signal-proxy/), the proxy didn't work. Upon review of the [official Signal nginx config](https://github.com/signalapp/Signal-TLS-Proxy/blob/master/data/nginx-relay/nginx.conf) (which makes up part of their Docker setup), it became apparent that the upstream URLs the Signal messenger app connects to had changed.  

This nginx.conf is updated to reflect those changes and allow the user to run a Signal proxy on FreeBSD.

### Steps to run a Signal TLS proxy on FreeBSD

(1) Install acme.sh or certbot (or similar) and obtain a cert for your domain. That's outside of the scope of this brief guide, but essentially culminates with `certbot --certonly -D domain.xyz`. 

(2) Amend the enclosed nginx.conf to fit your requirements. Change `PATH` to the location in which your certs from step (1) are stored (e.g. `/usr/local/etc/letsencrypt/live/domain.xyz/fullchain.pem`).

(3) Install nginx (`pkg install nginx`) and move this nginx.conf to `/usr/local/etc/nginx/nginx.conf`. Don't forget your changes from step (2)!

(4) Enable and start Nginx with `sysrc nginx_enable=YES` and `service nginx start`. 

(5) Done! The proxy is now online. 

(6) If you have the IPFW running, you'll need to allow at least ports 80/tcp and 443/tcp. You can drop port 80 once you have your certs if required, the proxy runs on 443/tcp. Something like `ipfw -q add allow tcp from any to any 443 setup keep-state` will work. 

The proxy is now live, and you can share it in the normal way using `https://signal.tube/#domain.xyz`. Congratulations! 
