# SSH协议理解

SSH（Secure SHell protocol）：安全Shell程序协议，基于TCP的应用层协议

# 原理

## 非对称加密

主要是透过两把不一样的公钥与私钥 (Public and Private Key) 来进行加密与解密的过程

- 公钥 (public key)：提供给远程主机进行数据加密的行为，也就是说，大家都能取得你的公钥来将数据加密的意思；
- 私钥 (private key)：远程主机使用你的公钥加密的数据，在本地端就能够使用私钥来进行解密。由于私钥是这么的重要， 因此私钥是不能够外流的！只能保护在自己的主机上

每部主机都有自己的密钥（公钥和私钥），且公钥用来加密而私钥用来解密， 其中私钥不可外流。但因为网络联机是双向的，所以，每个人应该都要有对方的公钥。

> 加密技术众多，存在不同的优缺点，有的指令周期快，但不安全，有的安全但加密解密速度慢。
> SSH采用RSA/DSA/DiffieHellman等机制

SSH基本原理  
![](http://cn.linux.vbird.org/linux_server/0310telnetssh_files/keypair-2.gif)

SSH联机过程  
![](http://cn.linux.vbird.org/linux_server/0310telnetssh_files/ssh-keypair2.gif)

经过上图中的第5步，服务器获取了客户端的公钥，客户端拿到了服务器的公钥，因此服务器和客户端的密钥系统（公钥+私钥）不同，因此被称为非对称密钥系统。建立了连接后，数据交互只用公私钥加解密即可：
- 服务器到客户端:服务器传送数据时,拿用户的公钥加密后送出。客户端接收后,用自己的私钥解密;
- 客户端到服务器:客户端传送数据时,拿服务器的公钥加密后送出。服务器接收后,用服务器的私钥解密。

## SSH服务

服务端提供的公钥与自己的私钥都放置于 /etc/ssh/ssh_host*，对于不同具体加密协议有不同的密钥对
```sh
root@LFG1000847446:~# ll /etc/ssh/ssh_host*
-rw------- 1 root root  672 Aug  5  2015 /etc/ssh/ssh_host_dsa_key
-rw-r--r-- 1 root root  601 Aug  5  2015 /etc/ssh/ssh_host_dsa_key.pub
-rw------- 1 root root  227 Aug  5  2015 /etc/ssh/ssh_host_ecdsa_key
-rw-r--r-- 1 root root  173 Aug  5  2015 /etc/ssh/ssh_host_ecdsa_key.pub
-rw------- 1 root root  399 Aug  5  2015 /etc/ssh/ssh_host_ed25519_key
-rw-r--r-- 1 root root   93 Aug  5  2015 /etc/ssh/ssh_host_ed25519_key.pub
-rw------- 1 root root 1675 Aug  5  2015 /etc/ssh/ssh_host_rsa_key
-rw-r--r-- 1 root root  393 Aug  5  2015 /etc/ssh/ssh_host_rsa_key.pub
```

启动服务和服务端口

```sh
root@LFG1000847446:~# /etc/init.d/sshd restart
root@LFG1000847446:~# netstat -tlnp | grep ssh
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      44781/sshd
tcp6       0      0 :::22                   :::*                    LISTEN      44781/sshd
```

> SSH服务不光包含ssh protocol的内容，还提供了安全的ssh ftp server服务

当服务器的ssh公钥发生变化，则会出现公钥不匹配的问题，需要手工删除.ssh/known_hosts的对应行（该例是第1行）

```sh
[root@www ~]# ssh root@localhost
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @ <==就告诉你可能有问题
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
a7:2e:58:51:9f:1b:02:64:56:ea:cb:9c:92:5e:79:f9.
Please contact your system administrator.
Add correct host key in /root/.ssh/known_hosts to get rid of this message.
Offending key in /root/.ssh/known_hosts:1 <==冒号后面接的数字就是有问题数据行号
RSA host key for localhost has changed and you have requested strict checking.
Host key verification failed.
```

## 免密码远程登录

既然ssh的数据交互是通过公私钥加解密，因此理论上只要获取对方的公钥后就可以进行数据交换，因此ssh支持免密码远程登录。

默认情况下，客户端的公私钥数据是在连接建立过程中（第4步）随机运算出来的，如果客户端先创建出固定的公私钥数据，然后公钥数据放置到服务器上，服务器就可以根据客户端的公钥数据认为客户端远程登录是合法的，也就不用提供密码。

```sh
xic@xic-desktop:~$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/xic/.ssh/id_rsa): 
Created directory '/home/xic/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/xic/.ssh/id_rsa.
Your public key has been saved in /home/xic/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:A9rA5VmAfWQIyhzhV6OikBD1iw6hR049fEdEL4inrE4 xic@xic-desktop
The key's randomart image is:
+---[RSA 2048]----+
|oo+..+=*B        |
|.= B.*oB..       |
|+ O @ B.o .      |
|o* = X o .       |
|+ + = . S        |
| + .     .       |
|  E              |
| o               |
|  .              |
+----[SHA256]-----+
```

生成的公私钥数据存放在`～/.ssh/`中，将`~/.ssh/id_rsa.pub`的内容存放到服务端的`~/.ssh/authorized_keys`中，执行`ssh-add`更新ssh agent中的key，即可免密码登录。

```sh
xic@xic-desktop:~$ ssh-add
Identity added: /home/xic/.ssh/id_rsa (/home/xic/.ssh/id_rsa)
```

# 应用

#### sftp

安全加密的FTP协议，相当于通过ssh协议登录，然后执行FTP操作：

```
cd/ls/mkdir/rmdir/pwd/chgrp/chown/rm/lcd/lls/lmkdir/put/get

```
#### scp

安全的网络复制工具，shell脚本中经常使用。如果第一次登录且不想交互式输入密码，需要使用另外一个工具：passssh

```sh
xic@xic-desktop:~$ sshpass -p "huawei123" scp -r -o StrictHostKeyChecking=no root@100.120.252.146:~/test.txt ~/
```

#### git

git支持的两种同步方式：http和ssh，ssh免密码参考isource的配置。


# 参考文档

- 鸟哥的linux私房菜-服务器：[第十一章、远程联机服务器SSH / XDMCP / VNC / RDP](http://cn.linux.vbird.org/linux_server/0310telnetssh_2.php)

