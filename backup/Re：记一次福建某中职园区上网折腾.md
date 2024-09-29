~~副：记一次宁技/宁财经上网折腾并拷打fvv信息科~~

> [!WARNING]
> **免责警告**
> **可以通过此文来反推做出一些违法行为，本人不对所做出违法行为负责。**
> **确保人身安全，确保你使用之后不会被喝茶。**

> [!TIP]
> **脑子是最重要的 不会帮你搭建 不接受直接搭建（指线下）。**
> **多Bing多Google 不要使用Baidu 因为你所在的地区不太支持你这么做（类似于搭梯）。**
> **由于手上相关硬件基本都卖完了，所以基本没图。**
> **实在不行建议用现成品，百度一搜一大堆“校园网免认证绕过”，基本都是基于UDP53。**

> [!NOTE]
> **[前章（23/11）](https://www.coolapk.com/feed/51778131)写的过于乱了所以重写了，其中也有对本校网络架构的猜测。**
> **IP段50-57都是贵集团的，但是7条IP日常只见到50、51、52（现在不常见了），且上网疑似没有多线负载？57开放端口多一些其他开放端口都一样。**

> [!IMPORTANT]
> **如果是本校的：已经被发觉了，可能被限速了。所以切勿声张啦。**

## 前言

叠BUFF：本人初中基本没学，都在玩且摆烂，多一科及格就可以上普高。不接受学历歧视。如果你硬要杠，那就是你说得对。非信息系出身，纯兴趣爱好。

省流：园区大内网，没做隔离但网段和VLAN不同，故不能直接改网关。可以在内网中搭建一台代理服务器，通过代理服务器中转。

## 准备工作

1. 一台标准的x86或arm设备充当**服务端**，可运行Windows、Linux或OpenWRT（运行内存最好大于或等于512M，否则容易爆运存）。
2. OpenWRT设备[可选，充当透明代理，**客户端**]
3. **不建议任一端使用MT7621搭建，7621已经过时了。仅测试过Socks，200Mbps附近且CPU吃不满，重写前的测试环境为MT7621**。
4. **不建议任一端运存小于或等于512M，在IPQ6018、MT7981+256M+“PassWall”环境下经常爆内存导致卡死杀进程**。

**最高可用速率取决于设备和上层交换机、网关性能，以及中转会给带宽打折。比如我手上的路由普遍只有1000Mbps，没有2.5G 。**

**我没有测试过局域网内最高带宽是多少，也不知道是不是全校都在用一个千兆外网。**

## 



## 正文

### Socks或各类代理：

以下内容基于Socks，这类是转发数据。

#### 优点：

1. 用户层，无需内核模块，搭建简单
2. 纯吃CPU性能（在IPQ6018上可以跑满带宽且CPU没满）

#### 缺点：

1. 仅支持TCP、UDP传输协议，不支持ICMP类报文，会导致一些东西不可用（例如一些加速服务检测延迟不走tcping，会误报延迟）。
2. TCP延迟可能会高一些。

#### 实现效果：

1. 可吃满，巅峰下载60Mb/s（测试基于花瓣测速，设备只有1000Mbps，中转折半）
3. 日常够用，非必要无需折腾

#### 搭建：

#### 服务端：

***\*可以使用UA3F代替如下部署服务端部分。\****

1. 如果你是OpenWRT设备，前往“PassWall”中有个服务端页面，其搭建非常简单。

![image](https://github.com/user-attachments/assets/ff31e4ff-39f5-43f4-b6fc-a1c27196f4d7)

点击添加后，如果你的依赖没有问题的话，你应该会在类型里看见很多东西。

选择Socks，填写监听端口，并确保不会被占用。

当然，你也可以在这里选择用Sing-Box后再选择Socks。

身份认证勾选随意，安全性。不勾你也不会失去什么，还没看见有人会去扫IP。

**如果有“局域网访问”，请务必记得勾选。**

**创建完服务后，在用户管理前给相关服务端配置的打勾，记得勾选服务器端的启用。**

**如果连不上并且局域网内确实可以互连，请确保OpenWRT防火墙中的通信协议放行了监听端口**

在Windows/Linux环境下，使用Sing-Box可以搭建（需自行下载安装）。[配置文件示例](https://raw.githubusercontent.com/LinMoyu233/Archives/refs/heads/main/socks-config.conf)

运行命令：“sing-box run -c socks5-config.json”，其中socks5-config.json为文件名。



#### 客户端：

1. 电脑或手机选择任一代理软件（包括但不限于V2RayN、NekoBox等），服务协议选择Socks。IP填写服务端在局域网的IP，端口填写你选择的监听端口。如果你有勾选身份认证，记得填写用户名和密码。
2. OpenWRT可以透明代理局域网，使用一些同时支持Socks5的代理插件即可。

使用“SSR Plus+”时**记得修改UDP代理服务器。**

使用“PassWall”时**记得修改模式为全局代理，代理方式建议修改为tproxy。TCP转发端口修改为所有的，内存小请务必关闭日志。**

如果你的DNS默认可以被解析，可以安装“PassWall”的SmartDNS Dev版本，并且选择DNS服务SmartDNS后不要启用SmartDNS。可以让DNS直接解析防止再过一遍Socks。



### **WireGuard/OpenVPN**

#### 介绍：

WireGuard 是几乎无状态的VPN协议，切换网络零感知，不需要重新连接VPN，对于经常睡眠-唤醒的电脑特别有用。睡眠唤醒后可以立刻上网。此外 WireGuard 在 Linux 和 Windows 操作系统上均是纯内核态的实现，性能极其高。

OpenVPN相较于WireGuard性能会差，可基于TCP（WG强制要求UDP）。一般建议使用UDP搭建，不需要类似TCP一样的握手环节，否则延迟++。

#### 优点：

1. 支持ICMP类报文，兼容性++
2. 如果你在局域网内没有服务器，搭配UDP53端口绕过也是较为常见的方式

#### 缺点：

1. 速度略微逊于Socks5
2. WG在Linux系下需内核支持，否则只能使用用户态Go，性能打折（打折多少未知。在试用时遇到超时问题，写稿第一天才知系MTU问题，故未测试）
4. WireGuard的服务端“如”易搭建。在OpenWRT上有点给我绕麻了，其他平台搭建就很简单。
5. OpenVPN基于用户层且性能相较于WG差很多。在MT7981上跑到200Mbps就吃满CPU了，换用IPQ6018则是跑到200Mbps没有吃满，速度也上不去。

#### 实现效果：

1. 前文已述，WireGuard速度相较于Socks会稍微慢一点，但也够用且更完美些（50Mb/s）

#### 搭建：

#### **基于WireGuard**

1. Windows可以使用[WS4W](https://github.com/micahmo/WgServerforWindows)搭建，在Linux下可以使用Docker项目[wg-easy](https://github.com/wg-easy/wg-easy)。

#### **基于OpenVPN**

1. OpenVPN在Linux下可以用脚本[openvpn-install](https://github.com/hwdsl2/openvpn-install)，OpenWRT下有luciapp来搭建服务端。
2. 在Windows下我使用SoftEther VPN Server没有成功过，一直卡验证。网络上有不少教程。



## 一些注意事项/可能的规避检测？

1. 使用[UA2F](https://github.com/Zxilly/UA2F)/[UA3F](https://github.com/SunBK201/UA3F)修改UA规避检测。
通过修改UA来规避校园网UA检测，较为常见。UA2F需要内核支持，而UA3F是用户层，只需让你的代理软件连接跟UA3F搭建的Socks5服务器连接即可。
可以参考本文正文的Socks环节，可以省去搭建服务器环节。
但是UA3F本人在跑花瓣测速在测试上传时CPU会被进程爆满，原因未知。

2. 躲得过自动检测，躲不过人心。贵校有一套用户管理系统，居然一个IP划为一个用户。封了这个用户换个IP即可复活。在对抗了（只改IP）好一会后发现有几台机子还活着，只封跟内网流量大的机器，再过一会发现不封号了。推测是人工封号，过于高能。
![image](https://github.com/user-attachments/assets/486e9b88-647f-4346-a885-212502dd40d5)

3. 后记：现在可能被限速了。单设备对外网速率被限制在约10Mb/s，对内网不限。应该是中秋期间改的策略，当然也可能是玩客云抽风，但转念一想带宽会打折的话那客户端不应该还有10Mbs。后续有钱了可以换正儿八经的路由考虑搞多拨？
![image (1)](https://github.com/user-attachments/assets/68221f39-4e16-4e13-b652-17857f2eb05c)
![image (1)](https://github.com/user-attachments/assets/eab0074c-567f-43b4-bf60-70314b8909c2)

## 通过外挂设备使任一硬路由可用

其实就是旁路由，可用但是延迟较高。
![image (2)](https://github.com/user-attachments/assets/67f71d36-e1b4-45d1-978b-f13690dd648f)
![image (3)](https://github.com/user-attachments/assets/e74924a2-a80b-4604-80be-32e8a04cff78)

### **参考，虽然之前都没看**

> 1. [校园网白嫖思路分享：局域网中转-不花钱、不认证、高速上网 | Kenvix's Blog](https://kenvix.com/post/use-school-network-without-paying-guide/)
> 2. [记 WireGuard 部署及注意事项 | YooooEX](https://yooooex.com/2019/05/23/wireguard-deploy/)
> 3. [OpenWRT 配置 WireGuard 服务端及客户端配置教程 - 思有云 - IOIOX](https://www.ioiox.com/archives/143.html)
> 4. [rax3000m折腾记 - 双WAN下使用wireguard借线 - 哔哩哔哩 (bilibili.com)](https://www.bilibili.com/read/cv34606771/)



### **斐讯N1、我家云等旁路由类 参考**

> 1. [20块的玩客云，刷机并解锁NAS+软路由+智能家居服务器！_哔哩哔哩_bilibili](https://www.bilibili.com/video/av1703245218/)
> 2. [瞎折腾 - 斐讯 N1 变身 Armbian Linux 记录 | YooooEX](https://yooooex.com/2019/01/28/n1-armbian/)




### **感觉挺厉害的**

1. [GitHub - dnomd343/XProxy: 虚拟旁路由网关，支持内网设备IPv4与IPv6双栈透明代理](https://github.com/dnomd343/XProxy) 

看着挺厉害的，有点看不懂，懒得折腾。



如果某些人看见：**把极域自带功能再拉出来说是新机房特点，你这个面子工程？？**