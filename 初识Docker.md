# docker镜像
## 查看镜像
`docker images`   
`docker images -q  查看所用镜像的id`
## 搜索镜像
`docker search 镜像名称`

## 拉取镜像
从docker仓库下载镜像到本地，镜像名称格式为名称：版本号

`docker pull 镜像名称`  
## 删除镜像
`docker rmi 镜像id # 删除指定本地镜像`        
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
`问：`   
为什么数据卷基于容器内部创建，容器销毁数据卷不受影响？    
因为双向绑定数据，在宿主机里有一份同样的数据，新建容器时数据会自动到新建容器里。
# docker网络
由于内容较长，单独一章讲解
# docker仓库
在我们刚使用docker，如果想要看到点什么东西的时候，就会涉及到web服务器，例：nginx/tomcat。就需要去dockerhub上拉取下来镜像。  
我们在开发时，可以制作自己的docker镜像上传到dockerhub上，提供给他人访问。

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
<!-- ![节点](./1.PNG) -->

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