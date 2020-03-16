# Consul Web UI

Consul支持一个功能强大，界面优美的Web UI。通过Web UI可以查看所有服务和节点，还有他们的健康情况和当前状态，还可以读写键值对数据。Web UI自动支持多数据中心。

运行Consul的Web UI，有两个选择，一个是使用Atlas by HashiCorp，一个是使用主机本地的开源UI。

## Atlas by HashiCorp

<img src="{{ site.url }}/assets/images/atlas_web_ui-249f659e.png" width="960"/>

要使用Atlas UI，你需要在Consul配置文件中添加两个字段，Atlas的账号和Atlas的token。下面例子是在执行agent命令的时候设置帐号和token

```
$ consul agent -atlas=ATLAS_USERNAME/demo -atlas-token="ATLAS_TOKEN"
```

## 本地Web UI

<img src="{{ site.url }}/assets/images/consul_web_ui-3a1e7bf9.png" width="960"/>

要启动本地Web UI，只需要在启动agent的时候加上`-ui`参数即可。

```
$ consul agent -ui
...
```

启动后，访问地址为`http://localhost:8500/ui`。

