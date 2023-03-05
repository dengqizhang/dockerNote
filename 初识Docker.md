# docker概念
初学docker 我就想知道他到底能做什么用，了解到了这个工具如下几个优点。
## docker的镜像，仓库和容器
dockerhub上面有一个非常强大的镜像库，就像github一样，一行命令就可以拉取（mysql，nginx，tomcat等）在宿主机上，可以通过容器运行镜像的实例。
## 开发简化环境的搭建
在docker里，拉取到宿主机的java项目，不需要考虑jdk的版本，不需要任何可以依赖运行的程序，直接在宿主机运行。像虚拟机一样，即开即用。
## docker和虚拟机相比的好处
1，占用资源
如果我们想要开启一个虚拟机，就要考虑分配操作系统的镜像，分配内存和cpu资源。而docker只是操作系统是上的一个进程，可以开很多个，节省了更多的资源。
# docker生命周期管理
docker run ：创建一个新的容器并运行一个命令   
`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`   
docker start :启动一个或多个已经被停止的容器   
`docker start [OPTIONS] CONTAINER [CONTAINER...]`   
docker stop :停止一个运行中的容器   
`docker stop [OPTIONS] CONTAINER [CONTAINER...]`   
docker restart :重启容器   
`docker restart [OPTIONS] CONTAINER [CONTAINER...]`
docker kill :杀掉一个运行中的容器  
`docker kill [OPTIONS] CONTAINER [CONTAINER...]`   
docker rm ：删除一个或多个容器。   
`docker rm [OPTIONS] CONTAINER [CONTAINER...]`   
docker exec ：在运行的容器中执行命令   
`docker exec [OPTIONS] CONTAINER COMMAND [ARG...]  `

# docker镜像管理
## 查看镜像
`docker images`   
`docker images -q  查看所用镜像的id`
## 搜索镜像
`docker search 镜像名称`

## 拉取镜像
从docker仓库下载镜像到本地，镜像名称格式为名称：版本号

`docker pull 镜像名称`  
## 删除镜像
`docker rmi 镜像id  删除指定本地镜像`        
`docker rmi "docker images -q"  删除所有本地镜像`

# docker容器
## 查看容器
`docker ps  查看正在运行的容器`    
`docker ps -a  ` 查看所有容器   
## 创建并启动容器
`docker run 参数`  
参数说明：   
-i:保持容器运行，和it两个参数一起使用后，容器创建后自动进入容器中，退出容器后容器自动关闭。    
-t:为容器重新分配一个伪终端    
-d：后台运行容器，退出不会关闭    
-it:创建的容器一般称为交互式容器，-id：创建的容器被称为守护式容器     
--name：为创建的容器命名   
## 进入容器
`docker exec 参数`退出容器后容器不会关闭。   
## 停止容器
`docker stop 容器名称`   
## 启动容器
`docker start 容器名称`
## 删除容器:容器不能在运行状态删除
`docker rm 容器名称`

## 查看容器信息
`docker inspect 容器名称`
# docker数据卷
## 数据卷概念
容器启动后的数据销毁数据也会跟随销毁，防止数据丢失可以使用数据卷保存数据，容器(目录)=宿主机(目录),宿主机和容器之间的数据是双向绑定的。   

卷就是目录或文件，存在于一个或多个容器中，由Docker挂载到容器，但卷不属于联合文件系统（Union FileSystem），因此能够绕过联合文件系统提供一些用于持续存储或共享数据的特性。
## 数据卷使用
运行容器，指定挂载数据卷命令：      
`docker run -it -v 主机目录:容器目录`     


在Linux下的MySQL默认的数据文档存储目录为/var/lib/mysql，默认的配置文件的位置/etc/mysql/conf.d，为了确保MySQL镜像或容器删除后，造成的数据丢失，下面建立数据卷保存MySQL的数据和文件。   
```
docker run -d -p 6603:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
```

# docker网络
## 默认网络
安装 Docker 以后，会默认创建四种网络，可以通过 docker network ls 查看。  
`bridge:  `  
为每个容器分配，设置ip地址，并将容器连接到一个docker0虚拟网桥，默认为该模式。我们新建的容器，如果不另外指定网络，就不需要考虑网络问题，默认在一个网络下可以进行通信。  
`host:`  
容器不会虚拟自己的网卡，以及配置ip，而是使用宿主机的。这样的好处是，在容器内因为共享了宿主机的ip地址，如果宿主机是一个共有ip，那么容器同样可以通过这个共有ip和外部进行通信。  
`none：`   
none 网络模式即不为 Docker Container 创建任何的网络环境，容器内部就只能使用 loopback 网络设备，不会再有其他的网络资源  
`container:`  
新创建的容器不会创建自己的网卡，配置自己的ip，而是和一个指定的容器共享。


# docker镜像仓库
我们在开发时，可以制作自己的docker镜像上传到dockerhub上，提供给他人访问,也可以在dockerhun仓库里拉取别人的镜像。

## 制作镜像
一、
`首先我们需要在服务器端root/下新建一个文件夹作为数据卷目录   `   
二、`写一个简单的java项目，本地测试运行`   
指定主类：
````
<plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>cn.lnfvc.hello</mainClass> <!-- 指定主类 -->
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
````

三、`建立docker镜像文件Dockerfile并配置`
```
FROM openjdk:8
COPY $PWD/文件名.jar/
CMD [“java”,"-jar","/文件名.jar"]
```

四、`Dockerfile和打包后的jar包上传到服务器端的html里`

五、在html文件夹里构建镜像`docker build -t 镜像名称 .`
六、用`docker images`查看有没有自己建的镜像

8.启动镜像
`docker run 镜像名称`

## 镜像推送
注意：在推送镜像之前需要先登录    
`docker login`    
一、首先确保创建了Docker Hub的账号，请记住账号和密码。   
二、在本地使用以下命令为您的镜像添加一个标记：   
```
docker tag [IMAGE_NAME] [DOCKERHUB_USERNAME]/[REPOSITORY_NAME]:[TAG_NAME]
```
其中，[IMAGE_NAME]是本地镜像的名称，[DOCKERHUB_USERNAME]是您的Docker Hub账号用户名，[REPOSITORY_NAME]是要创建的新镜像仓库名称，[TAG_NAME]是标签名称。   
三、将标记推送到Docker Hub：
`docker push [DOCKERHUB_USERNAME]/[REPOSITORY_NAME]:[TAG_NAME]`    
如果成功推送，现在打开Docker Hub上的存储库就可以找到自己推送上去的镜像。

## 镜像拉取
`docker pull 用户名/存储卡名称:标签`  
输入`docker imges`查看一下拉取镜像