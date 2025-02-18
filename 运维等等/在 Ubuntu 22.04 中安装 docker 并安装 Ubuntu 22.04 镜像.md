# 在 Ubuntu 22.04 中安装 docker 并安装 Ubuntu 22.04 镜像

> 先看文档，再问 AI

### 0. 为什么要套娃？

是因为要把现有服务器分享给公司其他部门同事使用，为了防止环境混乱，所以需要创建一个容器

### 1.设置代理（可选）

由于**众所周知**的原因，强烈建议在使用 docker 之前配置代理，这样拉取镜像时会减少不必要的等待

```bash
# 1. 创建 dockerd 配置文件
sudo mkdir /etc/systemd/system/docker.service.d
# 2.创建 http-proxy.conf 文件
touch http-proxy.conf
# 3.配置代理
vim http-proxy.conf

[Service]
Environment="HTTP_PROXY=http://proxy.example.com:8080/"
Environment="HTTPS_PROXY=http://proxy.example.com:8080/"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
```

### 2.[安装 docker](https://yeasy.gitbook.io/docker_practice/install/ubuntu)

```bash
# 1.更新 apt 源
sudo apt update
# 2.安装前置依赖
sudo apt install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
# 3.为了确认所下载软件包的合法性，需要添加软件源的 GPG 密钥
  # 淘宝源
  curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

  # 官方源
  # $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o  /usr/share/keyrings/docker-archive-keyring.gpg
  
# 4.然后，我们需要向 sources.list 中添加 Docker 软件源
  # 淘宝源
  $ echo \
    "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


  # 官方源
  # $ echo \
  #   "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  #   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5.更新 apt 源并安装 docker-ce
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# 6.将 docker 添加到服务并开启开机启动
sudo systemctl enable docker
sudo systemctl start docker

# 7.创建用户组&将当前用户添加到用户组中
sudo groupadd docker
sudo usermod -aG docker $USER

# 8.关闭终端，然后打开新的窗口输入以下命令查看 docker 安装情况
docker info

# 如果设置了代理，可以在打印出来的内容中查看是否有 HTTP Proxy 和 HTTPS Proxy
```

### 3.使用 docker 安装 ubuntu 22.04

```bash
# 1.配置 dockerd 代理 https://docs.docker.com/reference/cli/dockerd/#on-linux
mkdir /etc/docker 
cd /etc/docker
vim daemon.json
# 2.将以下配置添加到 daemon.json 中
{
	"proxies": {
    "http-proxy": "http://proxy.example.com:80",
    "https-proxy": "https://proxy.example.com:443",
    "no-proxy": "*.test.example.com,.example.org"
  }
}
# 3.拉取镜像
docker pull ubuntu:22.04
# 4.运行容器
docker run -dit --name ubuntu_2204 -p 2222:22 --restart always ubuntu:22.04
# 5.进入容器
docker exec -it ubuntu_2204 bash
# 6.安装 curl 并测试代理工作状态
apt update 
apt install curl vim
curl -i google.com 
```

## 踩坑

网上很多资料都是过时的，docker在 17.x 版本后配置文件有比较大的更改，很多大模型的训练语料中包含了过时的资料，导致输出的答案有很多错误

所以还是以最新的官方文档为准，基本的手艺活不能丢～

Happy coding~





