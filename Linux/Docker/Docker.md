# Docker

## 什么是Docker？

Docker是一个开源的引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署，包括VMs（虚拟机）、 bare metal、OpenStack 集群和其他的基础应用平台。

Docker通常用于如下场景：

* web应用的自动化打包和发布；
* 自动化测试和持续集成、发布；
* 在服务型环境中部署和调整数据库或其他的后台应用；
* 从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境。



## 安装Docker（CentOS）

官方自动安装脚本（阿里源）：

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

也可以使用国内 daocloud 一键安装命令：

```shell
curl -sSL https://get.daocloud.io/docker | sh
```

如果安装过旧版本，需要先删除相关依赖 ` sudo yum remove docker*`，下面提供使用 Docker 仓库进行安装方法。

首先安装相关依赖和工具：

```shell
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

然后设置稳定仓库

```shell
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

安装最新版本的 Docker Engine-Community 和 containerd，或者转到下一步安装特定版本：

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

Docker 安装完默认未启动。并且已经创建好 docker 用户组，但该用户组下没有用户。

**要安装特定版本的 Docker Engine-Community，请在存储库中列出可用版本，然后选择并安装：**

1、列出并排序您存储库中可用的版本。此示例按版本号（从高到低）对结果进行排序。

```txt
$ yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7           docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7           docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7          docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7          docker-ce-stable
```



2、通过其完整的软件包名称安装特定版本，该软件包名称是软件包名称（docker-ce）加上版本字符串（第二列），从第一个冒号（:）一直到第一个连字符，并用连字符（-）分隔。例如：docker-ce-18.09.1。

```shell
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

启动 Docker。

```shell
sudo systemctl start docker
```

通过运行 hello-world 映像来验证是否正确安装了 Docker Engine-Community 。

```shell
sudo docker run hello-world
```

* **卸载 docker**

删除安装包：

```shell
yum remove docker-ce
```

删除镜像、容器、配置文件等内容：

```shell
rm -rf /var/lib/docker
```



## Docker简单配置

### 普通用户使用docker

Docker安装好是默认root用户使用，普通用户没有权限，如果要在普通用户下使用Docker需要，使用root用户把普通用户加入docker组中。

```shell
# 添加执行 docker 命令的用户，这里 username 为用户名
useradd username

# 把 username 用户加入 docker 组
usermod -G docker username  

# 切换回普通用户
su - username

# 测试 如果不行请重启docker
docker ps -a
```



### 开机自动启动

docker默认是开机不会启动的，开机自启动的化需要执行如下命令

```shell
sudo chkconfig docker on 
```

禁止自启动需要把`on`替换成`off`。

### 安装可用镜像

* 搜索镜像

例如搜索centos镜像：

```shell
[wangs7@localhost ~]$ docker search centos
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                            The official build of CentOS.                   6836      [OK]       
ansible/centos7-ansible           Ansible on Centos7                              135                  [OK]
consol/centos-xfce-vnc            Centos container with "headless" VNC session…   132                  [OK]
jdeathe/centos-ssh                OpenSSH / Supervisor / EPEL/IUS/SCL Repos - …   121                  [OK]
centos/systemd                    systemd enabled base container.                 105                  [OK]
centos/mysql-57-centos7           MySQL 5.7 SQL database server                   91                   
imagine10255/centos6-lnmp-php56   centos6-lnmp-php56                              58                   [OK]

```

* 安装镜像

下载镜像的命令非常简单，使用docker pull命令即可。在docker的镜像索引网站上面，镜像都是按照 用户名/ 镜像名的方式来存储的。有一组比较特殊的镜像，比如ubuntu这类基础镜像，经过官方的验证，值得信任，可以直接用 镜像名来检索到。例如上面的centos。我以前安装过，如果第一次安装提示可能不一样。                

```shell
[wangs7@localhost ~]$ docker pull centos
Using default tag: latest
latest: Pulling from library/centos
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Image is up to date for centos:latest
docker.io/library/centos:latest
```



## Docker常用命令

### pull

从镜像仓库中拉取或者更新指定镜像，在未声明镜像标签时，默认标签为latest。

```js
Usage: docker pull [OPTIONS] NAME[:TAG|@DIGEST] 
Options: 
    -a 拉取某个镜像的所有版本
    --disable-content-trust 跳过校验，默认开启
```

### run

创建并启动一个容器

```js
Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
Options:
  -d, --detach 后台运行容器，并输出容器ID
  -e, --env list 设置环境变量，该变量可以在容器内使用
  -h, --hostname string 指定容器的hostname
  -i, --interactive 以交互模式运行容器，通常与-t同时使用
  -l, --label list 给容器添加标签
  --name string 设置容器名称，否则会自动命名
  --network string 将容器加入指定网络
  -p, --publish list 设置容器映射端口
  -P,--publish-all 将容器设置的所有exposed端口进行随机映射
  --restart string 容器重启策略，默认为不重启
    on-failure[:max-retries]：在容器非正常退出时重启，可以设置重启次数。
    unless-stopped：总是重启，除非使用stop停止容器
    always：总是重启
  --rm 容器退出时则自动删除容器
  -t, --tty 分配一个伪终端
  -u, --user string 运行用户或者UID
  -v, --volume list 数据挂载
  -w, --workdir string 容器的工作目录
  --privileged 给容器特权
```























