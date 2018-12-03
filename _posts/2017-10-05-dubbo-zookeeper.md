---
title: Dubbo and Zookeeper
key: 20171005
tags: Dubbo
aside:
  toc: true
excerpt_separator: <!--more-->
excerpt_type: text # text (default), html
---
Dubbo的Provider，Consumer在启动时都会创建一个注册中心，注册中心可以选择Zookeeper，Redis。常用的是Zookeeper，我们这篇博客主要讲的就是Dubbo与Zookeeper的注册交互过程。

Dubbo里默认使用zkclient来操作zookeeper服务器，其对zookeeper原始客户单做了一定的封装，操作zookeeper时能便捷一些，比如不需要手动处理session超时，不需要重复注册watcher等等。
<!--more-->

Dubbo在Zookeeper上注册的节点目录：假设接口名称是：com.bob.dubbo.service.CityDubboService



Dubbo启动时，Consumer和Provider都会把自身的URL格式化为字符串，然后注册到zookeeper相应节点下，作为一个**临时节点**，当连断开时，节点被删除。

Consumer在启动时，不仅仅会注册自身到 …/consumers/目录下，同时还会订阅…/providers目录，实时获取其上Provider的URL字符串信息。

下面我们就看相关的代码实现：
```java
public class ZookeeperRegistry extends FailbackRegistry {

    ......

    /**
     * 默认端口
     */
    private final static int DEFAULT_ZOOKEEPER_PORT = 2181;
    /**
     * 默认 Zookeeper 根节点
     */
    private final static String DEFAULT_ROOT = "dubbo";

    /**
     * Zookeeper 根节点
     */
    private final String root;
    /**
     * Service 接口全名集合
     */
    private final Set<String> anyServices = new ConcurrentHashSet<String>();
    /**
     * 监听器集合
     */
    private final ConcurrentMap<URL, ConcurrentMap<NotifyListener, ChildListener>> zkListeners
        = new ConcurrentHashMap<URL, ConcurrentMap<NotifyListener, ChildListener>>();
    /**
     * Zookeeper 客户端
     */
    private final ZookeeperClient zkClient;

    public ZookeeperRegistry(URL url, ZookeeperTransporter zookeeperTransporter) {
        super(url);   // 调用父类FailbackRegistry的构造函数
        if (url.isAnyHost()) {
            throw new IllegalStateException("registry address == null");
        }
        // 获得 Zookeeper 根节点, 未指定 "group" 参数时为 dubbo
        String group = url.getParameter(Constants.GROUP_KEY, DEFAULT_ROOT); // `url.parameters.group` 参数值
        if (!group.startsWith(Constants.PATH_SEPARATOR)) {
            group = Constants.PATH_SEPARATOR + group;
        }
        this.root = group;   // root = "/dubbo"
        // 创建 Zookeeper Client
        zkClient = zookeeperTransporter.connect(url);
        // 添加 StateListener 对象。该监听器，在重连时，调用恢复方法。
        zkClient.addStateListener(new StateListener() {
            @Override
            public void stateChanged(int state) {
                if (state == RECONNECTED) {
                    try {
                        recover();
                    } catch (Exception e) {
                        logger.error(e.getMessage(), e);
                    }
                }
            }
        });
    }
}

public abstract class FailbackRegistry extends AbstractRegistry {

    ......

    /**
     * 失败发起注册失败的 URL 集合
     */
    private final Set<URL> failedRegistered = new ConcurrentHashSet<URL>();
    /**
     * 失败取消注册失败的 URL 集合
     */
    private final Set<URL> failedUnregistered = new ConcurrentHashSet<URL>();
    /**
     * 失败发起订阅失败的监听器集合
     */
    private final ConcurrentMap<URL, Set<NotifyListener>> failedSubscribed = new ConcurrentHashMap<URL, Set<NotifyListener>>();
    /**
     * 失败取消订阅失败的监听器集合
     */
    private final ConcurrentMap<URL, Set<NotifyListener>> failedUnsubscribed = new ConcurrentHashMap<URL, Set<NotifyListener>>();
    /**
     * 失败通知通知的 URL 集合
     */
    private final ConcurrentMap<URL, Map<NotifyListener, List<URL>>> failedNotified = new ConcurrentHashMap<URL, Map<NotifyListener, List<URL>>>();

    public FailbackRegistry(URL url) {
        super(url);
        // 重试频率，单位：毫秒 ，默认 5*1000
        int retryPeriod = url.getParameter(Constants.REGISTRY_RETRY_PERIOD_KEY, Constants.DEFAULT_REGISTRY_RETRY_PERIOD);
        // 创建失败重试定时器
        this.retryFuture = retryExecutor.scheduleWithFixedDelay(new Runnable() {
            public void run() {
                // Check and connect to the registry
                try {
                    retry();
                } catch (Throwable t) { // Defensive fault tolerance
                    logger.error("Unexpected error occur at failed retry, cause: " + t.getMessage(), t);
                }
            }
        }, retryPeriod, retryPeriod, TimeUnit.MILLISECONDS);
    }

    /**
     * 重试
     */
    // Retry the failed actions
    protected void retry() {
        // 重试执行注册
        if (!failedRegistered.isEmpty()) {
            ......
            for (URL url : failed) {
                try {
                    // 执行注册
                    doRegister(url);
                    // 移除出 `failedRegistered`
                    failedRegistered.remove(url);
                } catch (Throwable t) { // Ignore all the exceptions and wait for the next retry
                    logger.warn("Failed to retry register " + failed + ", waiting for again, cause: " + t.getMessage(), t);
                }
            }
        }
        // 重试执行取消注册
        if (!failedUnregistered.isEmpty()) {
            ......
            for (URL url : failed) {
                try {
                    // 执行取消注册
                    doUnregister(url);
                    // 移除出 `failedUnregistered`
                    failedUnregistered.remove(url);
                } catch (Throwable t) { // Ignore all the exceptions and wait for the next retry
                    logger.warn("Failed to retry unregister  " + failed + ", waiting for again, cause: " + t.getMessage(), t);
                }
            }
        }
        // 重试执行订阅
        if (!failedSubscribed.isEmpty()) {
            ......
            for (Map.Entry<URL, Set<NotifyListener>> entry : failed.entrySet()) {
                URL url = entry.getKey();
                Set<NotifyListener> listeners = entry.getValue();
                for (NotifyListener listener : listeners) {
                    try {
                        // 执行订阅
                        doSubscribe(url, listener);
                        // 移除监听器
                        listeners.remove(listener);
                    } catch (Throwable t) { // Ignore all the exceptions and wait for the next retry
                        logger.warn("Failed to retry subscribe " + failed + ", waiting for again, cause: " + t.getMessage(), t);
                    }
                }
            }
        }
        // 重试执行取消订阅
        if (!failedUnsubscribed.isEmpty()) {
            ......
            for (Map.Entry<URL, Set<NotifyListener>> entry : failed.entrySet()) {
                URL url = entry.getKey();
                Set<NotifyListener> listeners = entry.getValue();
                for (NotifyListener listener : listeners) {
                    try {
                        // 执行取消订阅
                        doUnsubscribe(url, listener);
                        // 移除监听器
                        listeners.remove(listener);
                    } catch (Throwable t) { // Ignore all the exceptions and wait for the next retry
                        logger.warn("Failed to retry unsubscribe " + failed + ", waiting for again, cause: " + t.getMessage(), t);
                    }
                }
            }
        }
    }

}
```
ZookeeperRegistry 在实例化时，调用父类构造函数。在父类构造函数中，会创建一个定时任务，每隔5S执行retry( ) 方法。

在retry( ) 方法中，重试那些失败的动作。重试的动作包括：

```
1. Provider向zookeeper注册自身的url，生成一个临时的znode
2. Provider从Dubbo容器中退出，停止提供RPC调用。也就是移除zookeeper内自身url对应的znode
3. Consumer订阅 ” /dubbo/..Service/providers” 目录的子节点，生成ChildListener
4. Consumer从Dubbo容器中退出，移除之前创建的ChildListener
```

为什么如此设置？ 主要是和zookeeper的通信机制有关的。当zookeeper的Client和Server连接断开，或者心跳超时，那么Server会将相应Client注册的临时节点删除，当然注册的Listener也相应删除。

而Provider和Consumer注册的URL就属于临时节点，当连接断开时，Dubbo注册了zookeeper的StateListener，也就是状态监听器，当Dubbo里的zookeeper Client和Server重新连接上时，将之前注册的的URL添加入这几个失败集合中，然后重新注册和订阅。

看ZookeeperRegistry 的构造函数，其添加了一个StateListener：

```java
public class ZookeeperRegistry extends FailbackRegistry {

    public ZookeeperRegistry(URL url, ZookeeperTransporter zookeeperTransporter) {
        ......
        // 添加 StateListener 对象。该监听器，在重连时，调用恢复方法。
        zkClient.addStateListener(new StateListener() {
            @Override
            public void stateChanged(int state) {
                if (state == RECONNECTED) {
                    try {
                        recover();
                    } catch (Exception e) {
                        logger.error(e.getMessage(), e);
                    }
                }
            }
        });
    }

}

```

```java
public abstract class FailbackRegistry extends AbstractRegistry {

    protected void recover() throws Exception {
        // register 恢复注册，添加到 `failedRegistered` ，定时重试
        Set<URL> recoverRegistered = new HashSet<URL>(getRegistered());
        if (!recoverRegistered.isEmpty()) {
            if (logger.isInfoEnabled()) {
                logger.info("Recover register url " + recoverRegistered);
            }
            for (URL url : recoverRegistered) {
                failedRegistered.add(url);
            }
        }
        // subscribe 恢复订阅，添加到 `failedSubscribed` ，定时重试
        Map<URL, Set<NotifyListener>> recoverSubscribed = new HashMap<URL, Set<NotifyListener>>(getSubscribed());
        if (!recoverSubscribed.isEmpty()) {
            if (logger.isInfoEnabled()) {
                logger.info("Recover subscribe url " + recoverSubscribed.keySet());
            }
            for (Map.Entry<URL, Set<NotifyListener>> entry : recoverSubscribed.entrySet()) {
                URL url = entry.getKey();
                for (NotifyListener listener : entry.getValue()) {
                    addFailedSubscribed(url, listener);
                }
            }
        }
    }

}
```


ZookeeperRegistry 构造函数中为zookeeper的操作客户端添加了一个状态监听器 StateListener，当重新连接时( 重新连接意味着之前连接断开了 )，将已经注册和订阅的URL添加到失败集合中，定时重试，也就是重新注册和订阅。

zookeeper Client与Server断开连接后，会定时的不断尝试重新连接，当连接成功后就会触发一个Event，Dubbo注册了CONNECTED状态的监听器，当连接成功后重新注册和订阅。

zookeeper Server宕机了，Dubbo里的Client并没有对此事件做什么响应，当然其内部的zkClient会不停地尝试连接Server。当Zookeeper Server宕机了不影响Dubbo里已注册的组件的RPC调用，因为已经通过URL生成了Invoker对象，这些对象还在Dubbo容器内。当然因为注册中心宕机了，肯定不能感知到新的Provider。同时因为在之前订阅获得的Provider信息已经持久化到本地文件，当Dubbo应用重启时，如果zookeeper注册中心不可用，会加载缓存在文件内的Provider信息，还是能保证服务的高可用。

Consumer会一直维持着对Provider的ChildListener，监听Provider的实时数据信息。当Providers节点的子节点发生变化时，实时通知Dubbo，更新URL，同时更新Dubbo容器内的Consumer Invoker对象，只要是订阅成功均会实时同步Provider，更新Invoker对象，无论是第一次订阅还是断线重连后的订阅：

```java
public class ZookeeperRegistry extends FailbackRegistry {

    protected void doSubscribe(final URL url, final NotifyListener listener) {
        try {
            // 处理所有 Service 层的发起订阅，例如监控中心的订阅
            if (Constants.ANY_VALUE.equals(url.getServiceInterface())) {
                ......
                // 处理指定 Service 层的发起订阅，例如服务消费者的订阅
            } else {
                // 子节点数据数组
                List<URL> urls = new ArrayList<URL>();
                // 循环分类数组 , router, configurator, provider
                for (String path : toCategoriesPath(url)) {
                    // 获得 url 对应的监听器集合
                    ConcurrentMap<NotifyListener, ChildListener> listeners = zkListeners.get(url);
                    if (listeners == null) { // 不存在，进行创建
                        zkListeners.putIfAbsent(url, new ConcurrentHashMap<NotifyListener, ChildListener>());
                        listeners = zkListeners.get(url);
                    }
                    // 获得 ChildListener 对象
                    ChildListener zkListener = listeners.get(listener);
                    if (zkListener == null) { //  不存在子目录的监听器，进行创建 ChildListener 对象
                        // 订阅父级目录, 当有子节点发生变化时，触发此回调函数
                        listeners.putIfAbsent(listener, new ChildListener() {
                            @Override
                            public void childChanged(String parentPath, List<String> currentChilds) {
                                // 变更时，调用 `#notify(...)` 方法，回调 NotifyListener
                                ZookeeperRegistry.this.notify(url, listener, toUrlsWithEmpty(url, parentPath, currentChilds));
                            }
                        });
                        zkListener = listeners.get(listener);
                    }
                    // 创建 Type 节点。该节点为持久节点。
                    zkClient.create(path, false);
                    // 向 Zookeeper ，PATH 节点，发起订阅,返回此节点下的所有子元素 path : /根节点/接口全名/providers, 比如 ： /dubbo/com.bob.service.CityService/providers
                    List<String> children = zkClient.addChildListener(path, zkListener);
                    // 添加到 `urls` 中
                    if (children != null) {
                        urls.addAll(toUrlsWithEmpty(url, path, children));
                    }
                }
                // 首次全量数据获取完成时，调用 `#notify(...)` 方法，回调 NotifyListener, 在这一步从连接Provider,实例化Invoker
                notify(url, listener, urls);
            }
        } catch (Throwable e) {
            throw new RpcException("Failed to subscribe " + url + " to zookeeper " + getUrl() + ", cause: " + e.getMessage(), e);
        }
    }

}

```


订阅获取Providers的最新URL字符串，调用notify(…)方法，通知监听器，最终会执行如下代码：

```java
public class RegistryDirectory<T> extends AbstractDirectory<T> implements NotifyListener {

    private volatile List<Configurator> configurators;

    private volatile Map<String, Invoker<T>> urlInvokerMap;

    private volatile Map<String, List<Invoker<T>>> methodInvokerMap;  

    private volatile Set<URL> cachedInvokerUrls; 


    private void refreshInvoker(List<URL> invokerUrls) {
        // 从zookeeper获取到的url已经没有合适的了,在订阅返回为空时,会手动生成一个 EMPTY_PROTOCOL 的 url
        if (invokerUrls != null && invokerUrls.size() == 1 && invokerUrls.get(0) != null
            && Constants.EMPTY_PROTOCOL.equals(invokerUrls.get(0).getProtocol())) {
            this.forbidden = true; // Forbid to access
            this.methodInvokerMap = null; // Set the method invoker map to null
            destroyAllInvokers(); // Close all invokers
        } else {
            this.forbidden = false; // Allow to access
            Map<String, Invoker<T>> oldUrlInvokerMap = this.urlInvokerMap; // local reference
            if (invokerUrls.isEmpty() && this.cachedInvokerUrls != null) {
                invokerUrls.addAll(this.cachedInvokerUrls);
            } else {
                this.cachedInvokerUrls = new HashSet<URL>();
                this.cachedInvokerUrls.addAll(invokerUrls);//Cached invoker urls, convenient for comparison
            }
            if (invokerUrls.isEmpty()) {
                return;
            }
            Map<String, Invoker<T>> newUrlInvokerMap = toInvokers(invokerUrls);// Translate url list to Invoker map
            Map<String, List<Invoker<T>>> newMethodInvokerMap = toMethodInvokers(newUrlInvokerMap); // Change method name to map Invoker Map
            // state change
            // If the calculation is wrong, it is not processed.
            if (newUrlInvokerMap == null || newUrlInvokerMap.size() == 0) {
                logger.error(new IllegalStateException(
                    "urls to invokers error .invokerUrls.size :" + invokerUrls.size() + ", invoker.size :0. urls :" + invokerUrls.toString()));
                return;
            }
            this.methodInvokerMap = multiGroup ? toMergeMethodInvokerMap(newMethodInvokerMap) : newMethodInvokerMap;
            this.urlInvokerMap = newUrlInvokerMap;
            try {
                destroyUnusedInvokers(oldUrlInvokerMap, newUrlInvokerMap); // Close the unused Invoker
            } catch (Exception e) {
                logger.warn("destroyUnusedInvokers error. ", e);
            }
        }
    }

}

```

更新Dubbo内的Invoker相关数据，保证Consumer能实时感知到Provider的信息，保证PRC调用不会出错。

以上就是Dubbo内Zookeeper注册中心的实现过程。

总结：

**1.Provider和Consumer向Zookeeper注册临时节点，当连接断开时删除相应的注册节点。** 

**2.Consumer订阅Providers节点的子节点，实时感知Provider的变化情况，实时同步自身的Invoker对象，保证RPC的可用性。**