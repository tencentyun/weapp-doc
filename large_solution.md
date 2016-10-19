# 大型扩容解决方案

对于小型的程序应用，通常在初始阶段访问量会维持在一个较低的水平。但随着访问量上来，虽然可以对单台服务器进行配置升级，但是到后期还是会遇到单机性能瓶颈。所以对于大型应用来说，我们可以考虑将云服务器横向扩容，通过负载均衡技术对外提供服务，可以有效解决单机性能瓶颈以及单点故障问题。

我们的[微信小程序解决方案](https://console.qcloud.com/la)的初始架构如下图：业务和会话管理服务器都只有一台。

![原架构图](https://mc.qcloudimg.com/static/img/822152039dca7161ad7217c1037066a8/19.png)

通过外网、内网负载均衡绑定扩容后的云服务器后，小程序的架构升级如下图所示：

![升级架构图](https://mc.qcloudimg.com/static/img/143a3113449c6af1c52733613cbbc546/20.png)

接下来我们具体来操作下`会话管理服务器`以及`业务服务器横`向扩容

>注意所有新创建的资源与存量资源的<font color="red">地域、分区、项目</font>需要保持一致

## 会话管理服务器扩容

### 1. 制作会话管理服务器镜像

首先需要登录[腾讯云CVM控制台](https://console.qcloud.com/cvm)，在会话管理CVM实例右侧操作栏，点击【更多】-【制作镜像】。

![制作镜像](https://mc.qcloudimg.com/static/img/6c42ba12fad765d5ee56ebd0ef557ae5/8.png)

填写好`镜像名`和`说明`后点击`确定`，接下来会提示该服务器的镜像正在创建中

![镜像制作中](https://mc.qcloudimg.com/static/img/3f3c4431a7567eba25e8635e788e59ba/9.png)

### 2. 基于镜像创建会话管理服务器

创建好镜像后，我们进入[镜像控制台](https://console.qcloud.com/cvm/image)，选中镜像点击创建云主机

![镜像控制台](https://mc.qcloudimg.com/static/img/1713a78dc7b759884cfb0e32f15b91eb/10.png)

在[云服务器购买页](https://buy.qcloud.com/cvm)使用我们之前创建好的镜像来创建云服务器

![购买选择镜像](https://mc.qcloudimg.com/static/img/6f4880fa18431780d760e65ab436e04e/11png.png)

### 3. 创建内网负载均衡

首先需要登录[负载均衡购买页](https://buy.qcloud.com/lb)，选择 `内网类型` `微信小程序`项目下负载均衡

![购买负载均衡](https://mc.qcloudimg.com/static/img/9c39b5a4ab00d5dffd2c1afb6c3e6c91/12.png)

创建完后在[负载均衡控制台](https://console.qcloud.com/loadbalance/index)找到刚创建好的负载均衡实例，点击`实例ID`进入实例详情，点击`绑定云主机`，勾选用于会话管理的云服务器（包括之前创建好的），点击`确定`进行绑定

![绑定云主机](https://mc.qcloudimg.com/static/img/1130782b0aee1e24ab721f9dd5d2c2eb/13.png)

创建监听器，监听协议`TCP`端口填业务服务器中访问的端口，后端端口填会话服务的端口

![监听器](https://mc.qcloudimg.com/static/img/32870a7be01e46d352948e21cabdccf7/15.png)

### 4. 修改业务服务器配置

配置文件`CentOS`系统在`/etc/qcloud`路径下，`Windows Server`系统`c://qcloud`路径下

修改`sdk.config`中的`authServerUrl`字段值，

```
"authServerUrl": "http://ip:port/", //ip填负载均衡的VIP，端口填负载均衡监听器创建时填的TCP端口
```

![负载均衡ip](https://mc.qcloudimg.com/static/img/9e5600b9e52677566c3139e86a40087f/14.png)

到此我们已经完成了对`会话管理服务器`的横向扩容，接下我们进行`业务服务器`的扩容

## 业务服务器扩容

### 1. 制作业务服务器镜像

制作流程可以参考`制作会话管理服务器镜像`流程，注意一点的是制作Windows Server系统镜像可能稍慢点，请耐心等待

### 2. 基于镜像创建业务服务器

创建流程可以参考`基于镜像创建会话管理服务器`流程，注意`地域``分区``项目`需要保持一致

### 3. 配置外网负载均衡

云主机创建好后，我们只需在[负载均衡控制台](https://console.qcloud.com/loadbalance/index)实例详情中，将新创建的云服务器绑定到负载均衡，流程可以参考上面的会话管理服务器绑定操作

到此为止我们的横向扩容操作已经全部完成

## 更智能的自动化扩容方案

用户明确何时需要扩缩容，则可提前设置 Auto Scaling 定时策略。到相应时间时，系统将自动添加或减少 CVM 实例，无需人工操作
通过告警策略新增的 CVM 实例还可直接关联已有负载均衡 CLB，以使伸缩组新增的实例承担分发流量，从而提高服务可用性。具体可查看[弹性伸缩](https://www.qcloud.com/doc/product/377/3154)文档