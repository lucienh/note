# 启动redis集群

`redis`按照时需要依赖很多项，这里把安装遇到的缺失包按照命令给出来


``` shell
yum install -y gcc 
yum install -y zlib
yum install -y ruby
yum install -y rubygems
gem install redis
wget http://download.redis.io/releases/redis-3.2.3.tar.gz
tar xzf redis-3.2.3.tar.gz
cd redis-3.2.3
make MALLOC=libc  && make test
```


``` shell

#!/bin/sh
## start redis clustere 
## author liaokailin

## redis command location

redis_location="/data/software/redis/redis-3.2.3/src"
redis_command="${redis_location}/redis-server"

## redis cluster config location

redis_conf_location="/data/software/redis/redis-3.2.3/redis-conf"


ip=`ifconfig | grep 'inet addr'  | cut -f2 -d ":" | cut -f1 -d " " | grep -v 127.0.0.1`
#ip=`/sbin/ifconfig -a | grep inet | grep -v 127.0.0.1 | grep -v inet6 | awk '{print $2}' | tr -d "addr:"`

## kill all redis process
ps -ef | grep redis-[server] | awk '{print $2}' | xargs kill -9

echo ${ip}


for i in {7000..7005};do
  echo "start node-${i},port is ${i}. " 
  ${redis_command} ${redis_conf_location}/${i}/redis.conf --protected-mode no &
  ips="${ips}${ip}:${i} "
done

  echo "all nodes started." 

  echo "nodes join cluster."


 ${redis_location}/redis-trib.rb  create --replicas 1 ${ips}

```
