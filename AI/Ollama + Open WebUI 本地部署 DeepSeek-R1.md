# Ollama + Open WebUI 本地部署 DeepSeek-R1

> 春节假期 DeepSeek-R1 全网刷屏，苦于回老家没有设备直到节后才有机会在本地部署 32b 和 70b ，全网攻略一大堆，基础部分一带而过，主要记录一些踩坑经历



|      | 32b q_4     | 70b q_4      |
| :--: | ----------- | ------------ |
| CPU  | i7-11700k   | I9-14900k    |
| 内存 | 32g         | 128g         |
| 显卡 | 2080Ti  22g | 4090 24g * 2 |

系统版本均为 Ubuntu 22.04 实测以上配置均可流畅运行

## 安装 Ollama

#### 1. 下载

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

#### 2.设置服务开机运行

```bash
sudo systemctl enable ollama
```

#### 3. 切换默认的模型储存位置（可选）

```bash
# 我的机器是500G的系统盘+1T数据盘 需要把模型下载到数据盘中
# 1. Ubuntu 挂载第二块硬盘（如果你还没有挂载的话）
sudo fdisk -l # 查看硬盘列表 如果是nvme硬盘，名字一般是 /dev/nvme1n1 这种的，然后通过名称和容量分辨
sudo mkdir /mnt/data # 在 /mnt 目录下创建文件夹作为新硬盘的挂载点
sudo mount /your/disk /mnt/data # 将新硬盘挂载到上面创建的文件夹中
sudo vim /etc/fstab # 编辑挂载文件
/your/disk   /mnt/data   ext4   defaults   0   0 # 在文件末尾添加这一行，实现开机自动挂载
# 2. 设置挂载点权限
sudo mkdir /mnt/data/ollama_models
sudo chown -R $USER:$USER /mnt/data/ollama_models
sudo chmod -R 755 /mnt/data/ollama_models
# 3. 修改 ollama 配置文件
# 2025-02-06 使用官方安装命令应该是没有 ~/.ollama/config 和 /etc/ollama/config 文件的，如果有网上一大把教如何修改环境变量的，因为上一部已经将 ollama 设置成系统服务，这时候我们需要修改服务配置文件
sudo vim /etc/system/systemd/ollama.service
# 在 [Service] 下增加一行然后保存更改
Environment="OLLAMA_MODELS=/mnt/data/ollama_models"
# 4. 重启服务
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

#### 4.拉取 DeepSeek-R1:32b

```bash
ollama pull deepseek-r1:32b
```

## 安装 Open WebUI

> Open WebUI 官方推荐使用 docker 安装

#### 1. Ubuntu 安装 docker

[Docker 中文教程](https://yeasy.gitbook.io/docker_practice/install/ubuntu)

#### 2.安装 Open WebUI

```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

### 打开 `http://your.ubuntu.server:8080/` 这时候应该就可以正常使用了

## 下面是不能正常使用的表现和解决方案

#### 1.Open WebUI 模型列表为空

```bash
# a.查看 ollama 模型列表，是否之前拉取模型失败
	ollama list # 如果为空，则重新拉取模型
# b.查看 Open WebUI 启动日志
	# 观察每条日志头的标识，查看 ERROR 标识日志，极大概率是 ollama 没有放开外部访问
	sudo vim /etc/systemd/system/ollama.service
	# 在 [Service] 下增加一行环境变量,允许外部访问
	Environment="OLLAMA_HOST=0.0.0.0:11434"
	# 保存并重启 ollama
	sudo systemctl daemon-reload
	sudo systemtctl restart ollama
```

#### 2. Open WebUI 页面加载很慢

> 第一次打开页面需要进行管理员账号注册，注册结束后大概率会发现页面一片空白，然后等待大约3-5分钟才会显示，通过观察发现 Open WebUI 默认会连接 OpenAI API ，但由于众所周知的原因大概率会失败，所以会耗费大量时间

###### 	a.点击右上角头像，打开管理员面板

###### 	![管理员面板](https://static.yuehaowei.fun/static/blog-images/ai/1.png)

###### 	b.关闭 OpenAI API 

![关闭 OpenAI API ](https://static.yuehaowei.fun/static/blog-images/ai/2.png)

这样就通过 Ollama + Open WebUI 本地部署了 DeepSeek-R1