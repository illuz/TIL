# Guava-2, Git branch, Docker network, DNS Wildcard

## Guava-2

#### Cache

和 `ConcurrentMap` 差不多，区别是 `Cache` 有失效时间，不需要这种特性用 `ConcurrentMap` 会更好（本来就是基于它封装的）。

`CacheBuilder` 使用**构建者模式**。

Cache 可以在构建时指定 CacheLoader 来作为它的 get 不到时自动 load 一个数据的方法，也可以在 get 时传第二个 Callable 的参数来作为取不到时调用的创建方法。（ "if cached, return; otherwise create, cache and return" pattern.）

**回收机制**

- size：kv 数量
- weight: 提供 weigher 方法计算 value 的重量，然后限定 maxWeight
- time: 时间，`expireAfterWrite` 定时失效，`expireAfterAccess` 是多久没读或写到就失效

`Cache` 还能设置失效时的 Listener，这边用在于管理 Connection 是很不错的方案。

```java
CacheLoader<Key, DatabaseConnection> loader = new CacheLoader<Key, DatabaseConnection> () {
  public DatabaseConnection load(Key key) throws Exception {
    return openConnection(key);
  }
};
RemovalListener<Key, DatabaseConnection> removalListener = new RemovalListener<Key, DatabaseConnection>() {
  public void onRemoval(RemovalNotification<Key, DatabaseConnection> removal) {
    DatabaseConnection conn = removal.getValue();
    conn.close(); // tear down properly
  }
};

return CacheBuilder.newBuilder()
  .expireAfterWrite(2, TimeUnit.MINUTES)
  .removalListener(removalListener)
  .build(loader);
```

这边的 Listener 在里面是同步进行的，可能会让 cache 操作变慢，所以可以用`RemovalListeners.asynchronous(RemovalListener, Executor)` 来修饰一下这个 Listener，让它变成异步的。

**注意的点**

Gauav 的 Cache 默认是不会 CleanUp 的，它的 CleanUp 是在 Read/Write 后进行的，也就是说它不是异步的！
所以如果要实现自动去 CleanUp，你得定时去手动调用。
异步实现的原因是因为有些环境下开多线程会有限制或者开不了，所以就设计成了同步的。

#### Functions 和 Predicates 方法

其实 JDK8 大部分都有了，不过有些还是能用上：

- Functions.compose，把 `<A,B>` 和 `<B,C>` 的 Function 组合起来，JDK8: `g.compose(f)` or `f.andThen(g)`
- Functions.constant，不管输入什么东西都返回一个固定值，类似 `i -> 12`
- Functions.forMap，返回值是 map 里的值，还可以设置默认值，类型 `i -> map.getOrDefault(i, _d)`
- Functions.identity，返回原值，`i->i`
- 等等...懒得列了，直接看文档快点：[Functions](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html) [Prdicates](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html)

#### ListenableFuture

ListenableFuture 是对 Future 的扩展，JDK 8 的 CompletableFuture 就是参考它的。

主要特性是**任务链**，能在一个 Callable 结束后对结果再进行异步处理。

[Future, CompleteableFuture, RxJava 区别](https://stackoverflow.com/questions/35329845/difference-between-completablefuture-future-and-rxjavas-observable/35347215)

#### String 工具类

- **Joiner**，string join 工具类，可`skipNulls()`，返回 immutable
- **Splitter**，string split 工具类，除了按分隔号，还能按 regex 和长度来切字串
  - `",a,,b,,".split(",")` 返回时会跳过后面的空串，得到 `"", "a", "", "b"`，而 Splitter 可以自由控制，比如 `trimResults`,  `omitEmptyStrings`
- **CaseFormat**：字串风格工具类，可以转风格，如和驼峰、锯齿之类的

```
CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "CONSTANT_NAME")); // returns "constantName"
```

#### IO

```java
// Read the lines of a UTF-8 text file
ImmutableList<String> lines = Files.asCharSource(file, Charsets.UTF_8)
    .readLines();

// Count distinct word occurrences in a file
Multiset<String> wordOccurrences = HashMultiset.create(
    Splitter.on(CharMatcher.whitespace())
        .trimResults()
        .omitEmptyStrings()
        .split(Files.asCharSource(file, Charsets.UTF_8).read()));

// SHA-1 a file
HashCode hash = Files.asByteSource(file).hash(Hashing.sha1());

// Copy the data from a URL to a file
Resources.asByteSource(url).copyTo(Files.asByteSink(file));
```



## 删除不需要的 Git branch

批量本地没用分支

```sh
# 批量本地没用分支

# 列出本地 merged branches
git branch --merged

# 从结果中找到你不想删的分支，再列出 (grep -v 是反向命中)
git branch --merged | grep -v 'my_branch1' | grep -v 'my_branch2'

# 确认完本地这些分支是要删的，那就删了
git branch --merged | 
  grep -v 'my_branch1' | 
  grep -v 'my_branch2' | 
  xargs git branch -d
```

批量远程没用分支

```sh
# 批量远程没用分支

# 列出远程 merged branches
git branch -r --merged

# 从结果中找到你不想删的分支，再列出(grep -v 是反向命中)
git branch -r --merged | grep -v 'my_branch1' | grep -v 'my_branch2'

# 确认完远程这些分支是要删的，那就删了
git branch -r --merged |
  grep -v 'my_branch1' |
  grep -v 'my_branch2' | 
  sed 's/origin\///' |
  xargs git push origin --delete
```

## Docker 网络方案

Docker 上的网络是用虚拟网络设备实现的，叫 **veth**：

- veth pair是用于不同network namespace间进行通信的方式
- veth pair将一个network namespace数据发往另一个network namespace的veth
- namespace 和 veth 连着的一般是 TAP/TUN，详细见 [TUN, TAP and Veth - Virtual Networking Devices Explained](https://www.fir3net.com/Networking/Terms-and-Concepts/virtual-networking-devices-tun-tap-and-veth-pairs-explained.html)

如果多个network namespace需要进行通信，则需要借助 **bridge**，就类似交换机，bridge 上面有很多 TAP，可以把各个不同 namespace 并在一起通信。

docker 在 bridge 网络模式下就使用了**iptables**规则进行了nat转化。（说到iptables必然提到Netfilter，iptables是应用层的，其实质是一个定义规则的配置工具，而核心的数据包拦截和转发是Netfiler。）

### **几种网络方案**

#### bridge 模式：

![bridge](http://blog.daocloud.io/wp-content/uploads/2015/01/Docker_Topology.jpg)

1、使用一个 linux bridge，默认为 docker0。

2、使用 veth 对，一头在容器的网络 namespace 中，一头在 docker0 上。

3、该模式下Docker Container不具有一个公有IP，因为宿主机的IP地址与veth pair的IP地址不在同一个网段内。

4、Docker采用 NAT 方式，将容器内部的服务监听的端口与宿主机的某一个端口port 进行“绑定”，使得宿主机以外的世界可以主动将网络报文发送至容器内部。

5、外界访问容器内的服务时，需要访问宿主机的 IP 以及宿主机的端口 port。

6、NAT 模式由于是在三层网络上的实现手段，故肯定会影响网络的传输效率。

7、容器拥有独立、隔离的网络栈；让容器和宿主机以外的世界通过NAT建立通信。

#### host 模式

1. 和宿主机用同个网卡、Network Namespace、IP 和端口

#### none 模式

1、网络模式为 none，即不为 Docker 容器构造任何网络环境。

2、一旦Docker 容器采用了none 网络模式，那么容器内部就只能使用loopback网络设备，不会再有其他的网络资源。

3、容器不需要网络功能，这种模式适用于容器包含写数据到磁盘卷的一些任务。

#### Container 模式

Container 网络模式是 Docker 中一种较为特别的网络的模式。处于这个模式下的 Docker容器会共享其他容器的网络环境，因此，至少这两个容器之间不存在网络隔离，而这两个容器又与宿主机以及除此之外其他的容器存在网络隔离。  

## DNS Wildcard

[Wildcard DNS record - Wiki](https://en.wikipedia.org/wiki/Wildcard_DNS_record)，中文叫外卡。

就是让 DNS 能按二级或更深的域名来匹配 ip，比如 user_xxx.example.com 打到 xxx 用户的 ip 上。

一般的域名服务商都能在后台配 wildcard，当然也可以自己配 DNS 服务器。

自己配的话，可以用 http://xip.io/ 的 powerDNS：https://github.com/basecamp/xip-pdns