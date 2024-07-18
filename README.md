# 本文目的是合理利用旧手机，利用废物和低功耗高性能的特点，将其作为环保可靠的家庭软路由/服务器设备。
Android Router / 安卓路由器
Android as router / server

## 笔者环境：
- 旧手机zuk z2 pro，6+128，骁龙820，支持2.4g/5g wifi，otg
- 系统：自编译lineageos 17.1（安卓10, Linux 4.9，LinuxDeploy ubuntu 18）


## 最终效果：
- 稳定运行已有2个星期
- 整机待机功耗 < 0.5W，代理功耗约 < 2W（功耗表测）
- openssl 测试aes算力大概单核100M/s。（受限于火龙820，上次测试红米k40（骁龙870），算力可以到900M/s。）

## 费用
合计3元：
- 可调电源模块：3元（也可以直接用电阻）
- 12v/2A DC电源：0元（旧设备的，约10元）
- type-c/RJ45网卡：0元（笔记本送的，约25元）


## 硬件改造：
### 散热：
由于我的zuk z2用的是火龙820，因此可以增强一下散热以应对极端满载情况。
拆开后盖，在对应发热位置，垫上导热硅胶片，再盖上铝制散热鳞甲。

### 网卡：
因为我的目的是有线链接路由器，无线开热点，因此需要利用type-c口来otg一块RJ45网卡，刚好我有笔记本送的闲置着，就直接插上使用。


### 供电：
网上有直接用电容替换电芯，手机直插type-c口充电的方案，因为我网卡占了type-c口，所以我只能改电芯的方案。
淘宝3块钱买了个可调降压模块。
将电池拆开，小心去除电芯剩下电池保护板，按照对应的正负极把供电模块的输出端接到电池保护板。降压输出端显示4.3v，手机显示电量是84%。4.2v显示电量80%。注意这里我买的可调降压模块必须要是压差2.5v以上，电流2A以上，所以就意味着我这个情况5v/2a的充电器并不可以用，在负载高的场景会掉电压然后就断电重启了。我最终用的是12v/2a的dc电源，非常稳定。


## 系统改造：
优先选择LineageOS。
### 内核配置
这个手机有lineageos官方支持17.1，内核4.9，把一些常见的内核config打开（可选）：
```
CONFIG_NET_NS=y
CONFIG_PID_NS=y
CONFIG_IPC_NS=y
CONFIG_UTS_NS=y
CONFIG_CGROUP_DEVICE=y
CONFIG_MEMCG=y
CONFIG_VETH=y
CONFIG_NETFILTER_XT_MATCH_ADDRTYPE=y
CONFIG_NETFILTER_XT_MATCH_IPVS=y
CONFIG_POSIX_MQUEUE=y
CONFIG_DEVPTS_MULTIPLE_INSTANCES=y
CONFIG_USER_NS=y
CONFIG_CGROUP_PIDS=y
CONFIG_MEMCG_SWAP=y
CONFIG_MEMCG_SWAP_ENABLED=y
CONFIG_NENCG_KMEM=y
CONFIG_BLK_DEV_THROTTLING=y
CONFIG_CGROUP_PERF=y
CONFIG_CGROUP_HUGETLB=y
CONFIG_NET_CLS_CGROUP=y
CONFIG_CGROUP_NET_PRIO=y
CONFIG_CFS_BANDWIDTH=y
CONFIG_IP_VS=y
CONFIG_IP_VS_NFCT=y
CONFIG_IP_VS_PROTO_TCP=y
CONFIG_IP_VS_PROTO_UDP=y
CONFIG_IP_VS_RR=y
CONFIG_SECURITY_APPARMOR=y
CONFIG_EXT4_FS_POSIX_ACL=y
CONFIG_VXLAN=y
CONFIG_BRIDGE_VLAN_FILTERING=y
CONFIG_CRYPTO_AEAD=y
CONFIG_CRYPTO_GCM=y
CONFIG_CRYPTO_SEQIV=y
CONFIG_CRYPTO_GHASH=y
CONFIG_XFRM=y
CONFIG_XFRM_USER=y
CONFIG_XFRM_ALGO=y
CONFIG_INET_ESP=y
CONFIG_NETFILTER_XT_MATCH_BPF=y
CONFIG_INET_XFRM_MODE_TRANSPORT=y
```

### Docker原生支持（可选）：
参照这个帖子，不可以自动化安装，实测会出错，需要手动逐步安装。
最后将/目录手动remount成rw，可以成功运行hello world。
但是后来跑了个nginx，也能跑起来，但运行3-4分钟以后系统会crash重启，看了一圈日志也没看到什么异常。感觉自己暂时也用不到docker就弃坑了。
https://gist.github.com/arno01/ebf570af208e28c1a0cf78da4f63bc9c

### 手机开AP热点并走vpn：
在LineageOS系统中：
设置 - 网络与连接 - 热点 - 允许接入热点的设备使用VPN


## 软件改造：
# 软件列表
- macrodroid：安卓自动化工具
- linux deploy：原生的linux环境，因为我们有root，所以用linuxdeploy实现完整的root linux，而不是其他使用proot的linux。
- via浏览器：浏览网页时使用
- 迅雷破解版apk：下载时使用

### 开机启动：

#### macrodroid：安卓自动化工具，系统启动时执行以下动作：
- 启动adb wifi，可以方便远程用scrcpy连接手机
- 启动热点
- 禁用电池优化和待机相关策略
- 模拟充电：直供电会有系统掉电的bug（可能），因此直接供电
- 启动vpn（v2rayng）

#### linux deploy：
- 开机自启动
- 启用网络变化监控
- 挂载sdcard

#### v2rayng
- geo dat数据文件避免更新，或使用轻量版的geo 数据(https://github.com/Hackl0us/GeoIP2-CN)
