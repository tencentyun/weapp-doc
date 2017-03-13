# 集群版架构扩容

对于小型的程序应用，通常在初始阶段访问量会维持在一个较低的水平。但随着访问量上来，虽然可以对单台服务器进行配置升级，但是到后期还是会遇到单机性能瓶颈。所以对于大型应用来说，我们可以考虑将云服务器横向扩容，通过负载均衡技术对外提供服务，可以有效解决单机性能瓶颈以及单点故障问题。

我们的[微信小程序解决方案](https://console.qcloud.com/la)的初始架构如下图：业务和会话管理服务器都只有一台。

![单机版架构图](https://mc.qcloudimg.com/static/img/7d40df0347cbfad83c16a011a435271c/24.png)

通过外网、内网负载均衡绑定扩容的云服务器后，业务和会话管理服务器都可形成如下所示集群架构：

![集群版架构图](https://mc.qcloudimg.com/static/img/6031cecf849a1b43174244b3189dad0c/25.png)

接下来我们具体来操作下`会话管理服务器`以及`业务服务器`横向扩容

## 会话管理服务器扩容

>注意所有新创建的资源与存量资源的<font color="red">地域、可用区、项目</font>需要保持一致

### 1. 制作会话管理服务器镜像

首先需要登录[腾讯云CVM控制台](https://console.qcloud.com/cvm)，在会话管理CVM实例右侧操作栏，点击【更多】-【制作镜像】。

![制作镜像](https://mc.qcloudimg.com/static/img/6c42ba12fad765d5ee56ebd0ef557ae5/8.png)

填写好`镜像名称`和`镜像描述`后点击`确定`，接下来会提示该服务器的镜像正在创建中

![镜像制作中](https://mc.qcloudimg.com/static/img/3f3c4431a7567eba25e8635e788e59ba/9.png)

### 2. 基于镜像创建会话管理服务器

创建好镜像后，我们进入[镜像控制台](https://console.qcloud.com/cvm/image)，选中镜像点击创建云主机

![镜像控制台](https://mc.qcloudimg.com/static/img/1713a78dc7b759884cfb0e32f15b91eb/10.png)

在[云服务器购买页](https://buy.qcloud.com/cvm)使用我们之前创建好的镜像来创建云服务器

![购买选择镜像](https://mc.qcloudimg.com/static/img/6f4880fa18431780d760e65ab436e04e/11png.png)

### 3. 创建内网负载均衡

首先需要登录[负载均衡购买页](https://buy.qcloud.com/lb)，选择 `内网类型` `微信小程序`项目下负载均衡

![购买负载均衡](https://mc.qcloudimg.com/static/img/9c39b5a4ab00d5dffd2c1afb6c3e6c91/12.png)

创建完后在[负载均衡控制台](https://console.qcloud.com/loadbalance/index)找到刚建好的负载均衡实例，点击`实例ID`进入实例详情，点击`绑定云主机`，勾选用于会话管理的云服务器（包括之前创建好的），点击`确定`进行绑定

>注意只要绑定的云服务器开启了安全组，必须将负载均衡 VIP 加入到后端服务器的放通规则中
>左侧列表中显示的是未被隔离、未过期、带宽不为0的可选云服务器列表

![绑定云主机](https://mc.qcloudimg.com/static/img/1130782b0aee1e24ab721f9dd5d2c2eb/13.png)

创建负载均衡监听器

- 监听协议选`TCP`，监听端口即业务服务器访问会话管理服务器用的端口(配置文件中默认是**80**)
- 后端端口填会话服务的端口(默认是**80**)

![监听器](https://mc.qcloudimg.com/static/img/a5dd10bac2f3629ed78fa013a7d312fe/21.png)

### 4. 修改业务服务器配置

配置文件`CentOS`系统在`/etc/qcloud`路径下，`Windows Server`系统在`c://qcloud`路径下

修改`sdk.config`中的`authServerUrl`字段值，

```
"authServerUrl": "http://VIP/mina_auth/", //VIP上个步骤创建负载均衡实例的VIP
```

重启服务来生效配置
- Node.js环境，进入`/data/release/node-weapp-demo`下执行`pm2 start process.json`
- .Net环境 需要重启 IIS 中的网站来生效配置
- JAVA环境 重启tomcat执行命令`systemctl restart tomcat`

![负载均衡ip](https://mc.qcloudimg.com/static/img/9e5600b9e52677566c3139e86a40087f/14.png)

到此我们已经完成了对`会话管理服务器`的横向扩容，接下我们进行`业务服务器`的扩容

## 业务服务器扩容

>注意所有新创建的资源与存量资源的<font color="red">地域、可用区、项目</font>需要保持一致

### 1. 制作业务服务器镜像

制作流程可以参考`制作会话管理服务器镜像`流程

>注意选择业务服务器实例来进行镜像制作
>如果制作Windows Server系统镜像的话可能稍慢点，请耐心等待

### 2. 基于镜像创建业务服务器

创建流程可以参考`基于镜像创建会话管理服务器`流程
>注意`地域``可用区``项目`需要保持一致，镜像则选择业务服务器镜像

### 3. 配置外网负载均衡

云主机创建好后，我们只需在[负载均衡控制台](https://console.qcloud.com/loadbalance/index)实例详情中，将新创建的云服务器绑定到负载均衡，流程可以参考上面的会话管理服务器绑定操作

到此为止我们的横向扩容操作已经全部完成


#配置更智能的弹性伸缩策略
我们推荐您为小程序的业务服务器集群及会话服务器集群配置上弹性伸缩AS（auto scaling）策略，充分利用公有云“灵活”“弹性”的优势。

##为什么配置弹性伸缩？
以下两个场景下弹性伸缩能降低成本和提高业务连续性：

1、 小程序访问有明显的高峰低谷：根据测算，如果业务服务器集群和会话服务器集群需要不只一台机器，且高峰短于8个小时，采用“按闲时保留固定服务器+高峰时期增加临时服务器”的方式，能节约30%左右成本。您通过AS设置定时扩缩容任务，让腾讯云在忙时扩容临时服务器，闲时回收终止未充分利用的服务器。

2、 小程序访问量稳定的预期下，配置基于监控告警的伸缩策略可应对意外高负载，保障服务的持续可用，给问题解决争取时间。异常高负载包括[CC攻击](http://baike.baidu.com/link?url=aSNcL5Q_xzDxPvFYRU3qbS11NIQXD5vwvI5yxtJTVlL0xhjAaLntwmDHVW8buUlH4bbNJqMzCPp8b1N2LX-OnwAUR3MnE9GhH-F7fomUac3)，以及意外流量（类似“脸萌”刚上线的远超预期的传播速度，或特定事件带来的突发访问）。可参看公益网站[“宝贝回家”](https://www.qcloud.com/community/article/651089001483090830)的案例。

>注：弹性伸缩能力免费，扩容的CVM按秒正常计费。

##弹性伸缩会做哪些事情？
1、 定时给集群增加机器或减少机器

2、 根据集群服务器的负载情况自适应地增加机器或减少机器

3、 增加的机器会自动注册到负载均衡中，实现全自动扩容


##为会话服务器配置弹性伸缩策略
###1、创建启动配置
扩容时以启动配置为模板创建机器，因此我们事先通过启动配置指定地域、机型、镜像。


登录[弹性伸缩控制台](https://console.qcloud.com/autoscaling/config)，点击导航条中的【启动配置】。

选择项目和地域：
![](https://mc.qcloudimg.com/static/img/653ebf516d940a90fd79728e5d319cdc/image.png)
这里要注意必须选择小程序所在的项目和地域。

接下来的操作与购买机器类似。您可跟着指引完成启动配置创建。

![](https://mc.qcloudimg.com/static/img/4cecf25e8ad9caa67271159c67d0b770/image.png)


>注意：此前您已经做好了镜像，这里需确保镜像里的应用能随操作系统启动的，这样扩容出来的机器就能直接工作，无需人工介入。
![](https://camo.githubusercontent.com/c58d92b133f44b3d70a0936cf4d6f087e7e0d3ee/68747470733a2f2f6d632e71636c6f7564696d672e636f6d2f7374617469632f696d672f33663363343433316137353637656261323565383633356537383865353962612f392e706e67)



###2、 创建伸缩组

在弹性伸缩的[【控制台】](https://console.qcloud.com/autoscaling)，点击【新建】，按如下填写集群的管理信息：

【名称】：按需起一个名字。这里填“会话服务器集群”

【最小伸缩数】：集群服务器数量的下限，这里填0即可

【起始实例数】：伸缩组刚创建时，自动创建的机器数量。这里填0即可.

【最大伸缩数】：集群服务器数量的上限，这里按需填写

【启动配置】：选择刚才您创建的启动配置

【支持网络】：会话服务器的网络环境，一般选“基础网络”即可

【支持可用区】：即选择机扩容器落在哪个可用区里，此处按会话服务器所在的可用区勾选即可

【移出策略】：选择默认即可

【负载均衡】：选择会话服务器的负载均衡，即刚才创建的

![](https://mc.qcloudimg.com/static/img/f665314e51db863d3f57cd75534f69f6/932.jpg)

最后点击“确定”，完成创建。

###3、 添加现有机器进伸缩组

在[【控制台】](https://console.qcloud.com/autoscaling)点击伸缩组名字，进入管理页，在页面下方点击“添加云主机”，

![](https://mc.qcloudimg.com/static/img/8ed547b6d545cff5b6e22cd71a75402c/08.jpg)

在弹出的对话框中选择已有的会话服务器加入伸缩组。
![](https://mc.qcloudimg.com/static/img/5c91d826a3aab5bbb478a1c0524302e8/08113043.jpg)

加入后对服务器设置“免于缩容”，这样在缩容活动中，伸缩组不会选择这台服务器缩容。
![](https://mc.qcloudimg.com/static/img/62319473a1a05e98d51c64c22ca24424/0308113553.jpg)

###4、 设置扩容策略
您可以选择定时扩容、或者基于告警动态扩容。通常扩容任务和缩容任务成对出现。

![](https://mc.qcloudimg.com/static/img/41763806c8d05ae89128b5a87e772974/08121006.jpg)

**a. 定时扩容**：

比如一个点餐小程序，可预期午饭时间的负载比其他时间高，就可以设置每天11：00-13：00扩容两台额外服务器支撑负载。


先设置一个定时扩容任务：

![](https://mc.qcloudimg.com/static/img/d276c8d6924b4126a1532ddcefae8f0c/0170308120453.jpg)

再设置一个定时缩容任务：

![](https://mc.qcloudimg.com/static/img/ae7b0f21529d9f483a455e6148594926/20170308120822.jpg)

**b. 基于告警扩容**：

预期无明确的扩容，但需要防范出现意料之外的流量/攻击：

先设置一个告警扩容策略，用于应对异常流量：
![](https://mc.qcloudimg.com/static/img/d23dde14b8e12241d3315286682c2d8d/455.jpg)

再设置一个告警缩容策略，用于清退未充分利用的服务器：
![](https://mc.qcloudimg.com/static/img/92b7fccdfb2b863e4798574e0cb06bde/22630.jpg)


##为业务服务器配置弹性伸缩策略

这个过程与会话服务器类似：

- 为业务服务器创建一个启动配置
- 为业务服务器创建一个伸缩组“业务服务器集群”，指向业务服务器的负载均衡
- 为业务服务器设置伸缩策略

##验证伸缩性和查看伸缩活动
将伸缩组的期望实例数+1，伸缩组会自动扩容一台服务器到集群中。如果新扩的机器能正常处理请求，说明伸缩组已正常工作。
![](https://mc.qcloudimg.com/static/img/665e029c6abfa6a7ef3f9063c88df486/05.jpg)

伸缩组还支持[查询历史伸缩活动](https://www.qcloud.com/document/product/377/3804)，确保您完全掌控伸缩活动情况。

至此，您的小程序已经具备了智能扩容的能力。您无需为扩容缩容的问题再操心，留意伸缩组通知或者不定期查看历史伸缩活动即可。
其他问题具体可查看[弹性伸缩](https://www.qcloud.com/doc/product/377/3578)文档。 
