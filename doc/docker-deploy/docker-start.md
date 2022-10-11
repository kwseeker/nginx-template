# nginx docker 部署



## dockerhub 官方镜像快速部署

```shell
docker pull nginx:1.23.1
docker run --name nginx-temp -p 10080:80 nginx:1.23.1
# 进入容器内部看看nginx文件都在哪里
docker exec -it nginx-temp /bin/bash
find ./ -name nginx
# ./etc/default/nginx
# ./etc/init.d/nginx
# ./etc/logrotate.d/nginx
# ./etc/nginx
# ./usr/share/doc/nginx
# ./usr/share/nginx
# ./usr/sbin/nginx
# ./usr/lib/nginx
# ./var/cache/nginx
# ./var/log/nginx
```

nginx-deploy.sh

```shell
#!/bin/bash

NGINX_IMAGE_VERSION=nginx:1.23.1
INSTALL_DIR=/home/lee/docker
CONTAINER_DIR=nginx
HOST_PORT=10080
HOST_PORT_TLS=10443

cd $INSTALL_DIR || exit
mkdir $CONTAINER_DIR
cd $CONTAINER_DIR || exit

mkdir -p conf/conf.d html logs

baseDir=${INSTALL_DIR}"/"${CONTAINER_DIR}
# 使用“绑定挂载”，如果需要用到容器内部的文件，需要先cp出来
# 	sudo docker cp nginx-temp:/usr/share/nginx/html ./html
# 	sudo docker cp nginx-temp:/etc/nginx/nginx.conf ./conf/nginx.conf
# 	sudo docker cp nginx-temp:/etc/nginx/conf.d conf/conf.d
# 	/var/log/nginx/	 有数据写入就会同步到宿主机
#如果使用“卷挂载”(--mount source=conf.d,target=/etc/nginx/conf.d \)，单个文件(nginx.conf)又没办法与卷关联
docker run -d \
    --name nginx-single \
    -p ${HOST_PORT}:80  \
    -p ${HOST_PORT_TLS}:443  \
    -v ${baseDir}/html:/usr/share/nginx/html \
    -v ${baseDir}/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v ${baseDir}/conf/conf.d:/etc/nginx/conf.d \
    -v ${baseDir}/logs:/var/log/nginx \
    ${NGINX_IMAGE_VERSION}
```

> [Manage data in Docker](https://docs.docker.com/storage/)