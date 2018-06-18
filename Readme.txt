# REF: https://www.humankode.com/ssl/how-to-set-up-free-ssl-certificates-from-lets-encrypt-using-docker-and-nginx
# NOTES: https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/?utm_source=tool.lu

cd /opt/docker-compose/nginx-letsencrypt

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

docker stack deploy -c docker-compose.yml letsencrypt-starter
docker service ls | grep letsencrypt-starter_nginx
docker service logs -f letsencrypt-starter_nginx

#docker stack rm letsencrypt-starter
#rm -Rf /opt/nginx/

curl http://www.sitesexpress.com/.well-known/acme-challenge/index.html


iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT

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


docker stack rm letsencrypt-starter



cp conf.d/prod.conf  /opt/nginx/conf.d/nginx.conf
chmod a+r /opt/nginx/conf.d/nginx.conf


docker stack deploy -c docker-compose.yml nginx-letsencrypt
#docker stack rm letsencrypt nginx-letsencrypt

docker service ls | grep nginx-letsencrypt_nginx
docker service logs -f nginx-letsencrypt_nginx



