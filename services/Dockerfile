FROM nginx:latest

ARG SERVER_PORT
ARG HOSTNAME

RUN echo '<html><head><title>NGINX Docker Example</title></head><body>' > /usr/share/nginx/html/index.html \
    && echo "<h1>Hello from NGINX running on port $SERVER_PORT</h1>" >> /usr/share/nginx/html/index.html \
    && echo "<p>Hostname: $HOSTNAME</p>" >> /usr/share/nginx/html/index.html \
    && echo '</body></html>' >> /usr/share/nginx/html/index.html

EXPOSE 80