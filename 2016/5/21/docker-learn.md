<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>docker学习笔记</title>
  <link rel="stylesheet" href="http://app.classeur.io/base-min.css" />
  <script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML"></script>
</head>

<body>
  <div class="export-container"><p>@(学习)[docker, noteton]</p>
<h2 id="docker学习笔记">docker学习笔记</h2>
<p>GitBook <a href="https://www.gitbook.com/book/yeasy/docker_practice/details">Docker —— 从入门到实践</a></p>
<p>目录 ：</p>
<div class="toc"><ul><li><ul><li><a href="#docker学习笔记">docker学习笔记</a><ul><li><ul><li><a href="#运行容器">运行容器</a></li><li><a href="#进入容器">进入容器</a></li><li><a href="#终止容器">终止容器</a></li><li><a href="#启动已终止容器">启动已终止容器</a></li><li><a href="#轻量级的虚拟化">轻量级的虚拟化</a></li><li><a href="#commit提交修改">commit提交修改</a></li><li><a href="#根据当前dockerfile创建镜像">根据当前dockerfile创建镜像</a></li><li><a href="#从本地文件导入">从本地文件导入</a></li><li><a href="#存出镜像">存出镜像</a></li><li><a href="#载入镜像">载入镜像</a></li><li><a href="#移除本地镜像">移除本地镜像</a></li><li><a href="#清理所有未打过标签的本地镜像">清理所有未打过标签的本地镜像</a></li><li><a href="#nsenter-命令">nsenter 命令</a><ul><li><a href="#安装">安装</a></li><li><a href="#使用">使用</a></li></ul></li><li><a href="#运行dockerui">运行DockerUI</a></li></ul></li></ul></li></ul></li></ul></div><p>一张图总结 Docker 的命令<br>
<img src="https://i.imgur.com/IVtcpGn.png" alt="enter image description here"></p>
<h4 id="运行容器">运行容器</h4>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> docker run <span class="token operator">-</span>ti ubuntu<span class="token punctuation">:</span><span class="token number">14.04</span> <span class="token operator">/</span>bin<span class="token operator">/</span><span class="token function">bash</span>
</code></pre>
<p><code>14.04</code>是tag，用于区分，不指定时默认使用<code>latest</code>tag信息<br>
启动后attache进容器，运行<code>/bin/bash</code><br>
<code>-t</code> 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， <code>-i</code>则让容器的标准输入保持打开。加上<code>-d</code>表示后台运行。<br>
<code>docker ps</code>和<code>docker logs [container ID or NAMES]</code>查看运行容器和日志</p>
<p>运行时，后台执行的标准操作包括：</p>
<ul>
<li>检查本地是否存在指定的镜像，不存在就从公有仓库下载</li>
<li>利用镜像创建并启动一个容器</li>
<li>分配一个文件系统，并在只读的镜像层外面挂载一层可读写层</li>
<li>从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去</li>
<li>从地址池配置一个 ip 地址给容器</li>
<li>执行用户指定的应用程序</li>
<li>执行完毕后容器被终止</li>
</ul>
<h4 id="进入容器">进入容器</h4>
<p><code>docker attach</code>或 nsenter 工具。<br>
*<strong>注意</strong>：当多个窗口同时 attach 到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时,其他窗口也无法执行操作了。</p>
<h4 id="终止容器">终止容器</h4>
<p>可以使用 <code>docker stop</code> 来终止一个运行中的容器。<br>
此外，当Docker容器中指定的应用终结时，容器也自动终止。 例如对于上一章节中只启动了一个终端的容器，用户通过 exit 命令或 Ctrl+d 来退出终端时，所创建的容器立刻终止.</p>
<h4 id="启动已终止容器">启动已终止容器</h4>
<p>可以利用 <code>docker start</code> 命令，直接将一个已经终止的容器启动运行。或者<code>docker restart</code>重启容器。</p>
<h4 id="轻量级的虚拟化">轻量级的虚拟化</h4>
<p>容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。可以在伪终端中利用 <code>ps</code> 或 <code>top</code> 来查看进程信息。</p>
<pre class=" language-bash"><code class="prism  language-bash">root@ba267838cc1b<span class="token punctuation">:</span><span class="token operator">/</span><span class="token comment" spellcheck="true"># ps
</span>  PID TTY          TIME CMD
    <span class="token number">1</span> <span class="token operator">?</span>        <span class="token number">00</span><span class="token punctuation">:</span><span class="token number">00</span><span class="token punctuation">:</span><span class="token number">00</span> <span class="token function">bash</span>
   <span class="token number">11</span> <span class="token operator">?</span>        <span class="token number">00</span><span class="token punctuation">:</span><span class="token number">00</span><span class="token punctuation">:</span><span class="token number">00</span> <span class="token function">ps</span>
</code></pre>
<p>可见，容器中仅运行了指定的 bash 应用。这种特点使得 Docker 对资源的利用率极高，是货真价实的轻量级虚拟化。</p>
<h4 id="commit提交修改">commit提交修改</h4>
<p>可以在里面做一些更改，如<br>
<code>sudo apt-get install something</code><br>
然后<code>commit</code>得到一个新的镜像</p>
<pre><code>sudo docker commit -m "I install a cool thing" -a "liuyc" 0b2616b0e5a8 ouruser/sinatra:v2
</code></pre>
<h4 id="根据当前dockerfile创建镜像">根据当前dockerfile创建镜像</h4>
<p><code>commit</code>的方式容易创建镜像但不利于分享，于是用Dockerfile</p>
<pre class=" language-bash"><code class="prism  language-bash">FROM pdr2<span class="token punctuation">.</span>qa<span class="token punctuation">:</span><span class="token number">5043</span><span class="token operator">/</span>debug<span class="token operator">/</span>tomcat6
ADD pptv<span class="token punctuation">.</span>conf <span class="token operator">/</span>usr<span class="token operator">/</span>local<span class="token operator">/</span>tomcat<span class="token operator">/</span>conf<span class="token operator">/</span>pptv<span class="token punctuation">.</span>conf

<span class="token comment" spellcheck="true"># 听云Server探针的运行和配置文件添加到tomcat根目录
</span><span class="token comment" spellcheck="true"># catalina.sh里加入了-javaagent参数用于tomcat启动时运行听云探针
</span>ADD tingyun  <span class="token operator">/</span>usr<span class="token operator">/</span>local<span class="token operator">/</span>tomcat<span class="token operator">/</span>tingyun
COPY catalina<span class="token punctuation">.</span>sh <span class="token operator">/</span>usr<span class="token operator">/</span>local<span class="token operator">/</span>tomcat<span class="token operator">/</span>bin<span class="token operator">/</span>catalina<span class="token punctuation">.</span>sh

EXPOSE <span class="token number">8888</span>
RUN <span class="token operator">/</span>usr<span class="token operator">/</span>bin<span class="token operator">/</span><span class="token function">ssh</span><span class="token operator">-</span>keygen <span class="token operator">-</span>A
RUN <span class="token keyword">echo</span> <span class="token string">'root:oak'</span> <span class="token operator">|</span> chpasswd
EXPOSE <span class="token number">22</span>

<span class="token comment" spellcheck="true"># 设置Java环境变量
</span>ENV PATH <span class="token property">${PATH}</span><span class="token punctuation">:</span><span class="token property">${JAVA_HOME}</span><span class="token operator">/</span>bin<span class="token operator">/</span>
ENV LANG en_US<span class="token punctuation">.</span>UTF<span class="token operator">-</span><span class="token number">8</span>
<span class="token comment" spellcheck="true"># 将听云证书添加到JDK证书中，详情可见key.sh
</span>ADD tingyunCert <span class="token operator">/</span>tmp
WORKDIR <span class="token operator">/</span>tmp
RUN sh key<span class="token punctuation">.</span>sh

CMD  <span class="token operator">/</span>usr<span class="token operator">/</span>sbin<span class="token operator">/</span>sshd <span class="token operator">&amp;&amp;</span> <span class="token function">chmod</span> <span class="token operator">+</span>x <span class="token operator">/</span>usr<span class="token operator">/</span>local<span class="token operator">/</span>tomcat<span class="token operator">/</span>bin<span class="token operator">/</span>catalina<span class="token punctuation">.</span>sh <span class="token operator">&amp;&amp;</span>  <span class="token operator">/</span>usr<span class="token operator">/</span>local<span class="token operator">/</span>tomcat<span class="token operator">/</span>bin<span class="token operator">/</span>catalina<span class="token punctuation">.</span>sh run
</code></pre>
<p><code>#</code>注释，<code>FROM</code>指定镜像基础，<code>RUN</code>后的命令会在创建过程中执行<br>
ADD 命令复制本地文件到镜像；用 EXPOSE 命令来向外部开放端口；用 CMD 命令来描述容器启动后运行的程序等。<br>
<code>CMD</code>也可以写成这样的格式<code>CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]</code></p>
<pre class=" language-bash"><code class="prism  language-bash">docker build <span class="token operator">-</span>t tomcat6<span class="token operator">-</span>tingyun <span class="token punctuation">.</span>  <span class="token comment" spellcheck="true"># 用当前路径下的dockerfile创建镜像
</span>docker images` <span class="token comment" spellcheck="true">#查看镜像
</span>docker tag f358d5297fb7 pdr2<span class="token punctuation">.</span>qa<span class="token punctuation">:</span><span class="token number">5043</span><span class="token operator">/</span>debug<span class="token operator">/</span>tomcat6<span class="token operator">-</span>tingyun  <span class="token comment" spellcheck="true">#将镜像重命名
</span>docker push pdr2<span class="token punctuation">.</span>qa<span class="token punctuation">:</span><span class="token number">5043</span><span class="token operator">/</span>debug<span class="token operator">/</span>tomcat6<span class="token operator">-</span>tingyun <span class="token comment" spellcheck="true"># 推到远程
</span></code></pre>
<p>Dockfile 中的指令被一条一条的执行。每一步都创建了一个新的容器，在容器中执行指令并提交修改（就跟之前介绍过的 docker commit 一样）。当所有的指令都执行完毕之后，返回了最终的镜像 id。所有的中间步骤所产生的容器都被删除和清理了。<br>
*<strong>注意</strong>：一个镜像不能超过 127 层</p>
<h4 id="从本地文件导入">从本地文件导入</h4>
<p><code>sudo cat ubuntu-14.04-x86_64-minimal.tar.gz |docker import - ubuntu:14.04</code><br>
用<code>docker images</code>可以看到多了一个</p>
<h4 id="存出镜像">存出镜像</h4>
<p><code>sudo docker save -o ubuntu_14.04.tar ubuntu:14.04</code></p>
<h4 id="载入镜像">载入镜像</h4>
<p>可以使用 <code>docker load</code> 从导出的本地文件中再导入到本地镜像库，例如<br>
<code>$ sudo docker load --input ubuntu_14.04.tar</code><br>
或<br>
<code>$ sudo docker load &lt; ubuntu_14.04.tar</code><br>
这将导入镜像以及其相关的元数据信息（包括标签等）。</p>
<h4 id="移除本地镜像">移除本地镜像</h4>
<p><code>docker rmi</code> , 注意 <code>docker rm</code> 命令是移除容器<br>
*<strong>注意</strong>：在删除镜像之前要先用 docker rm 删掉依赖于这个镜像的所有容器。</p>
<h4 id="清理所有未打过标签的本地镜像">清理所有未打过标签的本地镜像</h4>
<p>使用下面的命令可以清理所有未打过标签的本地镜像<br>
<code>$ sudo docker rmi $(docker images -q -f "dangling=true")</code><br>
其中 -q 和 -f 是缩写, 完整的命令其实可以写着下面这样，是不是更容易理解一点？<br>
<code>$ sudo docker rmi $(docker images --quiet --filter "dangling=true")</code></p>
<h4 id="nsenter-命令">nsenter 命令</h4>
<h5 id="安装">安装</h5>
<p>nsenter 工具在 util-linux 包2.23版本后包含。 如果系统中 util-linux 包没有该命令，可以按照下面的方法从源码安装。</p>
<pre><code>$ cd /tmp; curl https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz | tar -zxf-; cd util-linux-2.24;
$ ./configure --without-ncurses
$ make nsenter &amp;&amp; sudo cp nsenter /usr/local/bin
</code></pre>
<h5 id="使用">使用</h5>
<p>nsenter 可以访问另一个进程的名字空间。nsenter 要正常工作需要有 root 权限。 很不幸，Ubuntu 14.04 仍然使用的是 util-linux 2.20。安装最新版本的 util-linux（2.24）版，请按照以下步骤：</p>
<pre><code>$ wget https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz; tar xzvf util-linux-2.24.tar.gz
$ cd util-linux-2.24
$ ./configure --without-ncurses &amp;&amp; make nsenter
$ sudo cp nsenter /usr/local/bin
</code></pre>
<p>为了连接到容器，你还需要找到容器的第一个进程的 PID，可以通过下面的命令获取。<br>
<code>PID=$(docker inspect --format "{{ .State.Pid }}" &lt;container&gt;)</code><br>
通过这个 PID，就可以连接到这个容器：<br>
<code>$ nsenter --target $PID --mount --uts --ipc --net --pid</code><br>
下面给出一个完整的例子。</p>
<pre><code>$ sudo docker run -idt ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           nostalgic_hypatia
$ PID=$(docker-pid 243c32535da7)
10981
$ sudo nsenter --target 10981 --mount --uts --ipc --net --pid
root@243c32535da7:/#
</code></pre>
<p>更简单的，建议大家下载 .bashrc_docker，并将内容放到 .bashrc 中。</p>
<pre><code>$ wget -P ~ https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker;
$ echo "[ -f ~/.bashrc_docker ] &amp;&amp; . ~/.bashrc_docker" &gt;&gt; ~/.bashrc; source ~/.bashrc
</code></pre>
<p>这个文件中定义了很多方便使用 Docker 的命令，例如 <code>docker-pid</code> 可以获取某个容器的 PID；而 <code>docker-enter</code> 可以进入容器或直接在容器内执行命令。</p>
<pre><code>$ echo $(docker-pid &lt;container&gt;)
$ docker-enter &lt;container&gt; ls
</code></pre>
<h4 id="运行dockerui">运行DockerUI</h4>
<pre class=" language-bash"><code class="prism  language-bash">docker run <span class="token operator">-</span>d <span class="token operator">-</span>p <span class="token number">9000</span><span class="token punctuation">:</span><span class="token number">9000</span> <span class="token operator">--</span>privileged <span class="token operator">-</span><span class="token function">v</span> <span class="token operator">/</span>var<span class="token operator">/</span>run<span class="token operator">/</span>docker<span class="token punctuation">.</span>sock<span class="token punctuation">:</span><span class="token operator">/</span>var<span class="token operator">/</span>run<span class="token operator">/</span>docker<span class="token punctuation">.</span>sock uifd<span class="token operator">/</span>ui<span class="token operator">-</span><span class="token keyword">for</span><span class="token operator">-</span>docker
</code></pre>
<p>打开  http://:9000</p></div>
</body>

</html>
