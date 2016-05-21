


## docker学习笔记
GitBook [Docker —— 从入门到实践](https://www.gitbook.com/book/yeasy/docker_practice/details)

目录 ：

[toc]

一张图总结 Docker 的命令
![enter image description here](https://i.imgur.com/IVtcpGn.png)

#### 运行容器
```bash
sudo docker run -ti ubuntu:14.04 /bin/bash
```
`14.04`是tag，用于区分，不指定时默认使用`latest`tag信息
启动后attache进容器，运行`/bin/bash`
`-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i `则让容器的标准输入保持打开。加上`-d`表示后台运行。
`docker ps`和`docker logs [container ID or NAMES]`查看运行容器和日志

运行时，后台执行的标准操作包括：
- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

#### 进入容器
`docker attach`或 nsenter 工具。
***注意**：当多个窗口同时 attach 到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时,其他窗口也无法执行操作了。

#### 终止容器
可以使用 `docker stop` 来终止一个运行中的容器。
此外，当Docker容器中指定的应用终结时，容器也自动终止。 例如对于上一章节中只启动了一个终端的容器，用户通过 exit 命令或 Ctrl+d 来退出终端时，所创建的容器立刻终止.

#### 启动已终止容器
可以利用 `docker start` 命令，直接将一个已经终止的容器启动运行。或者`docker restart`重启容器。

#### 轻量级的虚拟化
容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。可以在伪终端中利用 `ps` 或 `top` 来查看进程信息。
```bash
root@ba267838cc1b:/# ps
  PID TTY          TIME CMD
    1 ?        00:00:00 bash
   11 ?        00:00:00 ps
```
可见，容器中仅运行了指定的 bash 应用。这种特点使得 Docker 对资源的利用率极高，是货真价实的轻量级虚拟化。

#### commit提交修改
可以在里面做一些更改，如
`sudo apt-get install something`
然后`commit`得到一个新的镜像
```
sudo docker commit -m "I install a cool thing" -a "liuyc" 0b2616b0e5a8 ouruser/sinatra:v2
```

#### 根据当前dockerfile创建镜像
 `commit`的方式容易创建镜像但不利于分享，于是用Dockerfile
```bash
FROM pdr2.qa:5043/debug/tomcat6
ADD pptv.conf /usr/local/tomcat/conf/pptv.conf

# 听云Server探针的运行和配置文件添加到tomcat根目录
# catalina.sh里加入了-javaagent参数用于tomcat启动时运行听云探针
ADD tingyun  /usr/local/tomcat/tingyun
COPY catalina.sh /usr/local/tomcat/bin/catalina.sh

EXPOSE 8888
RUN /usr/bin/ssh-keygen -A
RUN echo 'root:oak' | chpasswd
EXPOSE 22

# 设置Java环境变量
ENV PATH ${PATH}:${JAVA_HOME}/bin/
ENV LANG en_US.UTF-8
# 将听云证书添加到JDK证书中，详情可见key.sh
ADD tingyunCert /tmp
WORKDIR /tmp
RUN sh key.sh

CMD  /usr/sbin/sshd && chmod +x /usr/local/tomcat/bin/catalina.sh &&  /usr/local/tomcat/bin/catalina.sh run
```
`#`注释，`FROM`指定镜像基础，`RUN`后的命令会在创建过程中执行
 ADD 命令复制本地文件到镜像；用 EXPOSE 命令来向外部开放端口；用 CMD 命令来描述容器启动后运行的程序等。
 `CMD`也可以写成这样的格式`CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]`

```bash
docker build -t tomcat6-tingyun .  # 用当前路径下的dockerfile创建镜像
docker images` #查看镜像
docker tag f358d5297fb7 pdr2.qa:5043/debug/tomcat6-tingyun  #将镜像重命名
docker push pdr2.qa:5043/debug/tomcat6-tingyun # 推到远程
```
Dockfile 中的指令被一条一条的执行。每一步都创建了一个新的容器，在容器中执行指令并提交修改（就跟之前介绍过的 docker commit 一样）。当所有的指令都执行完毕之后，返回了最终的镜像 id。所有的中间步骤所产生的容器都被删除和清理了。
***注意**：一个镜像不能超过 127 层

#### 从本地文件导入
`sudo cat ubuntu-14.04-x86_64-minimal.tar.gz  |docker import - ubuntu:14.04`
用`docker images`可以看到多了一个

#### 存出镜像
`sudo docker save -o ubuntu_14.04.tar ubuntu:14.04`

#### 载入镜像
可以使用 `docker load` 从导出的本地文件中再导入到本地镜像库，例如
`$ sudo docker load --input ubuntu_14.04.tar`
或
`$ sudo docker load < ubuntu_14.04.tar`
这将导入镜像以及其相关的元数据信息（包括标签等）。

#### 移除本地镜像
`docker rmi` , 注意 `docker rm` 命令是移除容器
***注意**：在删除镜像之前要先用 docker rm 删掉依赖于这个镜像的所有容器。

#### 清理所有未打过标签的本地镜像
使用下面的命令可以清理所有未打过标签的本地镜像
`$ sudo docker rmi $(docker images -q -f "dangling=true")`
其中 -q 和 -f 是缩写, 完整的命令其实可以写着下面这样，是不是更容易理解一点？
`$ sudo docker rmi $(docker images --quiet --filter "dangling=true")`


#### nsenter 命令

##### 安装
nsenter 工具在 util-linux 包2.23版本后包含。 如果系统中 util-linux 包没有该命令，可以按照下面的方法从源码安装。
```
$ cd /tmp; curl https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz | tar -zxf-; cd util-linux-2.24;
$ ./configure --without-ncurses
$ make nsenter && sudo cp nsenter /usr/local/bin
```
##### 使用
nsenter 可以访问另一个进程的名字空间。nsenter 要正常工作需要有 root 权限。 很不幸，Ubuntu 14.04 仍然使用的是 util-linux 2.20。安装最新版本的 util-linux（2.24）版，请按照以下步骤：
```
$ wget https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz; tar xzvf util-linux-2.24.tar.gz
$ cd util-linux-2.24
$ ./configure --without-ncurses && make nsenter
$ sudo cp nsenter /usr/local/bin
```
为了连接到容器，你还需要找到容器的第一个进程的 PID，可以通过下面的命令获取。
`PID=$(docker inspect --format "{{ .State.Pid }}" <container>)`
通过这个 PID，就可以连接到这个容器：
`$ nsenter --target $PID --mount --uts --ipc --net --pid`
下面给出一个完整的例子。
```
$ sudo docker run -idt ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           nostalgic_hypatia
$ PID=$(docker-pid 243c32535da7)
10981
$ sudo nsenter --target 10981 --mount --uts --ipc --net --pid
root@243c32535da7:/#
```
更简单的，建议大家下载 .bashrc_docker，并将内容放到 .bashrc 中。
```
$ wget -P ~ https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker;
$ echo "[ -f ~/.bashrc_docker ] && . ~/.bashrc_docker" >> ~/.bashrc; source ~/.bashrc
```
这个文件中定义了很多方便使用 Docker 的命令，例如 `docker-pid` 可以获取某个容器的 PID；而 `docker-enter` 可以进入容器或直接在容器内执行命令。
```
$ echo $(docker-pid <container>)
$ docker-enter <container> ls
```

#### 运行DockerUI
``` bash
docker run -d -p 9000:9000 --privileged -v /var/run/docker.sock:/var/run/docker.sock uifd/ui-for-docker
```
打开  http://<dockerd host ip>:9000




