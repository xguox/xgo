---
title: "本地用 Vagrant 搭建 Elasticsearch 集群"
date: 2016-09-27T16:01:23+08:00
draft: false
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
---

**Vagrant** 在 Ruby/Rails 社区老早之前就听的挺多的, 但今天才开始尝试折腾.

当前下载的版本是 1.8.5, 虚拟机用的是 VirtualBox 5.1.

一开始尝试用 **'hashicorp/precise32'** 这个 box, 但是, 在设置 private_network 的时候貌似有 bug,

```ruby
Vagrant attempted to execute the capability 'configure_networks'
on the detect guest OS 'linux', but the guest doesn't
support that capability. This capability is required for your
configuration of Vagrant. Please either reconfigure Vagrant to
avoid this capability or fix the issue by creating the capability.
```

只好换一个 box, 选的是 **'ubuntu/xenial64'**, 但, `vagrant up` 装了半天, 那速度简直了.  最后只好自己单独去下载 box,

[https://atlas.hashicorp.com/ubuntu/boxes/xenial64/versions/20170331.0.0/providers/virtualbox.box](https://atlas.hashicorp.com/ubuntu/boxes/xenial64/versions/20170331.0.0/providers/virtualbox.box)

自己在家开下载软件, 几 M 每秒杠杠的, 要是靠慢慢 up 的话我就可以洗洗睡了.

```ruby
vagrant box add xenial64 ~/Downloads/virtualbox.box
vagrant init xenial64
```

完事以后, 如果只是试玩一下 **Vagrant** 的话可以直接  `vagrant up`.

这里因为准备要建三个虚拟机.

所以, 改了一下 **Vagrantfile**, 前面就是因为设置 `es.vm.network :private_network, ip: '192.168.10.111'` 所以换了这个 box.

```ruby
Vagrant.configure('2') do |config|
  config.vm.define 'es1' do |es|
    es.vm.box = 'xenial64'

    es.vm.network :private_network, ip: '192.168.10.111'

    es.vm.provider :virtualbox do |v|
      v.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
      v.customize ['modifyvm', :id, '--memory', 1024]
      v.customize ['modifyvm', :id, '--name', 'es1']
    end
  end

  config.vm.define 'es2' do |es|
    es.vm.box = 'xenial64'

    es.vm.network :private_network, ip: '192.168.10.112'

    es.vm.provider :virtualbox do |v|
      v.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
      v.customize ['modifyvm', :id, '--memory', 1024]
      v.customize ['modifyvm', :id, '--name', 'es2']
    end
  end

  config.vm.define 'es3' do |es3|
    es3.vm.box = 'xenial64'

    es3.vm.network :private_network, ip: '192.168.10.113'

    es3.vm.provider :virtualbox do |v|
      v.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
      v.customize ['modifyvm', :id, '--memory', 1024]
      v.customize ['modifyvm', :id, '--name', 'es32']
    end
  end
end
```

```ruby
vagrant up
```

这里三台虚拟机分别是, 'es1', 'es2', 'es3', 每个虚拟机分的内存是 1G. 后面要在虚拟机上安装的流程理论上是可以写进一个 shell 脚本, 让 Vagrant 在 up 的时候执行的, 不过我还是自己一台一台跑了. 主要是, 网络问题, 安装 Java 的时候特别蛋疼吧.
up 了以后, `vagrant ssh es1(2,3)` 分别执行下面这堆. (嫌麻烦, 网络好的可以写成 shell 脚本, 不用一直重复)

```ruby
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update

# 这里一会快一会慢. 也花了不少时间, 网络问题

sudo apt-get install oracle-java8-installer
# java -version
# java version "1.8.0_101"
# Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
# Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)

wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-2.3.5.deb

sudo dpkg -i elasticsearch-2.3.5.deb

sudo update-rc.d elasticsearch defaults 95 10
```

然之后是配置文件,
一开始时候, 三个虚拟机分别是三个 nodes, 一个 client node, 一个 master node, 一个 data node, 后来索性把自己的 host 机也搞上了. 我自己本机的 ip 用的是 `192.168.10.1`

先把自己这个 host机旧的 `elasticsearch.yml` 备份, 再开始配置集群.

```ruby
cp elasticsearch.yml elasticsearch.yml.bak
```

```yml
################################### Cluster ###################################

cluster.name: my-application

#################################### Node #####################################

node.name: es-client
node.client: true
node.data: false

# You can exploit these settings to design advanced cluster topologies.
#
# 1. You want this node to never become a master node, only to hold data.
#    This will be the "workhorse" of your cluster.
#
#node.master: false
#node.data: true
#
# 2. You want this node to only serve as a master: to not store any data and
#    to have free resources. This will be the "coordinator" of your cluster.
#
#node.master: true
#node.data: false
#
# 3. You want this node to be neither master nor data node, but
#    to act as a "search load balancer" (fetching data from nodes,
#    aggregating results, etc.)
#
#node.master: false
#node.data: false

#################################### Paths ####################################

path.data: /usr/local/var/elasticsearch/
path.logs: /usr/local/var/log/elasticsearch/
path.plugins: /usr/local/var/lib/elasticsearch/plugins

############################## Network And HTTP ###############################
network.host: ["192.168.10.1", "_local_"]

################################## Discovery ##################################

discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["192.168.10.1", "192.168.10.111", "192.168.10.112",  "192.168.10.113"]

```

host 机作为 **client node**, 接下来的话是 master node 和 data node,

> By default a node is both a master-eligible node and a data node. This is very convenient for small clusters but, as the cluster grows, it becomes important to consider separating dedicated master-eligible nodes from dedicated data nodes.

'es1' 的 **elasticsearch.yml**

```yml
################################### Cluster ###################################

cluster.name: my-application

#################################### Node #####################################

node.name: es-mster
#node.master: true
node.data: true

############################## Network And HTTP ###############################
network.host: ["192.168.10.111"]

################################## Discovery ##################################

discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["192.168.10.1", "192.168.10.111", "192.168.10.112",  "192.168.10.113"]

```

'es2' 的 **elasticsearch.yml**

```yml
################################### Cluster ###################################

cluster.name: my-application

#################################### Node #####################################

node.name: es-data-1
node.data: true

############################## Network And HTTP ###############################
network.host: ["192.168.10.112"]

################################## Discovery ##################################

discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["192.168.10.1", "192.168.10.111", "192.168.10.112",  "192.168.10.113"]
```

'es3' 的 **elasticsearch.yml**

```yml
################################### Cluster ###################################

cluster.name: my-application

#################################### Node #####################################

node.name: es-data-2
node.data: true

############################## Network And HTTP ###############################
network.host: ["192.168.10.113"]

################################## Discovery ##################################

discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["192.168.10.1", "192.168.10.111", "192.168.10.112",  "192.168.10.113"]
```

全部 Elasticsearch 重启, 因为自己的 host 机原本就装了 kopf 插件, 所以, 直接浏览器打开.

![](http://ww1.sinaimg.cn/large/62fdd4d5jw1f88gde5vzvj22801e0dqs.jpg)

实心星星是 mater node, 空心星星是 Master-eligible node.

[https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)

**Master-eligible node**

> A node that has node.master set to true (default), which makes it eligible to be elected as the master node, which controls the cluster.

**Data node**

> A node that has node.data set to true (default). Data nodes hold data and perform data related operations such as CRUD, search, and aggregations.

**Client node**

> A client node has both node.master and node.data set to false. It can neither hold data nor become the master node. It behaves as a “smart router” and is used to forward cluster-level requests to the master node and data-related requests (such as search) to the appropriate data nodes.

随意找点东西索引,

![](http://ww4.sinaimg.cn/large/62fdd4d5gw1f88gdhuxikj22801e045r.jpg)

0, 1, 2, 3, 4 那些是分片, 其中, 亮的是 Primary Shards, 暗的是 Replica Shards. 默认的每个 index 都是 5 个 Primary Shards, 对应的 5 个 Replica Shards.
得益于  Replica Shards 任意一台机器挂了, 数据也还是完整的.

BTW, 除了监控插件以外, 比如一些分词插件, 得每个 node 也即是每台机器都要安装一遍.

##### 插曲

一开始设置的集群只有三个虚拟机, 分别是 client node, master node(node.data: false), data node. 添加索引以后集群状态一直是黄的. 一堆的 **unassigned_shards** 导致的 `"status" : "yellow"`.  因为只有一个 data node, 只能用来存 Primary Shards, 没地方存 Replica Shards. 最简单地, 把 master node 也作为 data node 用就绿了. 虽然官方说 master 和 data 最好分开.

##### 关于 Elasticsearch 的集群

当只有单台机器, 也就是我们一般在本地跑的时候, 通常只有是一个集群下面仅有一个 node.  master node  data node 都是这个 node, 集群状态一直就是黄的, 因为, 默认的配置是, 每个索引的每个 shard 都有一个 replica.

```
green
All primary and replica shards are active.

yellow
All primary shards are active, but not all replica shards are active.

red
Not all primary shards are active.
```

单台机器跑的时候, 单个 node 也就只能存 Primary Shards, 没地方存 Replica Shards, 所以就出现了 Unassigned 的 Shards, 也就黄了. 要想变回绿的话, 配置 `index.number_of_replicas` 为 0 就好了.

```yml
# Set the number of replicas (additional copies) of an index (1 by default):
#
index.number_of_replicas: 0
```

##### 容错
假设设定了,

```json
"number_of_shards" : 3,
"number_of_replicas" : 2
```

集群如图:

![](http://ww2.sinaimg.cn/large/62fdd4d5jw1f89bxwsba0j20ku069q36.jpg)

如果任意一台 down 了.

![](http://ww3.sinaimg.cn/large/62fdd4d5jw1f89bxyiluaj20ku0690sv.jpg)

这里 down 的是 master node, 索引在丢失主分片时不能正常工作. 此时检查集群 health，看到状态会是 red, 但 Elasticsearch 很快就会从剩下的 **master-eligible nodes** 里面选出一个新的 master node, 然后**集群状态变黄**.

这里因为指定了**每个主分片要有两个副本分片**, 但是, 因为现在每个主分片只剩下一个副本分片了, 所以, 还是不能绿. 不过, 假设这个时候剩余这两个 node 又不幸 down 掉某个的话, 我们的集群状态还是黄的, 并且可以正常跑起来, 没有任何数据丢失. 等到重启 Node 1 以后, 如果 Node 1 的数据并没有在 down 掉之后丢失旧数据，就可以尝试再利用起来，并只会从主分片上复制在故障期间有数据变更的那一部分数据.


