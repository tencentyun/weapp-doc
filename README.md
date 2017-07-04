# 小程序初始化配置指引

假如您已成功创建了小程序资源，需要对现有的资源进行一些简单配置后，才能让小程序跑起来
>未创建过资源的用户可以先在[小程序控制台](https://console.qcloud.com/la)进行创建


## 1.配置微信小程序通信域名

首先我们在[小程序资源视图](https://console.qcloud.com/la)中将二级域名拷贝下来，在后面的几个流程中都会用到。

![小程序资源视图](https://mc.qcloudimg.com/static/img/95d83cc575c1aabc66cbdcaf63bc8619/18.png)

然后前往[微信公众平台](https://mp.weixin.qq.com) -【开发】-【基本配置】-【服务器配置】-【修改配置】,使用二级域名完成通信域名设置，设置完后可能需要稍等几分钟重启微信开发者工具生效。

![微信域名配置](https://imgcache.qq.com/open_proj/proj_qcloud_v2/wechat_mc/css/img/doc/1.jpg)

- `request合法域名`：填腾讯云分配的二级域名
- `socket合法域名`：填腾讯云分配的 Socket 域名，如 `12345678.ws.qcloud.la`
- `uploadFile合法域名`：填腾讯云分配的二级域名
- `downloadFile合法域名`：填腾讯云分配的二级域名


## 2.修改业务服务器配置

>如果开发语言环境是`CentOS`操作系统，创建资源时已默认下发好配置到`/etc/qcloud/sdk.config`，可略过此步

>登录云服务器的密码请在[站内信](https://console.qcloud.com/message)、手机短信、邮箱中查看

Windows Server系统修改`c://qcloud`下`sdk.config`文件

```
{
    "serverHost": "xxxx.qcloud.la", //资源视图给出的二级域名
    "authServerUrl": "http://内网IP/mina_auth/", //内网IP改成会话管理服务器的内网IP
    "tunnelServerUrl": "https://xxxx.ws.qcloud.la", //不用修改
    "tunnelSignatureKey": "62aaa14292b3a65a61c14b8c30437bc648e087b2", //填写一份随机字符
    "networkTimeout": "30000"
}
```

修改完成后，需要重启 IIS 中的网站来生效。


## 3.下载微信小程序 Demo 和 SDK

1) 前往[github](https://github.com/tencentyun/qcloud-weapp-client-demo)将 Demo 下载到本地

2) 修改 Demo 根目录下的 config.js 配置文件里主机配置

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

5) 微信开发者工具导入 Demo 工程目录，然后点击调试即可打开聊天室 Demo 开始体验

>开发者工具下载地址：[windows 64](https://servicewechat.com/wxa-dev-logic/download_redirect?type=x64&from=mpwiki&t=1476434677599)、[windows 32](https://servicewechat.com/wxa-dev-logic/download_redirect?type=ia32&from=mpwiki&t=1476434677599)、[mac](https://servicewechat.com/wxa-dev-logic/download_redirect?type=darwin&from=mpwiki&t=1476434677599)

![小程序Demo](https://imgcache.qq.com/open_proj/proj_qcloud_v2/wechat_mc/css/img/doc/3.jpg)


## 4.升级方案
如果现有的配置满足不了您的业务需求，我们提供了[单机版架构升级](https://github.com/tencentyun/weapp-doc/blob/master/medium_solution.md)、[集群版架构扩容](https://github.com/tencentyun/weapp-doc/blob/master/large_solution.md)来对现有资源进行配置升级、扩容。


## 常见问题

### 微信 AppId 和 AppSecret 在购买时填写错误怎么办

如果在购买解决方案时，把 AppId 和 AppSecret 填写错误。小程序用户在登录时，便会返回错误码40029，错误信息MA_WEIXIN_CODE_ERR。此时便需要手动修改 AppId 和 AppSecret

修改步骤如下：

1) 登录会话管理服务器，进入`/opt/lampp/htdocs/mina_auth/system/db/`目录，查看`db.ini`文件，文件中保存着 云数据库 CDB 的ip、port、username、passwd以及 dbname 等信息。

1.1) 进入CDB配置文件目录

     cd /opt/lampp/htdocs/mina_auth/system/db/
     
1.2) 查看服务器配置文件

     vi db.ini

2) 拿到这些信息以后，登录云数据库 CDB，修改 cAppinfo 表中的 AppId 和 secretKey 即可。

2.1)进入安装mysql命令的目录

    cd /opt/lampp/bin/
    
2.2)连接CDB

    ./mysql -h #ip -P #port -u #username -p #passwd(其中#ip、#port、#username、#passwd是在1.2步骤中查看到的具体信息)
    
2.3)更新AppId 和 AppSecret

    use cAuth;//选中数据库。
    
	update cAppinfo set appid = "your appid",secret = "your secret";//更新正确的AppId 和 AppSecret

### 创建资源时填的AppID和AppSecret在哪找到？

要查看 AppID，请前往[微信公众平台](https://mp.weixin.qq.com) -【选择设置】- 开发者设置】在`开发者ID`一栏中可以看到。AppSecret 是小程序私有密钥，微信不再保存，无法查看，只能重置。重置后请妥善保管，并参考上面的流程修改 Wafer 服务中保管的版本。

### 一站式构建小程序分配的CVM/CDB密码哪里获取？

分配的服务器及数据库资源的密码请在[站内信](https://console.qcloud.com/message)、手机短信、邮箱中可以获取到


### 重装开发语言环境

> 目前业务服务器提供了PHP、Node.js、Java、.NET版本的语言环境，用户如果要切换需要做以下操作:

1) 备份配置文件

将`sdk.config`从服务器拷贝到本地、`CentOS`系统在`/etc/qcloud`路径下，`Windows Server`系统在`c://qcloud`路径下

2) 重装系统

> 如有重要数据请提前保存好

首先需要登录[腾讯云CVM控制台](https://console.qcloud.com/cvm)，在会话管理CVM实例右侧操作栏，点击【更多】-【重装系统】。

弹出框内镜像来源选`服务市场`，镜像选`基础环境`，下拉列表中找到四个语言的镜像来选中镜像，设置好系统密码后点`开始重装`

![重选系统镜像](https://mc.qcloudimg.com/static/img/57da50a2f470d7186020b4e39f5ea15a/22.png)

3) 上传配置文件

系统重装好后将步骤1中保存下来的`sdk.config`上传至服务器上，`CentOS`系统在`/etc/qcloud`路径下，`Windows Server`系统在`c://qcloud`路径下

4) 重启服务

- Node.js环境进入`/data/release/node-weapp-demo`下执行`pm2 start process.json`
- .NET环境需要重启 IIS 中的网站来生效配置
- Java环境重启tomcat执行命令`systemctl restart tomcat`
