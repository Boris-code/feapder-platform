# 爬虫管理系统

> 生而为虫，不止于虫

## 特性

1. **爬虫管理系统不仅支持 `feapder`、`scrapy`，且支持执行任何脚本，可以把该系统理解成脚本托管的平台** 。因为爬虫往往需要其他脚本辅助，如生产cookie脚本、搭建nodejs服务破解js，甚至是其他语言的脚本，本管理系统在设计之初就考虑到了这一点，因此可完美支持。

2. **支持集群**，工作节点根据配置定时启动，**执行完释放，不常驻**，节省服务器资源。一个爬虫实例一个节点，**彼此之间隔离**，互不影响。

## 功能概览

### 1. 项目管理

项目列表
![-w1786](http://markdown-media.oss-cn-beijing.aliyuncs.com/2021/07/06/16254967791920.jpg)

添加/编辑项目
![-w1785](http://markdown-media.oss-cn-beijing.aliyuncs.com/2021/07/06/16254968151490.jpg)

### 2. 任务管理

任务列表
![-w1791](http://markdown-media.oss-cn-beijing.aliyuncs.com/2021/07/06/16254968630425.jpg)

定时支持 crontab、时间间隔、指定日期、只运行一次 四种方式。只运行一次的定时方式会在创建任务后立即运行
![-w1731](http://markdown-media.oss-cn-beijing.aliyuncs.com/2021/07/06/16254968513292.jpg)

### 3. 任务实例

列表
![-w1785](http://markdown-media.oss-cn-beijing.aliyuncs.com/2021/07/06/16254981090479.jpg)

日志
![-w1742](http://markdown-media.oss-cn-beijing.aliyuncs.com/2021/07/06/16254983085371.jpg)

## 部署

> 下面部署以centos为例， 其他平台部署可参考docker官方文档：https://docs.docker.com/compose/install/

### 1. 安装docker

删除旧版本（需要重装升级时执行）

```shell
yum remove docker  docker-common docker-selinux docker-engine
```

安装：
```shell
yum install -y yum-utils device-mapper-persistent-data lvm2 && python2 /usr/bin/yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo && yum install docker-ce -y
```

启动
```shell
systemctl enable docker
systemctl start docker
```

### 2. 安装 docker swarm

初始化
    
    docker swarm init
    
    # 如果你的 Docker 主机有多个网卡，拥有多个 IP，必须使用 --advertise-addr 指定 IP
    docker swarm init --advertise-addr 192.168.99.100

初始化后会提示如下：

```bash
> docker swarm init --advertise-addr 192.168.99.100
Swarm initialized: current node (za53ikuwzpr11ojuj4fgx8ys0) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1ujljqjf3mli9r940vcdjd7clyrdfjkqyf8g4g6kapfvkjkj9e-41byjvvodfpk7nz4smfdq44w0 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

添加其他服务器为节点时使用上面提示的 `docker swarm join --token [token] [ip]`命令 

### 3. 安装docker-compose

```python
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 4. 部署管理系统

#### 1. 下载项目

```shell
git clone https://github.com/Boris-code/feapder-platform.git
```

#### 2. 运行 

首次运行需拉取镜像，时间比较久，且运行可能会报错，再次运行下就好了

```shell
cd feapder-platform
docker-compose up
```

*运行起来会提示购买授权码，购买后继续*

#### 3. 修改配置

```shell
cd feapder-platform
vim docker-compose.yaml
```

配置里有注释，注意必须修改下面两项

```shell
- FEAPDER_BACKEND_URL=http://ip:8000 # **必填 服务端内网地址
- AUTHORIZATION_CODE= # **必填  授权码
```

查看内网地址：

```shell
ifconfig
```
![](http://markdown-media.oss-cn-beijing.aliyuncs.com/2021/07/06/16255025919847.jpg)


#### 4. 后台运行
```shell 
docker-compose up -d
```

#### 5. 访问爬虫管理系统

默认地址：`http://localhost`

端口修改在`docker-compose.yaml`

#### 5. 停止

```shell
docker-compose stop
```

## 拉取私有项目

拉取私有项目需在git仓库里添加如下公钥

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCd/k/tjbcMislEunjtYQNXxz5tgEDc/fSvuLHBNUX4PtfmMQ07TuUX2XJIIzLRPaqv3nsMn3+QZrV0xQd545FG1Cq83JJB98ATTW7k5Q0eaWXkvThdFeG5+n85KeVV2W4BpdHHNZ5h9RxBUmVZPpAZacdC6OUSBYTyCblPfX9DvjOk+KfwAZVwpJSkv4YduwoR3DNfXrmK5P+wrYW9z/VHUf0hcfWEnsrrHktCKgohZn9Fe8uS3B5wTNd9GgVrLGRk85ag+CChoqg80DjgFt/IhzMCArqwLyMn7rGG4Iu2Ie0TcdMc0TlRxoBhqrfKkN83cfQ3gDf41tZwp67uM9ZN feapder@qq.com
```

## 自定义爬虫节点

默认的爬虫节点只打包了`feapder`、`scrapy`框架，若需要其它环境，可基于`boris0621/feapder_front:1.0`镜像自行构建

如替换git仓库的公钥私钥
```
FROM boris0621/feapder_front:1.0

COPY .ssh /root/.ssh

```

自己随便搞事情，搞完修改下 docker-compose.yaml 里 SPIDER_IMAGE 的值即可

欢迎提PR，大家一起构建一个🐂的镜像

