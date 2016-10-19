# 小程序创建资源配置指引

假如您已成功创建了小程序资源，现在需要对现有的资源进行一些简单配置后，才能让小程序跑起来
>未创建过资源的用户可以先在[小程序控制台](https://console.qcloud.com/la)进行创建

接下来就跟着文档一步步来操作吧

## 1.配置微信小程序通信域名

![小程序资源视图](https://mc.qcloudimg.com/static/img/9f89b4234acaef53819dd687b93c79c3/18.png)

首先我们在[小程序资源视图](https://console.qcloud.com/la)中将腾讯云分配的 免费二级域名 拷贝下来

![微信域名配置](https://mc.qcloudimg.com/static/img/79cdbc773a6030b6055951d72e8735e3/16.png)

然后前往[微信公众平台](https://mp.weixin.qq.com) -【选择设置】- 【开发者设置】选项卡，使用腾讯云分配好的二级域名完成通信域名设置

- `request合法域名`: 填腾讯云分配的二级域名，需要带上`https`协议
- `socket合法域名`: 填`wss://www.qcloud.la`
- `uploadFile合法域名`: 如果您基于二级域名搭建上传服务则可填二级域名，否则填自其它上传域名
- `downloadFile合法域名`: 如果您基于免费二级域名搭建资源下载服务则可填二级域名，否则填写其它下载域名

## 2.修改业务服务器配置

>如果您购买的是 `Centos` 系统云服务器，创建资源时已默认为您下发好配置到 `/etc/qcloud/sdk.config`，可略过此步

Windows Server 系统修改 `c:\qcloud\sdk.config`。

```
{
    "serverHost": "xxxx.qcloud.la", //资源视图给出的二级域名
    "authServerUrl": "http://内网ip:80/", //会话管理服务器的内网ip，以及会话服务启动的端口，初始程序端口是80
    "tunnelServerUrl": "https://ws.qcloud.com", //不用修改
    "tunnelSignatureKey": "62aaa14292b3a65a61c14b8c30437bc648e087b2" //填写一份随机字符
}
```

修改完成后，需要重启 IIS 中的网站以生效。

## 3.下载微信客户端demo和SDK

1) 前往[github](https://github.com/CFETeam/weapp-client-demo)将demo下载到本地

2) 如果未安装`npm`可以先安装[Node.js](https://nodejs.org/en/)，会将`npm`打包一起安装好

3) 使用`npm`安装包管理器`bower`，执行下面命令。

```
npm install bower -g
```

4) 进入demo根目录，使用`bower`下载依赖的SDK，
>因为npm下载的`node_modules`目录在微信开发者中工具打包时会被忽略，所以这里demo中使用`bower`来管理SDK模块

```
bower install
```

修改根目录下的`config.js`配置文件

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

5) 微信开发者工具导入demo工程目录，然后点击调试即可打开新片预告Demo开始体验

![小程序demo](https://mc.qcloudimg.com/static/img/05f7d737bc4dc74021aa5db49bf66aa0/17.png)

## 4.升级方案
如果现有的配置满足不了您的业务需求，我们提供了[中型解决方案](xxxx)、[大型解决方案](xxxx)对现有资源进行升级。

