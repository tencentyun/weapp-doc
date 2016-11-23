# 会话管理服务BUG修复与升级

## 程序BUG修复与代码升级

### 针对2016年11月9号之前(包括11月9号)购买腾讯云微信小程序解决方案的用户

1)通过公测发现小程序会话管理服务器的两处BUG

1.1)会话管理服务器程序bug导致鉴权失败。

1.2)如果用户昵称中包含emoji表情，会话管理服务无法解析。

1.3)数据库优化升级

现在两处bug已经修复，现在给出升级解决方案：

2.1)腾讯云微信小程序资源视图页(https://console.qcloud.com/la)

查看会话管理服务器的IP信息。如图1所示。点击进去，查看IP信息，如图2所示。

![image](https://cloud.githubusercontent.com/assets/12195370/20167547/37ad1042-a757-11e6-878d-54371a3be01e.png)
                                               
					                                        图1 微信小程序资源视图
					  
![image](https://cloud.githubusercontent.com/assets/12195370/20167593/6aa45b90-a757-11e6-8348-7d48e1e9b3f3.png)

                                                 图2 会话管理服务器IP地址
					       
2.2)使用拿到的IP信息，使用xshell或者其他工具登录至会话管理服务器，执行以下shell命令，完成升级。(注意：登录服务器的时候只能使用22端口)

     wget 'http://mina-1252789078.costj.myqcloud.com/update_mina.sh'  #下载程序升级脚本

     chmod 755 update_mina.sh  #修改程序升级脚本的权限

     ./update_mina.sh    #执行代码升级

注意：原来的会话管理程序已经被覆盖，原来的代码备份在当前的目录下以日期命名的目录下面，例如161110。在这个目录下的代码是不安全的，如果升级前的代码还想保留，建议手动备份到其他目录。

2.3）数据库升级命令

	wget 'http://mina-1252789078.costj.myqcloud.com/upgradeDb.sh'
	
	chmod 755 upgradeDb.sh
	
	./upgradeDb.sh
	
考虑到数据库的安全性和扩展性问题将数据库进行升级，原来表(cSessioninfo)中的数据将会被迁移到新的数据表(cSessionInfo)中，用户无需考虑数据迁移问题，只要执行上述三条shell命令即可。
