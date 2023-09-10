FROM nginx:latest
RUN mkdir /etc/nginx/certificates \
    && mkdir /etc/nginx/log \
    && touch /etc/nginx/log/gw_access.log \
    && touch /etc/nginx/log/gw_error.log \
    && touch /etc/nginx/log/am_access.log \
    && touch /etc/nginx/log/am_error.log 
COPY ./certificates /etc/nginx/certificates

