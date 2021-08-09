FROM centos:7.9.2009 AS workspace

LABEL maintainer="Yuya Kawakami <yuyarin@yuyarin.net>"

WORKDIR /tmp

RUN yum update -y && \
    yum install -y epel-release centos-release-scl && \
    yum install -y curl mercurial pcre-devel zlib-devel libunwind-devel golang devtoolset-10 python3-devel python3-pip && \
    scl enable devtoolset-10 sh && \
    scl enable devtoolset-10 bash && \
    pip3 install -U pip && \
    pip3 install mercurial

ENV BORINGSSL_BRANCH=master
ENV PATH=/opt/rh/devtoolset-10/root/usr/bin/:$PATH

# boringssl requires CMake > 3.5
RUN mkdir cmake && \
    cd cmake && \
    curl -O https://cmake.org/files/v3.6/cmake-3.6.2.tar.gz && \
    tar xzvf cmake-3.6.2.tar.gz && \
    cd cmake-3.6.2 && \
    ./bootstrap --prefix=/usr && \
    make && \
    make install && \
    cd ../../

RUN mkdir boringssl && \
    cd boringssl && \
    curl -O https://boringssl.googlesource.com/boringssl/+archive/refs/heads/$BORINGSSL_BRANCH.tar.gz && \
    tar xzvf $BORINGSSL_BRANCH.tar.gz && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_POSITION_INDEPENDENT_CODE=ON .. && \
    make && \
    cd ../..

RUN hg clone -b quic https://hg.nginx.org/nginx-quic && \
    cd nginx-quic && \
    ./auto/configure \
        --prefix=/etc/nginx \
        --sbin-path=/usr/sbin/nginx \
        --modules-path=/usr/lib64/nginx/modules \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --pid-path=/var/run/nginx.pid \
        --lock-path=/var/run/nginx.lock \
        --http-client-body-temp-path=/var/cache/nginx/client_temp \
        --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
        --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
        --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
        --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
        --user=nginx \
        --group=nginx \
        --with-compat \
        --with-file-aio \
        --with-threads \
        --with-http_addition_module \
        --with-http_auth_request_module \
        --with-http_dav_module \
        --with-http_flv_module \
        --with-http_gunzip_module \
        --with-http_gzip_static_module \
        --with-http_mp4_module \
        --with-http_random_index_module \
        --with-http_realip_module \
        --with-http_secure_link_module \
        --with-http_slice_module \
        --with-http_ssl_module \
        --with-http_stub_status_module \
        --with-http_sub_module \
        --with-http_v2_module \
        --with-mail \
        --with-mail_ssl_module \
        --with-stream \
        --with-stream_realip_module \
        --with-stream_ssl_module \
        --with-stream_ssl_preread_module \
        --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC -I../boringssl/include' \
        --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie -L../boringssl/build/ssl -L../boringssl/build/crypto' \
        --with-stream_quic_module \
        --with-debug \
        --with-http_v3_module && \
    make && \
    make install

FROM centos:7.9.2009
COPY --from=workspace /usr/sbin/nginx /usr/sbin/
COPY --from=workspace /etc/nginx/ /etc/nginx/

RUN groupadd -g 993 nginx && \
    useradd -m -u 995 -s /sbin/nologin -g nginx nginx && \
    mkdir -p /var/log/nginx && \
    touch /var/log/nginx/access.log /var/log/nginx/error.log && \
    chown -R nginx: /var/log/nginx/ && \
    ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log && \
    mkdir /var/cache/nginx && \
    chown -R nginx: /var/cache/nginx

EXPOSE 80
EXPOSE 443/tcp
EXPOSE 443/udp

STOPSIGNAL SIGQUIT

CMD ["nginx", "-g", "daemon off;"]
