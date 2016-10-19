# 小程序创建资源配置指引

假如您已成功创建了小程序资源，需要对现有的资源进行一些简单配置后，才能让小程序跑起来
>未创建过资源的用户可以先在[小程序控制台](https://console.qcloud.com/la)进行创建

## 1.配置微信小程序通信域名

首先我们在[小程序资源视图](https://console.qcloud.com/la)中将免费二级域名拷贝下来，在后面的几个流程中都会用到。

![小程序资源视图](https://mc.qcloudimg.com/static/img/95d83cc575c1aabc66cbdcaf63bc8619/18.png)

然后前往[微信公众平台](https://mp.weixin.qq.com) -【选择设置】- 【开发者设置】选项卡，使用二级域名完成通信域名设置，设置完后可能要等待几分钟生效

![微信域名配置](https://mc.qcloudimg.com/static/img/79cdbc773a6030b6055951d72e8735e3/16.png)

- `request合法域名`：填腾讯云分配的二级域名
- `socket合法域名`：填`ws.qcloud.com`
- `uploadFile合法域名`：基于二级域名搭建上传服务填腾讯云分配的二级域名
- `downloadFile合法域名`：基于二级域名搭建资源下载服务则填腾讯云分配的二级域名

## 2.修改业务服务器配置

>如果开发语言环境是`CetOS`操作系统，创建资源时已默认下发好配置到`/etc/qcloud/sdk.config`，可略过此步

Windows Server系统修改`c://qcloud`下`sdk.config`文件

```
{
    "serverHost": "xxxx.qcloud.la", //资源视图给出的二级域名
    "authServerUrl": "http://内网IP:80/", //会话管理服务器的内网IP
    "tunnelServerUrl": "https://ws.qcloud.com", //不用修改
    "tunnelSignatureKey": "62aaa14292b3a65a61c14b8c30437bc648e087b2" //填写一份随机字符
}
```

修改完成后，需要重启 IIS 中的网站以生效。

## 3.下载微信小程序 Demo 和 SDK

1) 前往[github](https://github.com/CFETeam/weapp-client-demo)将 Demo 下载到本地

2) 修改 Demo 根目录下的`config.js`配置文件

```
var host = 'www.qcloud.la'; //host替换成微信小程序资源视图中分配的二级域名

var config = {
    service: {
        host,
        loginUrl: `https://${host}/login`,
        tunnelUrl: `https://${host}/tunnel`
    }
};

module.exports = config;
```

5) 微信开发者工具导入Demo工程目录，然后点击调试即可打开聊天室Demo开始体验

![小程序Demo](https://mc.qcloudimg.com/static/img/05f7d737bc4dc74021aa5db49bf66aa0/17.png)

## 4.升级方案
如果现有的配置满足不了您的业务需求，我们提供了[单机版架构升级](https://github.com/CFETeam/weapp-doc/blob/master/medium_solution.md)、[集群版架构扩容](https://github.com/CFETeam/weapp-doc/blob/master/large_solution.md)来对现有资源进行配置升级、扩容。

## 常见问题

### 重装开发语言环境

> 目前业务服务器提供了PHP、Node.js、Java、.Net版本的语言环境，用户如果要切换需要做以下操作:

1) 备份配置文件

将`sdk.config`从服务器拷贝到本地、`CentOS`系统在`/etc/qcloud`路径下，`Windows Server`系统在`c://qcloud`路径下

2) 重装系统

> 如果有重要数据请提前保存好

首先需要登录[腾讯云CVM控制台](https://console.qcloud.com/cvm)，在会话管理CVM实例右侧操作栏，点击【更多】-【重装系统】。

弹出框内镜像来源选`服务市场`，镜像选`基础环境`，下拉列表中找到四个语言的镜像来选中镜像，设置好系统密码后点`开始重装`

![重选系统镜像](https://mc.qcloudimg.com/static/img/57da50a2f470d7186020b4e39f5ea15a/22.png)

3) 上传配置文件

系统重装好后将步骤1中保存下来的`sdk.config`上传至服务器上，`CentOS`系统在`/etc/qcloud`路径下，`Windows Server`系统在`c://qcloud`路径下

4) 重启服务

- Node.js环境，进入`/data/release/node-weapp-demo`下执行`pm2 process.json`
- .Net环境 需要重启 IIS 中的网站来生效配置
- JAVA环境 重启tomcat执行命令`systemctl restart tomcat`

