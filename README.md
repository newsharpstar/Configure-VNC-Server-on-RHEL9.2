# Configure VNC Server on RHEL9.2

本文如无特殊说明，所有用户登录全部为普通用户，不使用root用户。
本文不讨论如何关闭启用RHEL防火墙。
本文不讨论如何配置dnf安装软件基础环境。
## 1. 安装vncserver

> [test@localhost ~]$ sudo dnf install tigervnc-server -y

```bash
sudo dnf install tigervnc-server -y
```

## 2. 复制配置文件

> [test@localhost ~]$ sudo cp /usr/lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service

```bash
sudo cp /usr/lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
```
vncserver@:***ID***.service，这个ID与下一步的修改用户配置文件前面的ID必须相同，否者最后启动服务报错。使用journalctl -xeu vncserver@:1.service命令可以查看服务启动失败的log有一行提示：


> May 17 20:12:52 localhost.localdomain vncsession-restore[90750]: No user configured for display :1


## 3. 修改vnc登录用户配置文件

 

> [test@localhost ~]$ cat /etc/tigervnc/vncserver.users
:1=test

:***ID***=test，必须与上一步的配置文件ID.service相同。在原文件里面修改可以，或者自己单独添加也行。
```bash
sudo vim /etc/tigervnc/vncserver.users
```

## 4. 修改vnc账户登录密码

> [test@localhost ~]$ vncpasswd
Password:
Verify:
Would you like to enter a view-only password (y/n)? n
A view-only password is not used

vnc登录密码与test登录RHEL的系统密码相互独立。
```bash
vncpasswd
```

## 5. 修改vnc配置文件
```bash
 [test@localhost ~]$ pwd
 /home/test
[test@localhost ~]$ l.
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .cache  .config  .lesshst  .local  .mozilla  .session  .viminfo  ***.vnc***  .Xauthority
```
> [test@localhost ~]$ echo gnome-session > ~/.session

没有任何的输出内容

```bash
echo gnome-session > ~/.session
```

> [test@localhost ~]$ vim ~/.vnc/config

这一步一定要注意执行的目录。普通用户是在/home/用户。如果使用root登录，那么需要进入到/root目录，是l.命令查看是否有.vnc这个目录
```bash
vim ~/.vnc/config
```

> [test@localhost ~]$ cat ~/.vnc/config
session=gnome
securitytypes=vncauth,tlsvnc
geometry=1280x720

 
## 6. 启动vnc服务

> [test@localhost ~]$ sudo systemctl start vncserver@:1.service

```bash
sudo systemctl start vncserver@:1.service
```

## 7. 启用vnc服务

> [test@localhost ~]$ sudo systemctl enable vncserver@:1.service

```bash
sudo systemctl enable vncserver@:1.service
```

## 8. 查看vnc服务状态
```bash
 [test@localhost ~]$ sudo systemctl status vncserver@:1.service
● vncserver@:1.service - Remote desktop service (VNC)
     Loaded: loaded (/etc/systemd/system/vncserver@:1.service; enabled; preset: disabled)
     Active: active (running) since Fri 2024-05-17 20:15:32 CST; 7s ago
    Process: 91720 ExecStartPre=/usr/libexec/vncsession-restore :1 (code=exited, status=0/SUCCESS)
    Process: 91731 ExecStart=/usr/libexec/vncsession-start :1 (code=exited, status=0/SUCCESS)
   Main PID: 91738 (vncsession)
      Tasks: 0 (limit: 48625)
     Memory: 1012.0K
        CPU: 98ms
     CGroup: /system.slice/system-vncserver.slice/vncserver@:1.service
             ‣ 91738 /usr/sbin/vncsession test :1
May 17 20:15:32 localhost.localdomain systemd[1]: Starting Remote desktop service (VNC)...
May 17 20:15:32 localhost.localdomain systemd[1]: Started Remote desktop service (VNC).
```


## 9. 查看vnc启用的端口  
```bash
[test@localhost ~]$ netstat -lnp | grep vnc
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:5901            0.0.0.0:*               LISTEN      91748/Xvnc
tcp6       0      0 :::5901                 :::                    LISTEN      91748/Xvnc
unix  2      [ ACC ]     STREAM     LISTENING     144858   91748/Xvnc           @/tmp/.X11-unix/X1
unix  2      [ ACC ]     STREAM     LISTENING     144859   91748/Xvnc           /tmp/.X11-unix/X1
```

tcp右侧显示了当前vnc服务启用的端口。如果有多个用户，每个***ID***.service都是一个独立的服务，每个***ID***.service都是独立的服务端口号。
## 10. 连接vnc服务端
使用MobaXterm软件连接即可。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff1435d7e946dedce04cfc050f78bcbd.png)
一定要修改Port号，根据上面的查看服务端口号填写。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4708cd674d419a46844e2a3782380cd2.png)
