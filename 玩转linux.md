# 玩转linux

# 环境搭建

## 基础环境

### 设置源

不同linux版本的源设置不同，需要根据版本设置，参考[脚本]()
  
### 更新系统

- apt-get update  // 更新软件资源列表
- apt-get upgrade // 更新已安装的软件，会同时升级系统内核，可选
  
### 基础软件安装

- apt-get install vim gcc
  
### 特定开发软件安装

#### node
  
```
apt-get install node
```

如果需要使用http server

```
npm install http-server -g
http-server ./  -p 80
```
    
如果报错如下，这是因为没有node的执行文件是nodejs，创建对应的软链接

```
/usr/bin/env: node: No such file or directory
```

```
ln -s /usr/bin/nodejs /usr/bin/node
```
    
## 桌面版环境

### 搜狗输入法
在官方网站下载对应的deb包，然后在命令行进行安装，如果安装错误，执行下面脚本，重启电脑，然后在输入法管理器中选择非当前语言的输入法，找到搜狗输入法  

```
apt-get install -f
```

### 无线网卡

尽量插上无线网卡之后再安装操作系统，手动安装目前还没成功

# 基本操作

#### 后台运行程序

```
nohup java -jar test.jar &
```

nohup 意思是不挂断运行命令,当账户退出或终端关闭时，程序仍然运行，日志输出默认文件：nohup.out

#### 键盘对应

^对应ctrl

#### 磁盘挂载

[Ubuntu Linux 永久挂载(mount)分区](http://www.linuxidc.com/Linux/2014-04/100488.htm)

```
mkfs.ext3 -j /dev/xvdf
mount /dev/xvdf /usr3
```

# FAQ

#### 开机启动执行命令

```
crontab -e
@reboot /path/to/script
```

备用方法：编辑/etc/rc.local，添加命令。

参考[How to run scripts on start up?](https://askubuntu.com/questions/814/how-to-run-scripts-on-start-up)

#### error while loading shared libraries: libssl.so.10: cannot open shared object file

可以删除对应的ssl，如果出现错误，针对错误继续remove，完之后用apt-get install ssh重新安装对应包

```
apt-get remove libssl1.0.0 libssl-dev
```

#### 保证apt版本和source.list中一致
```
apt-get install --only-upgrade apt
```
安装特定版本或者降级参考：[How to install specific version of some package? ](https://askubuntu.com/questions/428772/how-to-install-specific-version-of-some-package/428778)
