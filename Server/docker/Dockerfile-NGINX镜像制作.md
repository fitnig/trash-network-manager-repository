# Dockerfile-NGINX镜像制作

## 2020.07.15

## 1 NGINX镜像制作:

### 1.1 NGINX-dockerfile

`FROM centos:7`

`LABEL maintainer www.chenleilei.net`

`RUN useradd  www -u 1200 -M -s /sbin/nologin`

`RUN mkdir -p /var/log/nginx`

`RUN yum install -y cmake pcre pcre-devel openssl openssl-devel gd-devel \`

  `zlib-devel gcc gcc-c++ net-tools iproute telnet wget curl &&\`

  `yum clean all && \`

  `rm -rf /var/cache/yum/*`

`RUN wget https://www.chenleilei.net/soft/nginx-1.16.1.tar.gz`

`RUN tar xf nginx-1.16.1.tar.gz`

`WORKDIR nginx-1.16.1`

`RUN ./configure --prefix=/usr/local/nginx --with-http_image_filter_module --user=www --group=www \`

  `--with-http_ssl_module --with-http_v2_module --with-http_stub_status_module \`

  `--error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log \`

  `--pid-path=/var/run/nginx/nginx.pid`

`RUN make -j 4 && make install && \`

  `rm -rf /usr/local/nginx/html/*  && \`

  `echo "leilei hello" >/usr/local/nginx/html/index.html  && \`

  `rm -rf nginx* && \`

  `ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\`

  `ln -sf /dev/stdout /var/log/nginx/access.log && \`

  `ln -sf /dev/stderr /var/log/nginx/error.log`

`RUN chown -R www.www /var/log/nginx`

`ENV LOG_DIR /var/log/nginx`

`ENV PATH $PATH:/usr/local/nginx/sbin`

`\#COPY nginx.conf /usr/local/nginx/conf/nginx.conf`

`EXPOSE 80`

`WORKDIR /usr/local/nginx`

`CMD ["nginx","-g","daemon off;"]`

### 1.2 启动nginx

 保存为: dockerfile-nginx

 `docker build -t nginx-v001 -f nginx-dockerfile .`

 `useradd  www -u 1200 -M -s /sbin/nologin`

 `mkdir /www -p`

 `docker run --name nginx-v001 -d -p 80:80 -v /www:/usr/local/nginx/html -v /var/log/nginx:/var/log/nginx --restart=always --privileged=true  nginx-v001`

'启动时将目录挂载到本地,以供php程序进行调用解析.' 

 

 

### 1.3 可选: (配置PHP解析)

 进入nginx配置文件修改php解析:

 `/usr/local/nginx/conf/nginx.conf`

`` 

 `location ~ \.php$ {`

  `root      html;`

  `fastcgi_pass  172.17.0.2:9000;`

  `fastcgi_index  index.php;`

  `fastcgi_param  SCRIPT_FILENAME  /usr/local/nginx/html$fastcgi_script_name;`

  `include     fastcgi_params;`

`}`

`` 

\#如何获取php容器IP地址:

 `docker inspect php-v001|grep IPAdd`

 `\--------------------------------------------------------`

 `[root@master docker-file]# docker inspect php-v001|grep IPAdd`

​      `"SecondaryIPAddresses": null,`

​      `"IPAddress": "172.17.0.2",`

​          `"IPAddress": "172.17.0.2"`

​       

 从这里可以看到php服务的容器地址: 172.17.0.2  ,将nginx配置文件种的php解析修改为:172.17.0.2 

 网页目录默认为: /usr/local/nginx/html

 

 配置完成重启nginx.