FROM nginx:latest

RUN rm /etc/nginx/conf.d/default.conf

RUN apt-get update && \
    apt-get install -y certbot python3-certbot-nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*