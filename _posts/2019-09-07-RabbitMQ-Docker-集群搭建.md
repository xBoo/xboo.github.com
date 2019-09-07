## RabbitMQ-Docker-集群搭建


cd /opt/rabbitmq
sudo vi hosts
#输入
192.168.10.101 rabbit rabbitmq1
192.168.10.102 rabbit rabbitmq2

docker run -d \
--net=host \
--name=rabbitmq \
--hostname=rabbit2 \  //必须跟机器 hostname 相同，即机器1 为 rabbit1，机器2为 rabbitmq2
--log-opt=max-size=1024m \
--log-opt=max-file=3 \
-v /opt/rabbitmq:/var/lib/rabbitmq:z \
-v /opt/rabbitmq/hosts:/etc/hosts \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
-e RABBITMQ_ERLANG_COOKIE='rabbitmq cookie' rabbitmq:3.7.16-management






