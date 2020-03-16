# Ice运行时详解

## 通信者（Communicators）

Ice运行时的核心是本地接口：Ice::Communicator，它包含了以下运行时资源：

* 客户端线程池，用于处理异步请求的回调，以避免在回调中出现死锁，和使用双向连接来处理请求。
* 服务端线程池，用于接收进来的连接和处理客户端的请求。
* 配置文件，Ice运行时的各个方面都可以通过配置文件进行配置，每个通信者都有自己的一套配置。
* 对象工厂，用于实例化Ice运行时所需的各种对象。
* 日志对象，实现了Ice::Logger接口，决定如何处理日志信息。
* 默认路由，把对象id路由给某个代理。
* 插件管理，插件可以为通信者增加新的特性。例如，IceSSL就是一个插件。每个通信者都有一个插件管理工具，实现了Ice::PluginManager接口，并提供了访问接口。
* 对象适配器，对象适配器负责分派请求，把请求转发给正确的服务。

对象适配器和那些使用不同通信者的对象都是完全独立的：

* 每个通信者使用自己的线程池，这意味着，一个通信者用完了线程池资源，只会影响使用这个通信者的对象，使用其他通信者的对象因为有自己的线程池所以不会受到影响。
* 搭配不同的通信者来使用不是一个好主意，使用同一个通信者可以避免很多请求的分派工作。

通常来说，服务只使用一个通信者，但个别情况下会使用多个通信者。例如，IceBox，给每一个Ice服务提供一个单独的通信者，确保不同服务不会使用互相调用。多个通信者还可以避免线程饥饿。

通信者接口定义在Slice，如下：

```
module Ice {    local interface Communicator {        string proxyToString(Object* obj);        Object* stringToProxy(string str);        PropertyDict proxyToProperty(Object* proxy, string property);        Object* propertyToProxy(string property);        Identity stringToIdentity(string str);        string identityToString(Identity id);        ObjectAdapter createObjectAdapter(string name);        ObjectAdapter createObjectAdapterWithEndpoints(string name,string endpoints);        ObjectAdapter createObjectAdapterWithRouter(string name, Router* rtr);        void shutdown();        void waitForShutdown();        bool isShutdown();        void destroy();        // ...};// ... };
```

通信者提供了一些操作：

* proxyToString<br/>
stringToProxy
* proxyToProperty<br/>
propertyToProxy
* identityToString<br/>
stringToIdentity
* createObjectAdapter<br/>
createObjectAdapterWithEndpoints<br/>
createObjectAdapterWithRouter
* shutdown