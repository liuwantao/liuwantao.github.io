---
layout: post
title: "ubuntu服务器管理"
date: 2016-03-22 23:14:54
categories: linux
---

## ubuntu 服务器管理

安装 Ubuntu 以后，需要对系统的安全性进行加固，并在日常使用中时刻关注系统安全。本篇内容包括：日志查看监控、加强SSH安全、使用ufw防火墙等。

查看用户活动
<hr>
     
  w  当前登录用户的详细状态
  ac  用户总的连接时间。从/var/log/wtmp文件获取信息
  last  所有用户登陆记录。从/var/log/wtmp文件获取信息
  lastb  所有失败的、错误的登录
  
日志查看与监控

<hr>

系统日志  
   
  /var/log/auth.log
  /var/log/kern.log
  /var/log/daemon.log
  /var/log/debug
  /var/log/dmesg  # Kernel Ring Buffer
  /var/log/messages
  /var/log/syslog

  logger ...
  logrotate 
  /etc/syslog.conf
  /etc/logrotate.conf
  /etc/cron.daily/logrotate
  /etc/cron.daily/sysklogd

应用日志  

  /var/log/apache2/access.log
  /var/log/apache2/error.log
  /var/log/rkhunter.log

  ~/.bash_history  #bash命令历史
  ~/.mysql_history  #mysql命令历史

日志查看  

  cat 和 grep
  tail 和 head
  less 和 more

  dmesg | less
  dmesg | grep pnp | less
  grep sshd /var/log/auth.log | less

  history  # shell命令历史
  faillog  # /var/log/faillog
  lastlog  # /var/log/lastlog
  who      # /var/log/wtmp

日志监控  

  tail -f example.log  #跟踪文件，Ctrl+C退出

## 分析系统日志, 可提取并邮件发送重要信息

  sudo aptitude install logwatch
  ls -l /usr/share/logwatch/scripts/services
  sudo cp /usr/share/logwatch/default.conf/logwatch.conf /etc/logwatch/conf/
  sudo vi /etc/logwatch/conf/logwatch.conf
  sudo logwatch | less

屏蔽IP防密码破解  

  sudo apt-get install fail2ban
  sudo vi /etc/fail2bin/jail.conf
  sudo /etc/init.d/fail2ban restart 

### 创建管理帐号，禁用root

创建管理帐号

  useradd -m -s /bin/bash abc
  gpasswd -a abc sudo  #加入sudo组
  passwd abc
  su - abc

创建管理组  

  sudo groupadd -r admin
  sudo usermod -aG admin YOUR_ADMIN_USERNAME

限制su使用  

  sudo dpkg-statoverride --update --add root admin 4750 /bin/su
  sudo dpkg-statoverride --list

禁用root  

  sudo passwd -l root

加强 SSH 安全
在服务端生成密钥   

  su - abc
  # 建立公钥与私钥
  ssh-keygen -t rsa
  # 密钥文件，可直接回车使用默认的文件名
  Enter file in which to save the key (/home/abc/.ssh/id_rsa): 
  # 输入passphrase，仅供密钥登录使用，加强安全，可以为空
  Enter passphrase (empty for no passphrase):
  Enter same passphrase again:
  # 改名为sshd支持的默认公钥文件名
  mv ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  # 设置授权公钥文件权限，只可以被自己访问
  chmod 600 ~/.ssh/authorized_keys
  # 下载私钥（可远程scp获取）删除备份
  rm –rf ~/.ssh/id_rsa  

或在客户端生成密钥  

  ssh-keygen -t rsa -C "youremail@example.com"
  scp ~/.ssh/id_rsa.pub user@server:/tmp/user.pub 或
  cat ~/.ssh/id_rsa.pub
  # 服务端登录SSH所用帐户
  su - abc
  mkdir .ssh
  chmod 700 .ssh
  vi .ssh/authorized_keys
  # 粘贴公钥到此文件，并保存退出，或
  cat /tmp/user.pub >> ~/.ssh/authorized_keys
  chmod 600 .ssh/authorized_keys
  # 保护好私钥(~/.ssh/id_rsa)

修改SSH配置文件  

  sudo vi /etc/ssh/sshd_config
  Port 1234  # 更改端口 1024-65535
  PermitRootLogin no  # 禁用root登录
  PasswordAuthentication no  # 禁用密码登录（小心）
  :wq 保存退出，重启 sshd:
  sudo service ssh restart

测试SSH登录  

  ssh -p 1234 abc@ip  # or
  ssh abc@ip -i id_rsa -p 1234

>提示：PasswordAuthentication no 配置，一定要在生成key并测试成功之后再使用，否则会导致无法登录。
>使用 ufw 防火墙
>UFW(Uncomplicated Firewall，简单防火墙)是用于配置 iptable 的一个简单接口。iptable 控制 Netfilter，Netfilter 内置于内核之中。  
  
端口信息查看  

  less /etc/services  #服务与端口列表
  sudo netstat -anpt | grep LISTEN  #查看在用端口
  安装配置 ufw
  sudo apt-get install ufw
  sudo vi /etc/default/ufw
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ls -l /etc/ufw/applications.d/
  sudo ufw app list  #可用服务
  sudo ufw show  #可用命令

网络连接管理  

  sudo ufw [delete] allow 1234/tcp
  sudo ufw [delete] allow 80/tcp
  sudo ufw [delete] deny ...
  sudo ufw show added
  sudo ufw enable
  sudo ufw status [verbose]
  sudo ufw status numbered
  sudo ufw delete [number]
  ufw 控制
  sudo ufw reset
  sudo ufw reload
  sudo service ufw restart
  sudo ufw disable && sudo ufw enable

保护共享内存
攻击一个运行中的服务(如httpd)经常要使用 /dev/shm，修改 /etc/fstab 使其更安全（需要reboot）。  

  sudo vi /etc/fstab
  tmpfs /dev/shm tmpfs defaults,noexec,nosuid 0 0
