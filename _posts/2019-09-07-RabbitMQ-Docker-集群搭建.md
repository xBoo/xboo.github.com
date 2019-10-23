## RabbitMQ-Docker-集群搭建


cd /data/rabbitmq
sudo vi hosts
#输入
10.10.168.6 rabbit rabbitmq1
10.10.94.142 rabbit rabbitmq2

docker run -d \
--net=host \
--restart=always \
--name=rabbitmq \
--hostname=rabbitmq1 \
--log-opt=max-size=1024m \
--log-opt=max-file=3 \
-v /data/rabbitmq:/var/lib/rabbitmq:z \
-v /data/rabbitmq/hosts:/etc/hosts \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
-e RABBITMQ_ERLANG_COOKIE='rabbitmq cookie' rabbitmq:3.7.16-management


//云服务器部署
***
## Log
10.10.66.135 rabbit rabbitmq1
127.0.0.1 rabbit rabbitmq2

docker rm -f rabbitmq-log

docker run -d \
-p 5673:5673 \
-p 15673:15673 \
-p 25673:25673 \
-p 4370:4370 \
--name=rabbitmq-log \
--hostname=rabbitmq1 \ //此处跟host相同
--log-opt=max-size=2048m \
--log-opt=max-file=3 \
-v /opt/rabbitmqlog:/var/lib/rabbitmq:z \
-v /opt/rabbitmqlog/hosts:/etc/hosts \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
-e RAM_NODE=true \
-e ERL_EPMD_PORT=4370 \
-e RABBITMQ_NODE_PORT=5673 \
-e RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]" \
-e RABBITMQ_ERLANG_COOKIE='rabbitmq cookie' \
rabbitmq:3.7.17-management


docker run -d \
-p 5673:5673 \
-p 15673:15673 \
-p 25673:25673 \
-p 4370:4370 \
--name=rabbitmq-log \
--hostname=rabbitmq2 \ //此处跟host相同
--log-opt=max-size=2048m \
--log-opt=max-file=3 \
-v /opt/rabbitmqlog:/var/lib/rabbitmq:z \
-v /opt/rabbitmqlog/hosts:/etc/hosts \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
-e RAM_NODE=true \
-e ERL_EPMD_PORT=4370 \
-e RABBITMQ_NODE_PORT=5673 \
-e RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]" \
-e RABBITMQ_ERLANG_COOKIE='rabbitmq cookie' \
rabbitmq:3.7.17-management
***
##EVOS
10.10.66.135 rabbit rabbitmq1
127.0.0.1 rabbit rabbitmq2

docker rm -f rabbitmq-evos

docker run -d \
-p 5672:5672 \
-p 15672:15672 \
-p 25672:25672 \
-p 4369:4369 \
--name=rabbitmq-evos \
--hostname=rabbitmq1 \
--log-opt=max-size=2048m \
--log-opt=max-file=3 \
-v /opt/rabbitmqevos:/var/lib/rabbitmq:z \
-v /opt/rabbitmqevos/hosts:/etc/hosts \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
-e RABBITMQ_ERLANG_COOKIE='rabbitmq cookie' \
rabbitmq:3.7.17-management


docker run -d \
-p 5672:5672 \
-p 15672:15672 \
-p 25672:25672 \
-p 4369:4369 \
--name=rabbitmq-evos \
--hostname=rabbitmq2 \
--log-opt=max-size=2048m \
--log-opt=max-file=3 \
-v /opt/rabbitmqevos:/var/lib/rabbitmq:z \
-v /opt/rabbitmqevos/hosts:/etc/hosts \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
-e RAM_NODE=true \
-e RABBITMQ_ERLANG_COOKIE='rabbitmq cookie' \
rabbitmq:3.7.17-management

***

加入RabbitMQ节点到集群
设置节点1：

docker exec -it myrabbit1 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
exit

设置节点2，加入到集群：

docker exec -it myrabbit2 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbit@rabbitmq1
rabbitmqctl start_app
exit

参数“--ram”表示设置为内存节点，忽略次参数默认为磁盘节点。