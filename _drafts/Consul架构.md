# Consul架构

Consul是一个复杂的系统，包含了很多个不同的灵活的部件。

## 词汇

在描述Consul的架构之前，我们先来看看下面一些词汇的解释，这样有助于我们更清楚的知道我们将要讨论的东西。

* Agent：agent是集群中每个成员都会运行的守护进程。用`consul agent`来启动，agent能够以*client*模式或*server*模式来运行。因为每一个节点都必须运行一个agent进程，因此可以简单的认为节点是一个服务端节点或客户端节点，但也有其他agent的实例。所有agent都可以运行DNS API或HTTP API，并负责健康检查和服务同步工作。
* Client：client是一个agent，负责转发所有RPC请求到server，client几乎是无状态的。client唯一要做的就是参与LAN gossip pool，这只会占用网络带宽中很小的一部分（开销很小）。
* Server：server是一个agent，但承担了更多的工作，包括参与Raft quorum，获取集群的状态，响应RPC请求，与其他数据中心交换WAN gossip，转发查询到leader进程或其他数据中心。
* Datacenter：数据中心的作用看起来显而易见，但也有一些细节需要注意。例如，在EC2，一个数据中心由多个可用区域构成？我们定义数据中心为一个私有的、低延迟的、高带宽的网络环境。它不包括与公共网络的交互，对我们来说，单个EC2下的多个有效区域可以组成一个数据中心。
* Consensus：在本文档中，我们用consensus来表示同意leader节点的推选和同意数据处理的顺序。当一批数据处理发送到有限状态机，我们对consensus的定义改为状态一致性。更多关于consensus的说明可以查看[wiki](https://en.wikipedia.org/wiki/Consensus_(computer_science))，或者[我们的描述](https://www.consul.io/docs/internals/consensus.html)
* Gossip：Consul是建立在[Serf](https://www.serfdom.io)之上的，Serf提供了完整的一套[gossip protocol](https://en.wikipedia.org/wiki/Gossip_protocol)。Serf提供了成员关系管理、故障检查和消息广播。我们对gossip的使用在[gossip文档](https://www.consul.io/docs/internals/gossip.html)中有更详细的介绍。这里我们只要知道gossip主要通过UDP来建立随机节点到节点之间的通信就行了。
* LAN Gossip：参考LAN gossip池，存储着所有分布在相同网络区域或相同数据中心的节点。
* WAN Gossip：参考WAN gossip池，只存储server节点。这些server节点分布在不同的数据中心，并且通过互联网或广域网来通信。
* RPC：Remote Procedure Call。

## 10,000 foot view

<img src="{{ site.url }}/assets/images/consul-arch-5d4e3623.png" width="960"/>

我们可以看到有两个数据中心，Consul对多数据中有着很高级别的支持力度。

在各自的数据中心里面，混合着多个client和server，一般建议一个数据中心包含3到5个server。这是为了在效率和故障之间取得平衡，因为添加机器会使consensus变得越来越慢。不过，对于client就没有限制了，可以轻松扩展到千万级别的数量。

数据中心里的所有节点都会加入gossip协议，这意味着有一个gossip池，包含了数据中心里所有的节点。这样就能达到几个目的：第一，不需要为client配置服务的地址，服务发现是自动的。第二，检查故障的工作不需要放在服务端，并且是分布的。这就使得故障检查比心跳机制更加灵活和可扩展。第三，作为消息层来通知重要的消息，例如发生了leader推选。

数据中心里的所有节点都是Raft的一部分。这意味着它们合作来推选出leader（拥有额外工作的服务）。leader负责处理所有查询和数据处理。数据处理必须被复用到所有遵循consensus协议的节点。因为这原因，当一个非leader服务接收到RPC请求后，会转发到集群的leader。

服务节点也要操作WAN gossip池，不同于LAN池，WAN池被优化过，用于高延迟的互联网通信，并且只跟其他Consul的服务节点通信。WAN池的目的是允许各个数据中心通过低接触方式来互相发现对方。创建了新的数据中心后也能容易地加入到已有的WAN gossip池，因为服务都会在这个池中操作，所以也就可以跨数据中心请求了。当一个服务接收到另一个数据中心的服务的请求时，它会转发给正确的数据中心的任意一个服务节点，该服务还可能继续转发给它的leader。

这就造成了数据中心之间的低接触方式，但因为故障检查、连接缓存和多路传输，跨数据中心请求是高效的，而且可靠的。