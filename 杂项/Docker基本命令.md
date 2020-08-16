# Docker入门

## 1.Docker Hello World

使用Docker执行一个最基本的镜像,让他输出Hello World

```sh
docker run ubuntu:15.10 /bin/echo "hello world"
```

在这里,run表示运行一个容器,ubuntu:15.10表示要进行运行的镜像,将镜像创建为一个容器.ubuntu为镜像名,15.10为版本号

/bin/echo 表示在启动的容器里面要执行的命令

以上命令完整的意思可以解释为：Docker 以 ubuntu15.10 镜像创建一个新容器，然后在容器里执行 bin/echo "Hello world"，然后输出结果。

## 2.运行交互式容器

```shell
docker run -i -t ubuntu:15.10 /bin/bash
```

- -t:表示在容器内指定一个终端或者伪终端
- -i:允许你对容器内的标准输入进行交互

## 3.以后台模式运行容器

```shell
docker run -d ubuntu:15.10 /bin/bash -c "while true;do echo hello world;sleep 1;done"
```

在这里,我们使用了-d使容器运行在后台,并用-c指定运行的命令

在运行这条命令后,我们得到了一串长字符串(每个人不同)

> c13c8395ed7661c5866ec3ed52b9b45a183bde95db32df61c2390352ffbf924e

这个就是容器的识别码了,我们可以使用`docker ps`看到正在运行的容器

还可以使用`docker logs [docker_code]`来查看容器内的标准输出

# Docker基本概念

我们在上面的程序中,以不同的方式运行了docker的镜像,并令其执行了我们指定的命令.那么,docker是怎么运行的呢?

首先,我们要知道,docker是以CS(客户端/服务机)模式运行的,我们使用的docker命令是docker的客户机,服务机是在后台以守护进程的方式运行的.

在docker容器的概念中,镜像与容器的关系就好像面向对象中类和对象的关系一样

| Docker | 面向对象 |
| :----- | :------- |
| 容器   | 对象     |
| 镜像   | 类       |

而我们也要知道几个基本的docker概念

| Docker 镜像(Images)    | Docker 镜像是用于创建 Docker 容器的模板。                    |
| ---------------------- | ------------------------------------------------------------ |
| Docker 容器(Container) | 容器是独立运行的一个或一组应用。                             |
| Docker 客户端(Client)  | Docker 客户端通过命令行或者其他工具使用 Docker API (https://docs.docker.com/reference/api/docker_remote_api) 与 Docker 的守护进程通信。 |
| Docker 主机(Host)      | 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。       |
| Docker 仓库(Registry)  | Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。Docker Hub([https://hub.docker.com](https://hub.docker.com/)) 提供了庞大的镜像集合供使用。 |
| Docker Machine         | Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。 |

# 构建web应用

我们可以试一下使用docker构建一个web应用

## 运行Flask实例

```shell
docker run -d -P training/webapp python app.py
```

这个命令的作用是运行一个Flask后端的web应用,-P是将容器内部使用的网络端口映射到我们使用的主机上面.

如果我们再使用`docker ps`来看看,就会发现在这次的容器中,不仅有容器启动的信息,还会有对应的端口号

```
737c53e4d641        training/webapp     "python app.py"          31 seconds ago      Up 30 seconds       0.0.0.0:32768->5000/tcp   trusting_rubin
```

其中 0.0.0.0:32768->5000/tcp 正是表示Docker将容器内的5000端口映射到我们主机的端口32769上面,我们可以使用这个端口加上服务器的地址访问这个网页,输出`Hello world!`

我们可以使用`docker logs -f `来像tail一样检视日志,使用`docker top`来查看使用的进程

## 停止实例

`docker stop [container_name | container_code]`

## 删除实例

`docker rm [container_name | container_code]`