##Docker Registry 本地搭建

#### 拉取镜像
``` shell
[root@localhost ~]# docker pull registry
Using default tag: latest
latest: Pulling from library/registry
c87736221ed0: Pull complete 
1cc8e0bb44df: Pull complete 
54d33bcb37f5: Pull complete 
e8afc091c171: Pull complete 
b4541f6d3db6: Pull complete 
Digest: sha256:8004747f1e8cd820a148fb7499d71a76d45ff66bac6a29129bfdbfdc0154d146
Status: Downloaded newer image for registry:latest
docker.io/library/registry:latest

```

#### 运行Docker
``` shell
//
mkdir /opt/docker-registry 
cd /opt/docker-registry  
mkdir certs 
mkdir auth
docker run --entrypoint htpasswd registry -Bbn docker docker > auth/htpasswd //docker docker 分别对应 账号 密码

mkdir config
vi /opt/docker-registry/config/config.yml
===================================================
version: 0.1
log:
  fields:
      service: registry
storage:
  filesystem:
    rootdirectory: /var/lib/registry
    maxthreads: 100
  delete:
    enabled: true
  redirect:
    disable: true
  cache:
    blobdescriptor: inmemory
  maintenance:
    uploadpurging:
      enabled: true
      age: 168h
      interval: 24h
      dryrun: false
auth:
  htpasswd:
    realm: basic-realm
    path: /auth/htpasswd
http:
    addr: 0.0.0.0:9527 //此处可以自定义私有仓库的地址端口号
    headers:
        X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
====================================================

docker run --name=docker-registry \
           --restart always\
           -p 9527:9527 \
           -v /opt/docker-registry/conf/config.yml:/etc/docker/registry/config.yml \
           -v /opt/docker-registry/auth/htpasswd:/auth/htpasswd \
           -v /opt/docker-registry/data:/var/lib/registry \
           -d registry:latest

```