---
title: SSHClient
date: 2017-04-24 18:56:22
tags:
categories: python
---
# SSHClient
用于连接远程服务器并执行基本命令
## 基于用户名密码连接：
``` python
import paramiko

# 创建SSH对象
ssh = paramiko.SSHClient()
# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 连接服务器
ssh.connect(hostname='10.10.39.213', port=22, username='username', password='password')

# 执行命令
stdin, stdout, stderr = ssh.exec_command('ls ./aa')
# 获取命令结果
result = stdout.read()
print(result)
# 关闭连接
ssh.close()
```
输出结果：
b'redis-3.2.8\nredis-3.2.8.tar.gz\n'
我用的是我们实验室里面的一台主机，\n是换行。
## Linux中ssh 第一次登陆某台主机的时候会出现：
![Alt text](/images/python/ssh6.png)
如果输入yes就会在,该用户的根目录下.ssh/known_hosts，这个文件中多了一条
![Alt text](/images/python/ssh7.png)
所以
``` python  
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
 ```
 这句话就是相当于把那个密钥加到.ssh/known_hosts 这里面
顺便Linux之间可以用scp这个命令来上传下载文件。
``` Linux
scp -rp ssh.py ssh_oj@10.10.39.213:/tmp/
```
上面是带两个参数，第一个参数r，是指目录，就是它是个目录也可以copy。
第二个参数是p，p是权限。就是说copy到另外一台主机之后，这个文件也会拥有在本机相同的权限。
ssh_oj指的是用户名，10.10.39.213是指主机的ip地址
/tmp/ 是将文件copy到那个目录。
结果:
![Alt text](/images/python/ssh1.png)
在登上该主机：
![Alt text](/images/python/ssh2.png)
有了这个文件
如果ssh的默认端口号不是22，可以改端口号通过-P
![Alt text](/images/python/ssh3.png)
上面的端口是52113
## 在SSHClient实现文件上长传下载
``` python
import paramiko

transport = paramiko.Transport(('hostname',22))
transport.connect(username='username',password='password')

sftp = paramiko.SFTPClient.from_transport(transport)
# 将location.py 上传至服务器 /tmp/test.py
sftp.put('/Volumes/ziyan/pythonworkspace/ssh.py', '/tmp/ssh.py')
# 将remove_path 下载到本地 local_path
sftp.get('remove_path', 'local_path')

transport.close()
```
结果：
![Alt text](/images/python/ssh4.png)
## 使用ssh公钥密钥自动登陆linux服务器
```linux
[root@server ~]# ssh-keygen -b 1024 -t rsa
Generating public/private rsa key pair.     #提示正在生成rsa密钥对
Enter file in which to save the key (/home/usrname/.ssh/id_dsa):     #询问公钥和私钥存放的位置，回车用默认位置即可
Enter passphrase (empty for no passphrase):     #询问输入私钥密语，输入密语
Enter same passphrase again:     #再次提示输入密语确认
Your identification has been saved in /home/usrname/.ssh/id_dsa.     #提示公钥和私钥已经存放在/root/.ssh/目录下
Your public key has been saved in /home/usrname/.ssh/id_dsa.pub.
The key fingerprint is:
x6:68:xx:93:98:8x:87:95:7x:2x:4x:x9:81:xx:56:94 root@server     #提示key的指纹
```
简单说明一下：
-b 1024　采用长度为1024字节的公钥/私钥对，最长4096字节，一般1024或2048就足够满足安全需要了，太长的话加密解密需要的时间也增长。
-t rsa　 采用rsa加密方式的公钥/私钥对，除了rsa还有dsa方式，rsa方式最短不能小于768字节长度。
如果还需要使用更多其他参数请参考man ssh-keygen。
         在生成密钥对的过程中你被询问：输入密码短句 Enter passphrase (empty for no passphrase) ，密码短句（passphrase）是你使用一个短语或者一句话作为密码输入，再由系统内部的加密或是散列算法生成虚拟密码后，进行下一步的认证。好处是增强了安全性不易被破解。看过很多文章，里面都把这个短句输入为空，也就是代表不使用密码短句。在这里我强烈要求你输入密码短句。有人会说使用密码短句后，登陆还要输入密码短句这样使用没有比使用用户名和密码登陆方便多少，我说请你不要急，接着看我的文章。
### 注意：
如果你生成密钥对而不设置密码短语，那么如果你的私钥丢失了，那么就你的麻烦可能会比丢失用户名密码还严重。
第二步：拷贝你的公钥到被管理的服务器上
在你的管理服务器上把你的公钥拷贝到被管理服务器上要进行自动登陆的用户目录下。
```Linux
[root@server ~]# scp .ssh/id_dsa.pub remote_usrname@192.168.0.2:      #比如你想使用用户peter登陆，则remote_usrname请以peter代替
```
改名和进行权限设置
    登陆被管理的服务器，进入需要远程登陆的用户目录，把公钥放到用户目录的 .ssh 这个目录下（如果目录不存在，需要创建~/.ssh目录，并把目录权限设置为700），把公钥改名为authorized_keys2，并且把它的用户权限设成600。
```Linux
[peter@client ~]$ ls
id_rsa.pub
[peter@client ~]$ mkdir ~/.ssh     #如果当前用户目录下没有 .ssh 目录，请先创建目录
[peter@client ~]$ chmod 700 ~/.ssh
[peter@client ~]$ mv id_rsa.pub ~/.ssh
[peter@client ~]$ cd ~/.ssh
[peter@client ~]$ cat id_rsa.pub >> authorized_keys2
[peter@client ~]$ rm -f id_rsa.pub
[peter@client ~]$ chmod 600 authorized_keys2
[peter@client ~]$ ls -l
total 4
-rw-------  1 peter peter 225 Oct 10 11:28 authorized_keys2
```    
测试使用密钥对进行远程登陆
```Linux
[root@server ~]# ssh peter@192.168.0.2
Enter passphrase for key '/root/.ssh/id_rsa':      #提示输入密码短语，请输入刚才设置的密码短语
Last login: Sun Oct 10 11:32:14 2010 from 192.168.0.1
[peter@client ~]$
```
如果你不能用正确的登录，应该重新检查一下你的authorized_keys2的权限。也可能要检查.ssh目录的权限。
使用 ssh-agent（ssh代理）自动输入密码短语
牢记你的“密码短句”，现在你可以用你的密钥而不是密码来登录你的服务器了，但是这样仍然没有省什么事，你还是要输入密钥的“密码短语”。有更简便的方法吗？答案就是采用SSH代理（ssh-agent），一个用来帮你记住“密码短语”的程序。 ssh-agent是OpenSSH中默认包括的ssh代理程序。
登陆管理服务器
```Linux
[root@server ~]# ssh-agent
SSH_AUTH_SOCK=/tmp/ssh-vEGjCM2147/agent.2147; export SSH_AUTH_SOCK;
SSH_AGENT_PID=2148; export SSH_AGENT_PID;
echo Agent pid 2148;
```
当你运行ssh-agent，它会打印出来它使用的 ssh 的环境和变量。要使用这些变量，有两种方法，一种是手动进行声明环境变量，另一种是运行eval命令自动声明环境变量。
方法一：手动声明环境变量
``` Linux
[root@server ~]# SSH_AUTH_SOCK=/tmp/ssh-vEGjCM2147/agent.2147; export SSH_AUTH_SOCK;
[root@server ~]# SSH_AGENT_PID=2148; export SSH_AGENT_PID;
[root@server ~]# printenv | grep SSH     #检查 ssh 环境变量是否已经加入当前会话的环境变量
SSH_AGENT_PID=2148
SSH_AUTH_SOCK=/tmp/ssh-vEGjCM2147/agent.2147
```
方法二：运行eval命令自动声明环境变量
``` Linux
[root@server ~]# eval `ssh-agent`
Agent pid 2157
[root@server ~]# printenv | grep SSH     #检查 ssh 环境变量是否已经加入当前会话的环境变量
SSH_AGENT_PID=2148
SSH_AUTH_SOCK=/tmp/ssh-vEGjCM2147/agent.2147
```
现在 ssh-agent 已经在运行了，但是 ssh-agent 里面是空白的不会有解密的专用密钥。我们要告诉它我们有私钥和这个私钥在哪儿。这就需要使用 ssh-add 命令把我们的专用密钥添加到 ssh-agent 的高速缓存中。
``` Linux
[root@server ~]# ssh-add ~/.ssh/id_dsa
Enter passphrase for /home/user/.ssh/id_dsa:     #输入你的密码短语
Identity added: /home/user/.ssh/id_dsa (/home/user/.ssh/id_dsa)
[root@server ~]# ssh-add -l     #查看 ssh代理的缓存内容
1024 72:78:5e:6b:16:fd:f2:8c:81:b1:18:e6:9f:77:6e:be /root/.ssh/id_rsa (RSA)
```
输入了密码短句，现在好了，你可以登录你的远程服务器而不用输入你的密码短语了，而且你的私钥是密码保护的。试试看是不是很爽！
``` Linux

[root@server ~]# ssh peter@192.168.0.2
Last login: Sun Oct 10 11:32:45 2010 from 192.168.0.1
[peter@client ~]$
```
登陆服务器进行操作结束后，记得还要把 ssh-agent 关掉，不然其他人登陆后也可以远程了。
``` Linux
[root@server ~]# ssh-agent -k
unset SSH_AUTH_SOCK;
unset SSH_AGENT_PID;
echo Agent pid 2148 killed;
[root@server ~]# ssh-add -l     #查看一下，缓存里已经没有了密钥了
The agent has no identities.
```
## 基于公钥密钥连接：
``` python
import paramiko

private_key = paramiko.RSAKey.from_private_key_file('/Users/liyongjun/.ssh/id_rsa')

# 创建SSH对象
ssh = paramiko.SSHClient()
# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 连接服务器
ssh.connect(hostname='hostname', port=22, username='username',pkey=private_key)
# 执行命令
stdin, stdout, stderr = ssh.exec_command('df')
# 获取命令结果
result = stdout.read()
print(result.decode())
# 关闭连接
ssh.close()
```
结果：
文件系统           1K-块    已用      可用 已用% 挂载点
udev             1006620       0   1006620    0% /dev
tmpfs             204796    5264    199532    3% /run
/dev/sda1      136750208 5922700 123857960    5% /
tmpfs            1023964     148   1023816    1% /dev/shm
tmpfs               5120       0      5120    0% /run/lock
tmpfs            1023964       0   1023964    0% /sys/fs/cgroup
cgmfs                100       0       100    0% /run/cgmanager/fs
tmpfs             204796      28    204768    1% /run/user/119
tmpfs             204796       0    204796    0% /run/user/1001
## 基于公钥密钥上传下载
``` python
import paramiko

private_key = paramiko.RSAKey.from_private_key_file('/Users/liyongjun/.ssh/id_rsa')

transport = paramiko.Transport(('hostname', 22))
transport.connect(username='username', pkey=private_key )

sftp = paramiko.SFTPClient.from_transport(transport)
# 将location.py 上传至服务器 /tmp/test.py
sftp.put('/Volumes/ziyan/pythonworkspace/ssh.py', '/tmp/test.py')
# 将remove_path 下载到本地 local_path
# sftp.get('remove_path', 'local_path')

transport.close()
```
结果：
![Alt text](/images/python/ssh8.png)
