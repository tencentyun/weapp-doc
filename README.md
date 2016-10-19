# 小程序创建资源配置指引

假如您已成功创建了小程序资源，现在需要对现有的资源进行一些简单配置后，才能让小程序跑起来
>未创建过资源的用户可以先在[小程序控制台](https://console.qcloud.com/la)进行创建

接下来就跟着文档一步步来操作吧

## 1.配置微信小程序通信域名

首先我们在[小程序资源视图](https://console.qcloud.com/la)中将免费二级域名拷贝下来

![小程序资源视图](https://mc.qcloudimg.com/static/img/95d83cc575c1aabc66cbdcaf63bc8619/18.png)

然后前往[微信公众平台](https://mp.weixin.qq.com) -【选择设置】- 【开发者设置】选项卡，使用二级域名完成通信域名设置

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

2) 如果未安装`npm`可以先安装[Node.js](https://nodejs.org/en/)，会将`npm`打包一起安装好

3) 使用`npm`安装包管理器`bower`，执行下面命令

```
npm install bower -g
```

4) 进入Demo根目录，使用`bower`下载依赖的 SDK，
>因为 npm 下载的`node_modules`目录在微信开发者中工具打包时会被忽略，所以这里 Demo 中使用`bower`来管理 SDK 模块

```
bower install
```

修改 Demo 根目录下的`config.js`配置文件

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
如果现有的配置满足不了您的业务需求，我们提供了[中型解决方案](https://github.com/CFETeam/weapp-doc/blob/master/medium_solution.md)、[大型解决方案](https://github.com/CFETeam/weapp-doc/blob/master/large_solution.md)对现有资源进行升级。

