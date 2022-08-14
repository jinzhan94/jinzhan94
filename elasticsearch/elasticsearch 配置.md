**下载**

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-linux-x86_64.tar.gz
```

**配置集群**

在ES里面默认有一个配置，cluster.name 默认值就是ElasticSearch,如果这个值是一样的就属于同一个集群，不一样的值就是不一样的集群。

主节点具体配置如下：

```
cluster.name: along-es # 集群的名字
node.name: node-1 # 节点的名字
node.master: true # 是否竞争主节点
network.host: 0.0.0.0 # 匹配本机地址，也可以写真实地址
http.port: 9200
discovery.seed_hosts: ["192.168.8.121","192.168.8.122"] # 参加集群的地址
cluster.initial_master_nodes: ["node-1"] # 设置主节点
```

> 副节点配置根据副节点情况变动。
>
> 注意：0.0.0.0是匹配本机的任意地址，可能出现意料外的地址，但一定确保其和discovery.seed_hosts中 的地址保持一致。否则节点会连接不上。

**error：**Exception in thread “main” java.nio.file.AccessDeniedException:/usr/local/elasticsearch

用户没有该文件夹的权限（用户是`es`），执行命令

```
# chown -R es:es /usr/local/elasticsearch/
```



**error**: elasticsearch：max number of threads [3818] for user [es] is too low, increase to at least [4096]

```
# 修改/etc/security/limits.conf 在末尾添加
*  soft nproc  4096
*  hard nproc  4096
*  soft nofile  65535
*  hard nofile  65535
```

error：failed to obtain node locks, tried [[/data/es/data]] with lock id [0]; maybe these locations are not writable or multiple nodes were started without increasing [node.max_local_storage_nodes] (was [1])?

```
# 1.无权访问/data/es/data
chown -R es:es /data/es/data

# 2.可能已经启动了ES
kill -9 
重启就好了
```

error: [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least 

```
#在/etc/sysctl.conf文件最后添加一行

$ vm.max_map_count=262144

# 执行/sbin/sysctl -p 立即生效
```



解决elasticsearch报错Limit of total fields [1000] in index [xxx]

```
# 临时解决
$ curl -X PUT  -H "Content-Type: application/json" -d '{"index.mapping.total_fields.limit":2000}' http://elasticserver:9200/logstash-2019.10.14/_settings
# 永久解决
$ curl -X PUT  -H "Content-Type: application/json" -d '{"template": "logstash-*","settings":{"index.mapping.total_fields.limit":2000}}' http://elasticserver:9200/_template/logstash
```



ES集群修改index副本数，报错 ：index read-only / allow delete (api)
原因：

es集群数据量增速过快，导致个别es node节点磁盘使用率在%80以上，接近%90 ，由于ES新节点的数据目录data存储空间不足，导致从master主节点接收同步数据的时候失败，此时ES集群为了保护数据，会自动把索引分片index置为只读read-only.

```
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/_all/_settings -d '{"index.blocks.read_only_allow_delete": null}'
```







