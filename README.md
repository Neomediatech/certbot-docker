# certbot-docker
Dockerized version of certbot, a Let's Encrypt Certification Authority client

## usage
* You should bind mount a host folder for certificates and a folder for acme-challenge verification
* You must have a reverse proxy or other forward method to listen on port 80

### docker-compose.yml example
(Supposing we call this stack "pusher")
```
version: '2'

x-environment: &common-vars
    TZ: Europe/Rome

services:
  redirector:
    image: nginx:1.15-alpine
#    restart: unless-stopped
    hostname: pusher
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - pusher_confs:/etc/nginx/conf.d
      - /srv/data/docker/docker-compose-projects/spiderweb-new/nginx/ssl-dhparams.pem:/data/ssl-dhparams.pem:ro
      - certbot_certs:/data/certs
      - certbot_deals:/var/www/certbot
      - /srv/data/docker/docker-compose-projects/spiderweb-new/nginx/404.html:/var/www/nil/404.html:ro
      - /srv/data/docker/docker-compose-projects/spiderweb-new/nginx/404.css:/var/www/nil/404.css:ro
    environment:
      << : *common-vars
    depends_on:
      - certbot
    command: "/bin/sh -c 'while :; do sleep 22h & wait $${!}; nginx -s reload; done & nginx -g \"daemon off;\"'"
#    healthcheck:
#      test: curl --fail -s http://localhost:5000/ || exit 1
#      interval: 1m30s
#      timeout: 10s
#      retries: 3
    external_links:
      - website_nginx_1
    networks:
      - default
      - website_default

  certbot:
    image: certbot/certbot
    #    restart: unless-stopped
    volumes:
      - certbot_certs:/etc/letsencrypt
      - certbot_deals:/var/www/certbot
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 23h & wait $${!}; done;'"
    environment:
      << : *common-vars

networks:
  website_default:
    external:
      name: website_default

volumes:
  pusher_confs:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /srv/data/redirector/nginx_confs
  certbot_certs:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /srv/data/certbot/conf
  certbot_deals:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /srv/data/certbot/www
```
### Create e certificate
`docker exec -it pusher_certbot_1 certbot certonly -d [domain or FQDN]`
