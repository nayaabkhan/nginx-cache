FROM nginx:alpine AS builder

# ENV NGINX_VERSION is provided by nginx image
ENV HTTP_REDIS_VERSION 0.3.9
ENV ECHO_VERSION 0.61
ENV REDIS2_VERSION 0.15
ENV SRCACHE_VERSION 0.31
ENV MISC_VERSION 0.32
ENV DEVELKIT_VERSION 0.3.1

RUN wget "http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" -O nginx.tar.gz
RUN wget "https://people.freebsd.org/~osa/ngx_http_redis-${HTTP_REDIS_VERSION}.tar.gz" -O httpredis.tar.gz
RUN wget "https://github.com/openresty/echo-nginx-module/archive/v${ECHO_VERSION}.tar.gz" -O echo.tar.gz
RUN wget "https://github.com/openresty/redis2-nginx-module/archive/v${REDIS2_VERSION}.tar.gz" -O redis2.tar.gz
RUN wget "https://github.com/openresty/srcache-nginx-module/archive/v${SRCACHE_VERSION}.tar.gz" -O srcache.tar.gz
RUN wget "https://github.com/vision5/ngx_devel_kit/archive/v${DEVELKIT_VERSION}.tar.gz" -O develkit.tar.gz
RUN wget "https://github.com/openresty/set-misc-nginx-module/archive/v${MISC_VERSION}.tar.gz" -O misc.tar.gz

RUN apk add --no-cache --virtual .build-deps \
  gcc \
  libc-dev \
  make \
  openssl-dev \
  pcre-dev \
  zlib-dev \
  linux-headers \
  curl \
  gnupg \
  libxslt-dev \
  gd-dev \
  geoip-dev

ENV APP_HOME /usr/src
RUN mkdir $APP_HOME

RUN CONFARGS=$(nginx -V 2>&1 | sed -n -e 's/^.*arguments: //p') \
  tar -zxC $APP_HOME/ -f nginx.tar.gz && \
  tar -xzvf "httpredis.tar.gz" && \
  tar -xzvf "echo.tar.gz" && \
  tar -xzvf "redis2.tar.gz" && \
  tar -xzvf "srcache.tar.gz" && \
  tar -xzvf "develkit.tar.gz" && \
  tar -xzvf "misc.tar.gz" && \
  REDISDIR="$(pwd)/ngx_http_redis-${HTTP_REDIS_VERSION}" && \
  ECHODIR="$(pwd)/echo-nginx-module-${ECHO_VERSION}" && \
  REDIS2DIR="$(pwd)/redis2-nginx-module-${REDIS2_VERSION}" && \
  SRCACHEDIR="$(pwd)/srcache-nginx-module-${SRCACHE_VERSION}" && \
  DEVELKITDIR="$(pwd)/ngx_devel_kit-${DEVELKIT_VERSION}" && \
  MISCDIR="$(pwd)/set-misc-nginx-module-${MISC_VERSION}" && \
  cd $APP_HOME/nginx-$NGINX_VERSION && \
  ./configure --with-compat $CONFARGS --add-dynamic-module=$REDISDIR --add-dynamic-module=$ECHODIR --add-dynamic-module=$REDIS2DIR --add-dynamic-module=$SRCACHEDIR --add-dynamic-module=$DEVELKITDIR --add-dynamic-module=$MISCDIR && \
  make && make install

FROM nginx:alpine

COPY --from=builder /usr/local/nginx/modules/ngx_http_redis_module.so /usr/local/nginx/modules/ngx_http_redis_module.so
COPY --from=builder /usr/local/nginx/modules/ngx_http_echo_module.so /usr/local/nginx/modules/ngx_http_echo_module.so
COPY --from=builder /usr/local/nginx/modules/ngx_http_redis2_module.so /usr/local/nginx/modules/ngx_http_redis2_module.so
COPY --from=builder /usr/local/nginx/modules/ngx_http_srcache_filter_module.so /usr/local/nginx/modules/ngx_http_srcache_filter_module.so
COPY --from=builder /usr/local/nginx/modules/ndk_http_module.so /usr/local/nginx/modules/ndk_http_module.so
COPY --from=builder /usr/local/nginx/modules/ngx_http_set_misc_module.so /usr/local/nginx/modules/ngx_http_set_misc_module.so
RUN rm /etc/nginx/conf.d/default.conf

COPY nginx.conf /etc/nginx/nginx.conf

RUN mkdir -p /tmp/nginx/cache

EXPOSE 80
STOPSIGNAL SIGTERM
CMD ["nginx"]
