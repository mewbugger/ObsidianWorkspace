#### 前提
对于集群模式，集群内的每个Nacos成员都需要相互通信。因此这就带来一个问题，该以何种方式去管理集群内部的Nacos成员节点信息，即Nacos内部的寻址机制。
#### 设计
要能够**感知到节点的变更情况**：节点是增加了还是减少了；当前最新的成员列表信息是什么；以何种方式去管理成员列表信息；如何快速的支持新的、更优秀的成员列表管理模式等等。
针对上述需求点，我们抽象出了一个 **MemberLookup** 接口，具体设计如下：
``` java
public interface MemberLookup {
    
    /**
     * start.
     *
     * @throws NacosException NacosException
     */
    void start() throws NacosException;
    
    /**
     * Inject the ServerMemberManager property.
     *
     * @param memberManager {@link ServerMemberManager}
     */
    void injectMemberManager(ServerMemberManager memberManager);
    
    /**
     * The addressing pattern finds cluster nodes.
     *
     * @param members {@link Collection}
     */
    void afterLookup(Collection<Member> members);
    
    /**
     * Addressing mode closed.
     *
     * @throws NacosException NacosException
     */
    void destroy() throws NacosException;
}
```
（`ServerMemberManager` 存储着本节点**所知道的所有成员节点列表信息**，提供了针对成员节点的增删改查操作，同时维护了一个 `MemberLookup` 列表，方便**进行动态切换成员节点寻址方式**。）

可以看到，`MemberLookup` 接口非常简单，核心接口就两个—— `injectMemberManager` 以及 `afterLookup` ，**前者**用于将 `ServerMemberManager` **注入**到 `MemberLookup` 中，方便利用 `ServerMemberManager` 的存储、查询能力，**后者** `afterLookup` 则是一个**事件接口**，当 `MemberLookup` 需要进行成员节点信息更新时，会将当前最**新的成员节点列表信息通过该函数进行通知**给 `ServerMemberManager`，具体的节点管理方式，则是隐藏到具体的 `MemberLookup` 实现中。
#### 内部实现
##### 单机寻址
`com.alibaba.nacos.core.cluster.lookup.StandaloneMemberLookup`
单机模式的寻址模式很简单，其实就是找到自己的**IP:PORT组合信息**，然后格式化为一个节点信息，调用`afterLookup` 然后将信息存储到 `ServerMemberManager` 中。
``` java
public class StandaloneMemberLookup extends AbstractMemberLookup {
    
    @Override
    public void start() {
        if (start.compareAndSet(false, true)) {
            String url = InetUtils.getSelfIp() + ":" + ApplicationUtils.getPort();
            afterLookup(MemberUtils.readServerConf(Collections.singletonList(url)));
        }
    }
}
```
##### 文件寻址
`com.alibaba.nacos.core.cluster.lookup.FileConfigMemberLookup`
文件寻址模式是 Nacos **集群**模式下的**默认**寻址实现。文件寻址模式很简单，其实就是**每个 Nacos 节点需要维护一个叫做** `cluster.conf` 的文件，如下;
``` plain text
192.168.16.101:8847
192.168.16.102
192.168.16.103
```
该文件**默认只需要**填写每个成员节点的 IP **信息即可**，端口会**自动选择 Nacos 的默认端口 8848**，如果说有特殊需求**更改了 Nacos 的端口**信息，则需要在该文件将该节点的完整网路地址信息**补充完整**（IP:PORT）
当 Nacos节点启动时，会读取该文件的内容，然后将文件内的 IP 解析为节点列表，调用 `afterLookup` 存入`ServerMemberManager` 。
``` java
private void readClusterConfFromDisk() {
    Collection<Member> tmpMembers = new ArrayList<>();
    try {
        List<String> tmp = ApplicationUtils.readClusterConf();
        tmpMembers = MemberUtils.readServerConf(tmp);
    } catch (Throwable e) {
        Loggers.CLUSTER
                    .error("nacos-XXXX [serverlist] failed to get serverlist from disk!, error : {}", e.getMessage());
    }
     afterLookup(tmpMembers);
}
```
如果发现**集群扩缩容**，那么就需要**修改每个 Nacos 节点下的 cluster.conf 文件**，然后 Nacos 内部的文件变动监听中心会**自动发现文件修改，重新读取文件内容、加载 IP 列表信息、更新新增的节点。**
``` java
private FileWatcher watcher = new FileWatcher() {
    @Override
    public void onChange(FileChangeEvent event) {
        readClusterConfFromDisk();
    }
        
    @Override
    public boolean interest(String context) {
        return StringUtils.contains(context, "cluster.conf");
    }
};

public void start() throws NacosException {
    if (start.compareAndSet(false, true)) {
        readClusterConfFromDisk();
            
        // Use the inotify mechanism to monitor file changes and automatically
        // trigger the reading of cluster.conf
        try {
            WatchFileCenter.registerWatcher(ApplicationUtils.getConfFilePath(), watcher);
        } catch (Throwable e) {
            Loggers.CLUSTER.error("An exception occurred in the launch file monitor : {}", e.getMessage());
        }
    }
}
```
**默认寻址模式**有一个**缺点**——**运维成本较大**，可以想象下，当你新增一个 Nacos 节点时，需要去手动修改每个 Nacos 节点下的 cluster.conf 文件，这是多么辛苦的一件工作，或者稍微高端一点，利用 ansible 等自动化部署的工具去推送 cluster.conf 文件去代替自己的手动操作，虽然说省去了较为繁琐的人工操作步骤，但是仍旧存在一个问题——每一个 Nacos 节点都存在一份 cluster.conf 文件，**如果其中一个节点的 cluster.conf 文件修改失败，就造成了集群间成员节点列表数据的不一致性**，因此，又引申出了新的寻址模式——**地址服务器寻址模式**。
##### 地址服务器寻址
`com.alibaba.nacos.core.cluster.lookup.AddressServerMemberLookup`
地址服务器寻址模式是 Nacos 官方推荐的一种集群成员节点信息管理，该模式利用了一个**简易的 web 服务器**，**用于管理 cluster.conf 文件**的内容信息，这样，运维人员**只需要管理这一份集群成员节点内容即可**，而**每个Nacos 成员节点，只需要向这个 web 节点定时请求当前最新的集群成员节点列表信息即可**。
![](../../../img/Pasted%20image%2020240317214446.png)因此，通过地址服务器这种模式，大大简化了 Nacos 集群节点管理的成本，同时，地址服务器是一个非常简单的 web 程序，其程序的稳定性能够得到很好的保障。