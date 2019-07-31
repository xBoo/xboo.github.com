
#### 启动容器命令：

``` docker
  sudo docker run -d \
  --publish 443:443 \
  --publish 8080:80 \
  --publish 8022:22 \
  --name gitlab \
  --restart always \
  --volume /home/application/gitlab/config:/etc/gitlab \
  --volume /home/application/gitlab/logs:/var/log/gitlab \
  --volume /home/application/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```


#### 运行 Runner

```
    docker run  --name gitlab-runner --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /home/application/gitlab-runner:/etc/gitlab-runner \
    gitlab/gitlab-runner

docker exec -it gitlab-runner gitlab-ci-multi-runner register -n \
  --url http://172.17.78.118:8080 \
  --registration-token gTSbXgX3-RnmrATTum2i \
  --tag-list=dev,uat,prod \
  --description "project_build_runner" \
  --docker-privileged=true \
  --docker-pull-policy="if-not-present" \
  --docker-image mcr.microsoft.com/dotnet/core/sdk \
  --docker-volumes /home/application/gitlab-runner/run/docker.sock:/var/run/docker.sock \
  --executor docker


    "/srv/gitlab-runner/confg:/etc/gitlab-runner",
                "/var/run/docker.sock:/var/run/docker.sock"



docker exec -it gitlab-runner gitlab-runner register -n \
   --url http://172.17.78.118 \
  --registration-token gTSbXgX3-RnmrATTum2i \
   --executor docker \
   --tag-list "01-user-api-builder" \ 
   --description "01-user-api-builder" \
   --docker-volumes /var/run/docker.sock:/var/run/docker.sock
   --docker-image "mcr.microsoft.com/dotnet/core/sdk" \


docker exec -it gitlab-runner gitlab-runner register -n \
   --url http://172.17.78.118:8080 \
  --registration-token gTSbXgX3-RnmrATTum2i \
   --executor docker \
   --tag-list "01-user-api-deploy" \
   --description "01-user-api-deploy" \
   --docker-image "docker:stable" \
   --docker-volumes /var/run/docker.sock:/var/run/docker.sock

```

http://172.17.78.118:8080/root/testproj.git