[toc]

# Docker

​		**官网: [Docker](https://www.docker.com/)**

​		**基于Go语言实现的应用容器引擎，其将应用与其依赖环境封装到一个可移植的容器中，使得应用能够持续集成并整体发布，从而做到“一次封装，处处运行”。**

---

## 前提

#### 比较

​		相比于传统虚拟化方式——虚拟机，Docker容器内的应用进程直接运行于宿主内核，并不对构建单独的操作系统和对硬件进行虚拟，每个容器间保证隔离，使得虚拟化环境更加轻便高效。

#### 概念

​		Docker的三大要素：容器(Container)、镜像(Image)、仓库(Repository)

- 容器是镜像创建出的示例，是独立运行的应用，可以被看作是**精简的**Linux环境及运行在其上的应用，其可以被启动、开始、停止和删除，容器间相互隔离。
- 镜像是容器的只读模板，可以用于创建多个Docker容器。镜像内部是分层结构，不同镜像可以共享相同的层。
- 仓库是集中存放镜像文件的场所，其又被存放在仓库注册服务器中，每个镜像有不同的标签(tag)，最大的公开仓库是[Docker Hub](https://hub.docker.com/)

#### 运行环境

​		如CentOS为例，运行在CentOS 7上时，系统内核版本需要在3.10以上。运行在CentOS 6.5及以上时，版本系统内核版本需要在2.6.32-431及以上。系统都必须为64位。

​		*Linux系统版本可以通过Linux命令`lsb_release -a`查看，内核版本和系统位数可以通过`uanme -r`进行查看*

---

## 安装与配置

#### 安装

​		*安装前都需要检查是否有旧版本的Docker并卸载，此处省略。*

##### CentOS 6.8

​		CentOS 6.8默认使用Docker-io版本

1. 安装EPEL源

``` 
yum install -y epel-release
```

2. 安装Docker-io

```
yum install -y docker-io
```

​		**注意：如果出错则用下载源安装命令代替，命令如下**

```
yum install -y https://get.docker.com/rpm/1.7.1/centos-6/RPMS/x86_64/docker-engine-1.7.1-1.el6.x86_64.rpm
```

3. 启动Docker服务

```
service docker start
```

##### CentOS 7+

​		[官方安装指引](https://docs.docker.com/install/linux/docker-ce/centos/)，CentOS 7以上版本默认使用Docker-ce版本

1. 安装工具包

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

2. 添加存储库

   ​		第一个为官方源，第二个为阿里云源，建议使用阿里云的源。

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

3. 更新索引(可省略)

```
yum makecache fast
```

4. 安装Docker-ce

```
yum install -y docker-ce
```

5. 启动Docker

```
systemctl start docker
```



#### 配置阿里云镜像加速服务

##### CentOS 6.8

1. 登录[阿里云容器镜像服务](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)，点击 镜像中心-镜像加速器 获取专属加速器地址

2. 输入命令`vim /etc/sysconfig/docker`修改Docker配置文件，将`other_args`修改为如下

```
other_args="--registry-mirror=加速服务网址"
```

3. 重启服务`service docker restart`
4. 测试配置是否已经被Docker使用，执行命令`ps -ef|grep docker`，并查看是否有加速器地址

##### CentOS 7+

1. 通过命令`vim /etc/docker/daemon.js`创建并Docker配置文件，内容如下

```
{
  "registry-mirrors": ["加速服务网址"]
}
```

2. 重启Docker并使得配置生效

```
systemctl daemon-reload && systemctl restart docker
```



#### Hello World

​		执行命令`docker run hello-world`，Docker会先从本地仓库寻找该镜像，确定没有后从远程镜像仓库(Docker Hub)拉取镜像，查看是否能执行该流程并获得`Hello from Docker!`信息。

---

## 命令

​		[Docker命令大全](https://www.runoob.com/docker/docker-command-manual.html)

#### 帮助命令

- `docker version `查看Docker版本相关信息
- `docker info`  查看Docker系统信息
- `docker --help` 查看Docker详细帮助



#### 镜像命令

- `docker images [参数]` 查看本地仓库所有镜像信息，如果在参数后指定镜像名则查看对应的所有本地镜像

  ​		**参数：**-a 显示所有镜像(含中间映像层)，-q 仅显示镜像id，--digests 额外显示摘要信息，--no-trunc 显示完整的镜像信息，包括完整镜像id和摘要信息

- `docker search [参数] [镜像名]` 从Docker Hub中搜索指定镜像，**注意镜像名可以为名字或id，下同**

  ​		**参数：** -s 仅显示stars数比给定数量(跟在-s后面)多的镜像，--no-trunc 显示完整的镜像信息，--automated 只列出自动构建类型的镜像

- `docker pull [镜像名]:[标签]` 从Docker Hub(或配置的远程镜像仓库)下载镜像 ，**不指定标签均默认为最新版(即latest，下省略)**

- `docker rmi [镜像名]:[标签]` 删除指定本地镜像，删除多个镜像则镜像名间通过空格分开

  ​		**参数：** -f 强制删除(包括正在运行的镜像)

  ​     `docker rmi $(docker images -qa)` 删除所有本地镜像，实际是获得本地所有镜像id并传给删除命令。

- `docker history [镜像ID]` 查看镜像构建过程，一般用于再现用DockerFile构建镜像的原过程



#### 容器命令

- `docker run [参数] [镜像名] [运行命令] [命令参数]` 创建新的容器并执行指定命令

  ​		**参数：** -d 后台运行容器并返回id(守护式容器，前台没有对应的进程运行将立即结束)，--name 为容器指定一个新名字，-e  指定环境变量

  ​					  -i 以交互模式运行，-t 为容器分配一个伪输入终端（**以上两个参数一般一起使用，效果是在Docker宿主机的控制台直接和运行容器交互**)

  ​					  -P 随机端口映射，-p 主机端口:容器端口 通过指定主机端口映射容器端口

  ​					  -v 指定数据卷，详细见下。

  ​		*如果通过Docker运行CentOS镜像提示FATAL:Kernel too old，即内核版本太低，[解决方案](https://blog.csdn.net/qq_34430649/article/details/103603294)*

- `exit`退出并停止当前容器 , Ctrl + P + Q 退出并后台运行容器

- `docker ps [参数]` 列出正在运行的容器

  ​		**参数：** -a 列出正在运行以及历史运行的容器，-l 列出最近创建的容器，-n 列出最近指定次数运行的容器，-q 只显示容器id

- `docker start [容器名] ` 启动已经创建的容器，**容器名可以为名字或id，下同**

- `docker restart [容器名]` 重启正在运行的容器

- `docker stop [容器名]` 停止正在运行的容器， `docker kill [容器名]` 强制停止

- `docker rm [容器ID]` 删除已停止的容器，**注意必须不能是容器名，且区分rm是删除容器，rmi是删除镜像**

  ​		删除所有容器可以通过`docker rm -f $(docker ps -aq)`，删除挂载的数据卷添加参数 -v

- `docker logs [参数] [容器ID]` 

  ​		**参数：**-t 加入时间戳，-f  实时更新日志，--tail 仅显示最后指定数量条日志

- `docker top [容器ID]` 查看指定容器内运行的进程，`docker inspect [容器ID]` 查看容器内部细节，以JSON串显示

- `docker attach [容器ID]`   开启并进入正在运行容器的交互终端(不建议使用)，  `docker exec -it [容器ID] [命令]]` 在一个正在运行的容器中执行命令，如命令部分为`/bin/bash`则以CentOS环境的shell进入运行容器所在目录

- `docker cp [容器ID]:[容器内路径] [主机路径]` 将容器内指定文件复制到宿主机中

- `docker commit [参数] [容器ID] ` 从容器创建一个镜像

  ​		**参数：**-a 提交镜像的作者，-m 提交的说明，-p commit时将容器暂停

###### 常用启动命令示例

- Tomcat

  ​		创建和webapps目录绑定的数据卷，并指定8080端口

```
docker run -d -p 8080:8080 -v 宿主机目录:/usr/local/tomcat/webapps --name tomcat tomcat:version
```

- MySQL

  ​		通过指定环境变量来指定root账户登陆密码，并指定3306端口

```
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql mysql:version
```

- Redis

  ​		创建和Redis持久化数据目录以及配置文件绑定的数据卷，执行启动命令`redis-server`，通过运行参数指定开启数据持久化，并指定6379端口

```
docker run -d -p 6379:6379 -v 宿主机目录:/data -v 宿主机文件目录:/usr/local/etc/redis/redis.conf --name redis redis redis-server --appendonly yes
```

#### 数据卷命令

​		数据卷(Volume)是一种实现容器与宿主机共享数据、容器数据持久化、容器之间共享文件的功能，数据卷的更新修改不会影响镜像，并且其内容会保持到没有被任何容器使用。该功能均作为`docker run `命令的参数来使用。

- **容器和宿主间共享**

  ​		`docker run -v /宿主机目录:/容器目录 [镜像名]`从指定镜像创建容器并挂载数据卷，即在宿主机和容器内创建这两个目录并绑定(文件保持同步)，从而容器和主机可以通过这个目录传输文件，在目录后添加 :ro ，使得容器内对该目录内文件只读。可以重复指定以创建多个数据卷。

- **容器间共享**

  ​		`docker run --volume-from [容器ID] [镜像名]` 从指定镜像创建容器并挂载指定容器中的数据卷，即指定容器(原运行中的容器)的数据卷共享列表中增加该新容器。

- **查看数据卷** 

  ​		查看Docker内所有数据卷 `docker volume list`

- **删除数据卷 **

  ​		 删除指定数据卷 `docker rm [数据卷名]`  清理所有无主数据卷`docker volume prune`

#### Dockerfile 命令

- `docker build [参数] [镜像名] [上下文目录]` 通过指定Dockerfile文件创建镜像，Docker将每行命令逐层构建镜像

  上下文目录指构建镜像时文件所在的目录，如果在Dockerfile文件目录运行命令时则为 .

​		**参数：**-f 指定Dockerfile文件路径，路径跟在参数后面(*当文件名为Dockerfile，且在当前目录，则可以省略该参数*)，-t 指定创建的镜像名和标签

###### 	示例

1. 在myDocker文件下新建Dockerfile文件，内容如下，

   ​		作用是引入centos镜像，创建两个目录如下的数据卷(将自动分配绑定的宿主机目录，启动后通过`docker inspect [容器ID]`查看)，并执行两行命令。  *Dockerfile命令详细见下节*

```
FROM centos
VOLUME ["/volumeContainer1","/volumeContainer2"]
CMD echo "success!"
CMD /bin/bash
```

2. 在该文件目录下执行命令`docker build -f Dockerfile -t priv/test .`

   ​		表示通过`/myDocker/Dockerfile`文件创建一个名为`priv/test`的镜像，构建时的上下文目录为当前目录，注意构建时会对Dockerfile文件中每行命令逐行构建层。

   *p.s. CentOS中粘贴出现前缀如 0~ 等错误情况时，执行命令`printf "\e[?2004l"`即可解决。*



#### 推送镜像到远程仓库

- 在[阿里云容器镜像服务](https://cr.console.aliyun.com/)中的 默认实例-镜像仓库 创建远程镜像仓库(*可以查看仓库中的操作指南*)

- Docker中登录到远程仓库

​		用户名为阿里云账户名，仓库地址在仓库中查看，输入命令后还要输入开通服务时设置的密码。

```
docker login --username=用户名 仓库地址
```

- 推送本地镜像到远程仓库

```
docker tag [镜像ID] 仓库地址/命名空间/仓库名:[镜像名:版本号]
docker push 仓库地址/命名空间/仓库名:[镜像名:版本号]
```

- 拉取远程仓库镜像到本地

```
docker pull 仓库地址/命名空间/仓库名:[镜像名:版本号]
```

---

## Dockerfile

​		Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和 参数构成的脚本。

#### 执行流程	

​		先从一切镜像的基础镜像——scratch(类似Java中的Object)运行一个容器，执行一条命令并对容器进行修改，类似commit操作提交一个镜像层，基于该镜像层再运行一个新容器，重复操作直至所有指令执行完毕。

#### 保留字指令

- `FROM` 当前镜像基于哪个镜像，所有镜像的基础镜像都是scratch
- `MAINTAINER` 镜像维护者的姓名和邮箱
- `RUN`镜像构建时需要运行的命令
- `EXPOSE` 容器暴露的端口号
- `WORKDIR` 容器创建后默认的工作目录
- `ENV [变量名] [变量值] ` 新建环境变量
- `ADD` 从宿主机目录下拷贝文件，并自动解压缩
- `COPY` 从构建上下文目录中拷贝文件或目录
- `VOLUME` 指定容器数据卷目录，用于数据共享和持久化
- `CMD` 指定容器启动时需要运行的命令，只有最后一个该命令会生效，且会被`docker run`中的参数替换
- `ENTRYPOINT` 同上，但是`docker run`中的参数只会追加而不会替换
- `ONBUILD` 若该镜像被继承，在子镜像构建时该命令会被执行

###### 示例一

​			以下Dockerfile文件指令功能依次为，继承镜像CentOS，指定维护者姓名为howard邮箱为604876547@qq.com，创建一个环境变量MYPATH值为/user/local，将镜像初始工作目录指定为MYPATH的值，运行命令以安装vim和网络支持工具，将80作为容器暴露端口，容器运行时运行/bin/bash的shell

```dockerfile
FROM centos
MAINTAINER howard<604876547@qq.com>

ENV MYPAHT /user/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD /bin/bash
```

######  示例二

```dockerfile
FROM centos
MAINTAINER howard<604876547@qq.com>

COPY c.txt /usr/local/cincontainer.txt

ADD jdk-8u231-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.30.tar.gz /usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_231
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.30
ENV CATALINA_B

ASE /usr/local/apache-tomcat-9.0.30
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

ENTRYPOINT ["/usr/local/apache-tomcat-9.0.30/bin/startup.sh"]
#CMD ["/usr/local/apache-tomcat-9.0.30/bin/catalina.sh","run"]
```

---

## Docker Compose

​		[官方文档](https://docs.docker.com/compose/)，[官方GitHub](https://github.com/docker/compose)，[第三方-参数列表](https://www.cnblogs.com/wutao666/p/11332186.html)

​		Docker Compose是 docker 提供的一个命令行工具，用来定义和运行由多个容器组成的应用。使用 compose，我们可以通过 YAML 文件声明式的定义应用程序的各个服务，并由单个命令完成应用的创建和启动。

##### 安装

1. 从[官方GitHub-releases](https://github.com/docker/compose/releases)中下载安装包

   ​		可通过以下命令完成，注意替换版本号，文件路径也可以替换。

```
curl -L https://github.com/docker/compose/releases/download/[版本号]/docker-compose-Linux-x86_64 -o /usr/local/bin/docker-compose
```

2. 为安装包添加可执行权限

```
chmod +x /usr/local/bin/docker-compose
```

3. 目录下通过`docker-compose version`安装并查看版本

##### 使用

1. 编写Docker Compose应用配置文件`docker-compose.yml`
2. 通过命令`docker-compose up -d`通过YML文件后台运行容器
3. 通过命令`docker-compose down`停止运行的容器

​	*粘贴yml文件时，可能会出现格式被破坏的情况，此时在vi编辑器中输入`:set paste`进入粘贴模式则可以保持格式。*

###### Tomcat示例

​		以下是一个运行tomcat容器的实例配置，`version`通过Docker Engine版本确定(*详情查看releases说明*)，`services`中指定多个容器的配置集合，指定服务名下`restart: always`指定该容器保持运行，`environment`中指定环境参数，其余参数和`docker run`参数大致对应。

```yml
version: '3.7'
services: 
  tomcat:
    restart: always
    image: tomcat
    container_name: tomcat
    ports:
      - 8080:8080
    volumes:
      - ./webapps:/usr/local/tomcat/webapps
    environment:
      TZ: Asia/Shanghai
```

###### MySQL示例

​		该示例中Docker Compose指定了多个容器的运行，第一个是MySQL服务容器配置，第二个是基于PHP的WEB数据库管理工具Adminer(可以忽略)。

​		`command`中指定的变量作用依次为：修改密码加密规则为mysql_native_password(MySQL8.0+后默认密码加密规则为caching_sha2_password，可能造成不兼容)；后两行都是指定数据库编码为UTF-8；接下来是允许timestamp类型数据为NULL；最后是指定对数据库、表、列名大小写不敏感(Linux系统对大小写默认敏感)。`volumes`中挂载的数据卷目录为MySQL数据存储目录。

```yml
version: '3.7'
services:
  mysql:
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
    ports:
      - 3306:3306
    volumes:
      - ./data:/var/lib/mysql
      
   adminer:
     imgae: adminer
     restart: always
     ports:
       - 8080:8080
```



