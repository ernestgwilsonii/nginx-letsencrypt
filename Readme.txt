# REF: https://www.humankode.com/ssl/how-to-set-up-free-ssl-certificates-from-lets-encrypt-using-docker-and-nginx
# NOTES: https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/?utm_source=tool.lu

# Git clone or git pull
cd /opt/docker-compose/nginx-letsencrypt
# git clone https://github.com/ernestgwilsonii/nginx-letsencrypt.git
git pull

# Create bind mount directories for Docker
mkdir -p /opt/nginx
mkdir -p /opt/nginx/conf.d
mkdir -p /opt/nginx/letsencrypt
mkdir -p /opt/nginx/letsencrypt/.well-known
mkdir -p /opt/nginx/letsencrypt/.well-known/acme-challenge
mkdir -p /opt/nginx/ssl
mkdir -p /opt/nginx/ssl/lib
mkdir -p /opt/nginx/ssl/log
cp letsencrypt/.well-known/acme-challenge/index.html /opt/nginx/letsencrypt/.well-known/acme-challenge/index.html
cp conf.d/letsencrypt-starter.conf  /opt/nginx/conf.d/nginx.conf
chmod a+r /opt/nginx/conf.d/nginx.conf
chmod -R 755 /opt/nginx/letsencrypt

# Deploy "TCP 80 HTTP" starter
docker stack deploy -c docker-compose.yml letsencrypt-starter

# Verify Docker "starter"
docker service ls | grep letsencrypt-starter_nginx
docker service logs -f letsencrypt-starter_nginx

#docker stack rm letsencrypt-starter
#rm -Rf /opt/nginx/

# Verify
curl http://www.sitesexpress.com/.well-known/acme-challenge/index.html

# Allow inbound ports
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT


# Pull down a "staging" SSL cert kit  (to make sure DNS and the kit work)
docker run -it --rm \
-v /opt/nginx/letsencrypt:/data/letsencrypt \
-v /opt/nginx/ssl:/etc/letsencrypt \
-v /opt/nginx/ssl/lib:/var/lib/letsencrypt \
-v /opt/nginx/ssl/log:/var/log/letsencrypt \
certbot/certbot certonly --webroot \
--agree-tos \
--email ernestgwilsonii\@gmail.com \
--eff-email \
--webroot-path=/data/letsencrypt \
--staging \
-d sitesexpress.com -d www.sitesexpress.com -d mqtt.sitesexpress.com -d nodered.sitesexpress.com

# It "staging" works and downloads "staging SSL certs" without errors (confirming DNS and config), move next:
# Stop the TCP 80 only "starter" kit
docker stack rm letsencrypt-starter


# Copy in (overwrite) with a "PROD" NGING config kit
cp conf.d/prod.conf  /opt/nginx/conf.d/nginx.conf
chmod a+r /opt/nginx/conf.d/nginx.conf

# Start NGING with a valid NGINX config for "PROD" that uses the "staging" SSL certs we got above
docker stack deploy -c docker-compose.yml nginx-letsencrypt
#docker stack rm nginx-letsencrypt

# Verify Docker
docker service ls | grep nginx-letsencrypt_nginx
docker service logs -f nginx-letsencrypt_nginx

# Removing the "staging" SSL certs
rm -Rf /opt/nginx/ssl

# Recreate directory structure to allow receiving "prod" (real) SSL certs
mkdir -p /opt/nginx/ssl
mkdir -p /opt/nginx/ssl/lib
mkdir -p /opt/nginx/ssl/log


# Download  "prod" (real) SSL certs (no longer "staging")
docker run -it --rm \
-v /opt/nginx/letsencrypt:/data/letsencrypt \
-v /opt/nginx/ssl:/etc/letsencrypt \
-v /opt/nginx/ssl/lib:/var/lib/letsencrypt \
-v /opt/nginx/ssl/log:/var/log/letsencrypt \
certbot/certbot certonly --webroot \
--agree-tos \
--email ernestgwilsonii\@gmail.com \
--eff-email \
--webroot-path=/data/letsencrypt \
-d sitesexpress.com -d www.sitesexpress.com -d mqtt.sitesexpress.com -d nodered.sitesexpress.com


# Restart "PROD" NGINX config that uses "prod" (real) SSL certs
docker stack rm nginx-letsencrypt
docker stack deploy -c docker-compose.yml nginx-letsencrypt

# Verify Docker
docker service ls | grep nginx-letsencrypt_nginx
docker service logs -f nginx-letsencrypt_nginx

# Visit site and verify real SSL cert
https://sitesexpress.com/.well-known/acme-challenge/index.html


