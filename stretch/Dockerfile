FROM xutl/debian:stretch

LABEL maintainer="xutongle@gmail.com"

RUN set -x \
#	&& sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
		curl \
		libpcre3 \
		libssl1.1 \
		libbz2-1.0 \
		libcurl3 \
		libxml2 \
		libzip4 \
		libxslt1.1 \
		libgd3 \
	&& rm -r /var/lib/apt/lists/*

ARG NGINX_VERSION

# Environment settings
ENV NGINX_VERSION=${NGINX_VERSION:-2.2.2}
ENV NGINX_TGZ_URL http://tengine.taobao.org/download/tengine-${NGINX_VERSION}.tar.gz
ENV NGINX_INI_DIR=/usr/local/etc/nginx

RUN set -xe \
	&& buildDeps=" \
		make \
		gcc \
		libpcre3-dev \
		libcurl4-openssl-dev \
		libssl-dev \
		libbz2-dev \
		libxml2-dev \
		libzip-dev \
		libxslt-dev \
		libgd-dev \
	" \
	&& apt-get update && apt-get install -y --no-install-recommends --no-install-suggests $buildDeps && rm -r /var/lib/apt/lists/* \
	\
	&& cd /usr/local/src \
	&& curl -fSL ${NGINX_TGZ_URL} -o tengine-${NGINX_VERSION}.tar.gz \
	&& tar zxf tengine-${NGINX_VERSION}.tar.gz \
	&& rm -rf tengine-${NGINX_VERSION}.tar.gz \
	&& cd tengine-${NGINX_VERSION}/ \
	&& ./configure \
		--prefix=/usr/local \
		--conf-path=${NGINX_INI_DIR}/nginx.conf \
		--error-log-path=/var/log/nginx/error.log \
		--http-client-body-temp-path=/var/lib/nginx/body \
		--http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
		--http-log-path=/var/log/nginx/access.log \
		--http-proxy-temp-path=/var/lib/nginx/proxy \
		--lock-path=/run/lock/nginx.lock \
		--pid-path=/run/nginx.pid \
		--with-http_ssl_module \
		--with-http_v2_module \
		--with-http_image_filter_module \
		--with-http_slice_module \
		--with-http_xslt_module \
		--with-http_realip_module \
		--with-http_stub_status_module \
		--with-pcre \
		--with-pcre-jit \
		--with-http_flv_module \
		--with-http_mp4_module \
		--with-http_addition_module \
		--with-threads \
		--with-http_secure_link_module \
		--with-http_degradation_module \
		--with-http_ssl_module \
		--with-http_gzip_static_module \
		--with-http_upstream_check_module \
		--without-mail_imap_module \
		--without-mail_pop3_module \
		--without-mail_smtp_module \
		--without-http_uwsgi_module \
		--without-http_scgi_module \
		--without-select_module \
		--with-http_sub_module \
		--with-cc-opt='-O2' \
	&& make -j "$(nproc)" \
	&& make install \
	&& mkdir -p /var/lib/nginx \
	&& mkdir -p /usr/local/etc/nginx/sites \
	&& rm -f /usr/local/etc/nginx/scgi_* \
	&& rm -rf /usr/local/src/tengine-${NGINX_VERSION} \
	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false $buildDeps

# Add configuration files
COPY image-files/ /

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log && ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 80

STOPSIGNAL SIGTERM

CMD ["nginx", "-c", "/usr/local/etc/nginx/nginx.conf"]