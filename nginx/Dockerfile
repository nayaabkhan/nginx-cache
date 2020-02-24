FROM openresty/openresty:buster-fat

COPY nginx.conf /etc/nginx/conf.d/nginx.conf
RUN rm /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["/usr/local/openresty/bin/openresty", "-g", "daemon off;"]
