# 一、Linux 中安装docker

## 1.使用Docker仓库安装

### a.卸载旧版本的docker:

```
$ **sudo** **yum remove** docker \
         docker-client \
         docker-client-latest \
         docker-common \
         docker-latest \
         docker-latest-logrotate \
         docker-logrotate \
         docker-engine
```



### b.安装 Docker Engine-Community

#### **设置仓库**

安装所需的软件包。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。

```
$ **sudo** **yum install** -y yum-utils \
 device-mapper-persistent-data \
 lvm2
```



##### 阿里云

```
$ **sudo** yum-config-manager \
  --add-repo \
  http:**//**mirrors.aliyun.com**/**docker-ce**/**linux**/**centos**/**docker-ce.repo
```



##### 清华大学源

```
$ **sudo** yum-config-manager \
  --add-repo \
  https:**//**mirrors.tuna.tsinghua.edu.cn**/**docker-ce**/**linux**/**centos**/**docker-ce.repo
```



### c.安装 Docker Engine-Community

安装最新版本的 Docker Engine-Community 和 containerd，或者转到下一步安装特定版本：

```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

Docker 安装完默认未启动。并且已经创建好 docker 用户组，但该用户组下没有用户。

### d.通过将用户添加到docker用户组可以将sudo去掉

命令如下:

sudo groupadd docker #添加docker用户组

sudo gpasswd -a $USER docker #将登陆用户加入到docker用户组中

newgrp docker #更新用户组

### e.启动 Docker。

```
$ sudo systemctl start docker
```

### f.查看Docker映像

```
$ docker images
```

### g.通过运行 hello-world 映像来验证是否正确安装了 Docker Engine-Community 。

```
$ sudo docker run hello-world
```

## 2.卸载 docker

删除安装包：

```
yum remove docker-ce
```

删除镜像、容器、配置文件等内容：

```
rm -rf /var/lib/docker
```

# 二、Windows中安装Docker

## 1.安装准备

### a.安装Ubuntu 

在Microsoft Store中搜索Ubuntu 20.04 LTS，自动安装，但是我安装的不怎么管用，所以我自己在Hyper-v管理器中自己弄了个CentOS，暂时还能用，也不知道后面会不会出问题，先这样，后面再说，反正只是学习用。

### b.安装 WSL 并更新到 WSL 2

按快捷键win+x，以管理员身份打开PowerShell ，并运行：

```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

安装 WSL 2 之前，必须启用“虚拟机平台”可选功能。 以管理员身份打开 PowerShell 并运行：

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

重新启动计算机，以完成 WSL 安装并更新到 WSL 2。

---------------------------------------------------------------------------------------------------今天终于是装好了，能成功启动了，电脑重启了一万遍，烦死

重启计算机前，按F12进入BIOS，开启虚拟化，设置好后保存并退出



### c.启用“虚拟机平台”可选组件

以管理员身份打开 PowerShell 并运行：

```cmd
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

启用这些更改后，需要重新启动计算机。

### d.默认启用 WSL 2

在 PowerShell 中运行：

```cmd
wsl --set-default-version 2
```

### e.安装docker

打开刚刚安装的Ubuntu，安装依赖：

```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common
```

信任 Docker 的 GPG 公钥：

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

### f.设置docker镜像仓库:

```
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ],
  "insecure-registries": [],
  "debug": false,
  "experimental": false,
  "features": {
    "buildkit": true
  },
  "builder": {
    "gc": {
      "enabled": true,
      "defaultKeepStorage": "20GB"
    }
  }
}
```



### <img src="C:\Users\16143\AppData\Roaming\Typora\typora-user-images\image-20211013172658868.png" alt="image-20211013172658868" style="zoom:80%;" />

## 2.问题解决

### 1.'docker' could not be found in this WSL 2 distro. 

是因为权限不够，需要给docker添加当前用户或者使用更高权限的用户，权限设置后需要重启WSL：

管理员权限打开powershell ,然后执行下面命令

关闭服务

```
net stop LxssManager
```

重启服务

```
net start LxssManager	
```

反正就是瞎搞



### 2.关于linux 命令行报bash command not found的解决办法：

命令行输入命令执行后报“bash:....:command not found”这是由于系统PATH设置问题，PATH没有设置对，系统就无法找到精确命令了。 

1、在命令行中输入：export PATH=/usr/bin:/usr/sbin:/bin:/sbin 这样可以保证命令行命令暂时可以使用。命令执行完之后先不要关闭终端。 

2、在命令行中输入 vi /etc/profile 查看是否自己另外设置了PATH属性。 



# 三、参考链接：

Windows使用WSL2安装Dockerhttps://www.jianshu.com/p/c27255ede45f

在 wsl2 上安装 docker：https://www.cnblogs.com/jiangbo44/p/12637389.html

Winux之路-WSL 2的使用及填坑：https://zhuanlan.zhihu.com/p/224753478

docker 权限问题：thttps://blog.csdn.net/u011337602/article/details/104541261

Linux开启防火墙：https://blog.csdn.net/centose/article/details/96975849



socket=/usr/local/mysql/mysql.sock

