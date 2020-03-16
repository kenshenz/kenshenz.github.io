# Consul命令

## Consul Agent

`consul agent`是Consul的核心，它运行agent进程，来执行获取成员关系信息、健康自检、同步服务、处理查询请求等等工作。

## Consul Configtest

`consul configtest`用于测试Consul的配置文件，这使得我们很方便的检查配置文件是否有问题，而不用启动agent进程。

### 使用

`consul configtest [options]`

至少要给一个`-config-file`或`-config-dir`参数。返回0表示配置文件没问题，返回1表示配置文件有问题。

参数说明：

* `-config-file`：配置文件的路径，有多个配置文件则传多个参数。
* `-config-dir`：配置文件所在的目录，包括目录下所有`.json`文件，有多个目录则传多个参数。

## Consul Event

`consul event`命令提供了一种机制，使得用户可以推送自定义的事件给每一个数据中心。这些事件Consul不一定能理解，但我们可以用来构建自动化部署、重启服务、执行一些特定任务等等。通过`watch`来监听事件并处理。

与Consul数据不同，事件数据是纯点对点的，基于gossip协议的。这就意味着事件是非持久化的，无序的。实际上，这意味着你不能依赖于发送消息的顺序。但有一个好处及时在server节点断开或网络中断的情况下，事件依然可以被使用。

基于gossip，因此事件消息也有限制大小，很难给出一个准确的数字，因为它依赖事件参数的多少，但建议保持小于100字节。如果事件太大会返回error。

### 使用

`consul event [options] [payload]`

必要的选择只有`-name`，表示事件的名称。payload（载量）选项是可选的，如果要用，放在最后面即可。

参数说明：

* `-http-addr`：agent进程的Http地址，事件会发送到这个地址上，如果没有设置，则默认的发送到“127.0.0.1:8500”。
* `-datacenter`：要查询的数据中心，默认是本地agent进程。
* `-name`：事件的名称。
* `-node`：过滤哪些节点可以处理事件的正则表达式
* `-service`：过滤拥有哪些服务的节点可以处理事件的正则表达式
* `-tag`：过滤拥有哪些tag的服务的节点可以处理事件的正则表达式
* `-token`：处理事件时用到的ACL token

## Consul Watch

`consul watch`命令提供一套机制来监控特定数据（节点列表、服务成员、键值等等）的变化，和对最新的数据指定处理的程序。如果没有提供处理程序，默认地会把当前值打印到标准控制台上，这有助于我们在Consul中检查数据。

### 使用

`consul watch [options] [child...]`

只有一个必要的选项`-type`，用于指定要监控的数据，根据type值的不同，其他选项可能是必须的，也可能是可选的。

参数说明：

* `-http-addr`：你想发往的agent的http地址，如果不提供，默认会发送到“127.0.0.1:8500”。
* `-datacenter`：要进行查询的数据中心，默认是agent。
* `-token`：ACL token。
* `-key`：监控的关键字，只有当type=key时才使用。
* `-name`：监控的事件名，只有当type=event时才使用。
* `-passingonly=[true|false]`：是否应该跳过返回的实体，默认为false，只有当type=service时才使用。
* `-prefix`：监控的关键字前缀，只有当type=keyprefix时才使用。
* `-service`：监控的服务名，当type=service时为必需选项，当type=checks时为可选选项。
* `-state`：过滤哪些状态检查，当type=checks时为可选选项。
* `-tag`：过滤拥有哪些标记的服务，当type=service时为可选选项。
* `-type`：监控的类型，必需选项，可选值为“key”，“keyprefix”，“services”，“nodes”，“service”，“checks”或“event”