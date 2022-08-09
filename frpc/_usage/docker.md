SakuraFrp 提供 [frpc 镜像](https://hub.docker.com/r/natfrp/frpc) 以便您借助 Docker 管理 frpc。

### 图形用户界面 :id=docker-gui

本教程只展示命令行操作，如果您使用以下平台，点击链接可查看对应的 GUI 操作说明：

 - [群晖 DSM](/app/synology)
 - [威联通 QNAP](/app/qnap)

### 设置隧道 :id=docker-create-tunnel

因为 docker 网络模型的原因，我们像从前一样把隧道的 本地IP 设置为 `127.0.0.1` 已经不再奏效，必须修改设置中的此项。

因为默认的 `bridge` 网桥模式兼容性和安全性更高，**下面的教程默认采用此方案**

首先运行下面的指令读出 docker 默认网桥的 网关IP

`ip addr show docker0`

结果预期如下：

```
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:c4:f5:83:8f brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```

其中 `172.17.0.1` 这一部分就是我们要的 IP

然后在新建隧道时将其设置为 本地IP

![](_images/docker-tunnel-new.png)

或者在 隧道列表 中，编辑一条隧道，设置 本地IP 为该 IP

![](_images/docker-tunnel-mod.png)

这样隧道就准备完了

### 设置Docker :id=docker-setup-docker

首先我们需要获取镜像：

```bash
# 默认 DockerHub 源，国内可能较慢：
docker pull natfrp/frpc

###### 或者 ######

# 阿里云容器镜像 香港地区源，适合国内用户：
docker pull registry.cn-hongkong.aliyuncs.com/natfrp/frpc
```

如果成功的话，返回应该会是下面这样：

```
~# docker pull registry.cn-hongkong.aliyuncs.com/natfrp/frpc
Using default tag: latest
latest: Pulling from natfrp/frpc
4c0d98bf9879: Pull complete 
292f768886fd: Pull complete 
Digest: sha256:9d33d6110ee53480f28cc99e39476d3d845ce70cf8a4d775da78f15620bbab5a
Status: Downloaded newer image for registry.cn-hongkong.aliyuncs.com/natfrp/frpc:latest
registry.cn-hongkong.aliyuncs.com/natfrp/frpc:latest
```

其中最后一行复制一下，这个是实际被下载到本地的镜像tag，启动时会用得上

接下来我们执行：`docker run -d --restart=always <你刚复制的镜像tag> -f <启动参数> --remote_control <远程控制密码>`，如果一切顺利，就能看到只有一行奇怪的hash的输出，就是实例ID

_* `--restart=always` 选项并不是必须，但开启此选项后可以自动重启容器实例_

### 获取连接信息 :id=docker-how-to-connect

连接信息在 docker实例 的日志中，执行 `docker logs <实例ID>` 就能看到，如对于这样的启动参数：

![](_images/docker-cli-run.png)

我们取实例ID的随便前几位就能查到日志：

![](_images/docker-cli-log.png)