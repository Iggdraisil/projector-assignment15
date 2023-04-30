FROM debian:stable

WORKDIR /tmp

# Install prerequisites for Nginx compile
RUN apt-get update -y
RUN apt-get install -y build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev libgd-dev libxml2 libxml2-dev uuid-dev git patch
RUN apt-get install -y wget libgeoip-dev  libcap2-bin
#RUN yum install -y wget tar openssl-devel gcc gcc-c++ make zlib-devel pcre-devel gd-devel krb5-devel openldap-devel  git
#RUN yum install -y patch
#RUN yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm 
#RUN yum install -y mod_geoip GeoIP GeoIP-devel GeoIP-data zlib-devel


# Download Nginx and Nginx modules source
RUN wget http://nginx.org/download/nginx-1.24.0.tar.gz -O nginx.tar.gz && \
    mkdir /tmp/nginx && \
    tar -xzvf nginx.tar.gz -C /tmp/nginx --strip-components=1
RUN git clone https://github.com/yaoweibin/nginx_upstream_check_module.git /tmp/nginx_http_upstream_check_module


WORKDIR /tmp/nginx
RUN patch -p1 < /tmp/nginx_http_upstream_check_module/check_1.20.1+.patch

# Build Nginx
RUN ./configure \
        --user=nginx \
        --with-debug \
        --group=nginx \
        --prefix=/usr/share/nginx \
        --sbin-path=/usr/sbin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --pid-path=/run/nginx.pid \
        --lock-path=/run/lock/subsys/nginx \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
	--with-stream \
	--with-stream_realip_module \
        --with-stream_ssl_module \
	--with-stream_ssl_preread_module \
 #       --with-http_spdy_module \
        --with-pcre \
	--with-threads \
        --with-file-aio \
        --with-ipv6 \
	--with-http_addition_module \
        --with-http_ssl_module \
        --with-http_gzip_static_module \
	--with-http_slice_module \
	--with-http_sub_module \
	--with-http_v2_module \
        --with-http_image_filter_module \
        --with-http_stub_status_module \
        --with-http_dav_module \
        --with-http_geoip_module \
        --with-http_flv_module \
        --with-http_mp4_module \
	#--with-http_upstream_module \
        --with-http_gunzip_module \
        --add-module=/tmp/nginx_http_upstream_check_module \
	--with-http_realip_module 
RUN make 
RUN make install

WORKDIR /tmp

# Add nginx user
RUN useradd -c "Nginx user" nginx && \
    setcap cap_net_bind_service=ep /usr/sbin/nginx

RUN touch /run/nginx.pid

RUN chown nginx:nginx /etc/nginx /etc/nginx/nginx.conf /var/log/nginx /usr/share/nginx /run/nginx.pid

RUN cd /var/lib &&     mkdir -p nginx &&     wget -q -O- https://dl.miyuru.lk/geoip/maxmind/country/maxmind.dat.gz | gunzip -c > nginx/maxmind-country.dat &&     wget -q -O- https://dl.miyuru.lk/geoip/maxmind/city/maxmind.dat.gz | gunzip -c > nginx/maxmind-city.dat &&     chown -R nginx. nginx
# Cleanup after Nginx build
RUN rm -rf /tmp/*

# PORTS
EXPOSE 80
EXPOSE 443

USER nginx
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
COPY docker-entrypoint.sh /
COPY etc /etc/nginx
COPY docker-entrypoint.d /docker-entrypoint.d
ENTRYPOINT ["/docker-entrypoint.sh"]
