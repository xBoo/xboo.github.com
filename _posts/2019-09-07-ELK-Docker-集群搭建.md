## ELK-Docker-集群搭建


#### ElasticSearch 集群搭建


mkdir /opt/elasticsearch
mkdir /opt/elasticsearch/config
mkdir /opt/elasticsearch/data
chmod 777  /opt/elasticsearch/data/

vi /opt/elasticsearch/config/elasticsearch.yml
```
cluster.name: elasticsearch-cluster
node.name: es-node102
network.bind_host: 0.0.0.0
network.publish_host: 192.168.10.102
http.port: 9200
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true
node.data: true
discovery.zen.ping.unicast.hosts: ["192.168.10.101:9300","192.168.10.102:9300"]
discovery.zen.minimum_master_nodes: 1
```
vi /opt/elasticsearch/config/jvm.options
主要目的是为了修改-Xms -Xmx 值，必须与docker 容器run 参数保持一直
```
## JVM configuration

################################################################
## IMPORTANT: JVM heap size
################################################################
##
## You should always set the min and max JVM heap
## size to the same value. For example, to set
## the heap to 4 GB, set:
##
## -Xms4g
## -Xmx4g
##
## See https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html
## for more information
##
################################################################

# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms512m
-Xmx1g

################################################################
## Expert settings
################################################################
##
## All settings below this section are considered
## expert settings. Don't tamper with them unless
## you understand what you are doing
##
################################################################

## GC configuration
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly

## G1GC Configuration
# NOTE: G1GC is only supported on JDK version 10 or later.
# To use G1GC uncomment the lines below.
# 10-:-XX:-UseConcMarkSweepGC
# 10-:-XX:-UseCMSInitiatingOccupancyOnly
# 10-:-XX:+UseG1GC
# 10-:-XX:InitiatingHeapOccupancyPercent=75

## DNS cache policy
# cache ttl in seconds for positive DNS lookups noting that this overrides the
# JDK security property networkaddress.cache.ttl; set to -1 to cache forever
-Des.networkaddress.cache.ttl=60
# cache ttl in seconds for negative DNS lookups noting that this overrides the
# JDK security property networkaddress.cache.negative ttl; set to -1 to cache
# forever
-Des.networkaddress.cache.negative.ttl=10

## optimizations

# pre-touch memory pages used by the JVM during initialization
-XX:+AlwaysPreTouch

## basic

# explicitly set the stack size
-Xss1m

# set to headless, just in case
-Djava.awt.headless=true

# ensure UTF-8 encoding by default (e.g. filenames)
-Dfile.encoding=UTF-8

# use our provided JNA always versus the system one
-Djna.nosys=true

# turn off a JDK optimization that throws away stack traces for common
# exceptions because stack traces are important for debugging
-XX:-OmitStackTraceInFastThrow

# flags to configure Netty
-Dio.netty.noUnsafe=true
-Dio.netty.noKeySetOptimization=true
-Dio.netty.recycler.maxCapacityPerThread=0

# log4j 2
-Dlog4j.shutdownHookEnabled=false
-Dlog4j2.disable.jmx=true

-Djava.io.tmpdir=${ES_TMPDIR}

## heap dumps

# generate a heap dump when an allocation from the Java heap fails
# heap dumps are created in the working directory of the JVM
-XX:+HeapDumpOnOutOfMemoryError

# specify an alternative path for heap dumps; ensure the directory exists and
# has sufficient space
-XX:HeapDumpPath=data

# specify an alternative path for JVM fatal error logs
-XX:ErrorFile=logs/hs_err_pid%p.log

## JDK 8 GC logging

8:-XX:+PrintGCDetails
8:-XX:+PrintGCDateStamps
8:-XX:+PrintTenuringDistribution
8:-XX:+PrintGCApplicationStoppedTime
8:-Xloggc:logs/gc.log
8:-XX:+UseGCLogFileRotation
8:-XX:NumberOfGCLogFiles=32
8:-XX:GCLogFileSize=64m

# JDK 9+ GC logging
9-:-Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m
# due to internationalization enhancements in JDK 9 Elasticsearch need to set the provider to COMPAT otherwise
# time/date parsing will break in an incompatible way for some date patterns and locals
9-:-Djava.locale.providers=COMPAT

# temporary workaround for C2 bug with JDK 10 on hardware with AVX-512
10-:-XX:UseAVX=2
```

run docker：
```
docker rm -f elasticsearch
rm /opt/elasticsearch/data/* -rf
docker run -tid \
 -e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
 -p 9200:9200 \
 -p 9300:9300 \
 -v /opt/elasticsearch/config/jvm.options:/usr/share/elasticsearch/config/jvm.options \
 -v /opt/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
 -v /opt/elasticsearch/data:/usr/share/elasticsearch/data \
 --name elasticsearch \
 elasticsearch:6.7.1
```
#### LogStash 

cd /opt/logstash & mkdir config & mkdir pipeline & mkdir data
chmod 777 data
vi /opt/logstash/config/logstash.yml
```
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.url: http://192.168.10.102:9200

```

vi /opt/logstash/pipeline/logstash.conf
rabbitmq
```
input {
      rabbitmq  {
        host => "192.168.10.102"
        port => 5672
        vhost => "log"
        exchange => "logexchange"
        exchange_type => "fanout"
        queue => "logqueue"
        heartbeat => 30
        ack => true
        user => "admin"
        durable => true
        password => "admin"
        codec => "json"
  }
}

output {
  stdout {
    codec => rubydebug
  }
}


output {
  elasticsearch {
         hosts => "192.168.10.102:9200"
        index => "log-%{+YYYY.MM.dd}"
        timeout => 300
  }
}

```
kafka
```
input{
      kafka{
        bootstrap_servers => ["192.168.10.101:9092,192.168.10.102:9092"]
        client_id => "192.168.10.101"
        group_id => "testgroupid"
        auto_offset_reset => "latest"
        consumer_threads => 1
        decorate_events => "true"
        topics => ["logtopic"]
        codec => "json"
      }
}
filter {
    mutate {
        convert => ["UseTime", "float"]
        gsub => ["EndTime", " ", "T"]
        gsub => ["StartTime", " ", "T"]
        gsub => ["LogTime", " ", "T"]
    }
}
output{
        elasticsearch{
               hosts => ["192.168.10.101:9200","192.168.10.102:9200"]
               index => "gaea-%{+YYYY.MM.dd}"
               document_type => "logs"  //此项6.x版本建议不配置
               timeout => 300
          }
}

```


docker rm -f logstash
docker run --rm  \
-v /opt/logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml \
-v /opt/logstash/pipeline/logstash.conf:/usr/share/logstash/pipeline/logstash.conf \
-v /opt/logstash/data/:/usr/share/logstash/data/ \
--name=logstash \
logstash:6.5.4


#### Kabana
docker run  -tid \
--name kibana \
--link elasticsearch:elasticsearch \
-p 5601:5601 \
kibana:6.7.1




