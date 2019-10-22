# zookeeper 命令及命令行工具

## zookeeper的四字命令

|ZooKeeper 四字命令|功能描述|
|---|---|
|conf|输出相关服务配置的详细信息。|
|cons|列出所有连接到服务器的客户端的完全的连接 / 会话的详细信息。包括“接受 / 发送”的包数量、会话 id 、操作延迟、最后的操作执行等等信息。|
|dump|列出未经处理的会话和临时节点。|
|envi|输出关于服务环境的详细信息（区别于 conf 命令）。|
|reqs|列出未经处理的请求|
|ruok|测试服务是否处于正确状态。如果确实如此，那么服务返回“imok ”，否则不做任何相应。|
|stat|输出关于性能和连接的客户端的列表。|
|wchs|列出服务器 watch 的详细信息。|
|wchc|通过 session 列出服务器 watch 的详细信息，它的输出是一个与watch 相关的会话的列表。|
|wchp|通过路径列出服务器 watch 的详细信息。它输出一个与 session相关的路径。|

### 四字命令的使用方法

```
echo dump | nc 10.128.129.200 2181 
```

## zookeeper的命令行客户端

zkCli.sh客户端位置zookeeper安装目录下的bin目录下，可以到该目录下执行

```
./zkCli.sh –server 10.77.20.23:2181
```

### zkCli 常用命令

* 查看节点

```
ls /
```

* 创建节点

```
create /nodepath nodevalue       (创建“nodepath”节点，其值为“nodevalue”)
```

* 获取节点的值

```
get /nodepath                    (获取指定节点的值)
```

* 更新节点的值

```
set /nodepath newnodevalue		 (更新节点的值)
```

* 删除节点

```
delete /nodepath				 (删除节点)
```