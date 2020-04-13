###### 虚拟化容器技术，Docker基于镜像，可以秒级启动各种容器，每一种容器都是一个完整的运行环境，容器之间互相隔离

每个容器相当于 linux 系统，可以使用 **docker exec -it 20089** /bin/bash；进入到容器id为 20089 的容器里面去

* yum install docker   安装docker
* 启动 systemctl start docker
* 查看版本号 docker -v
* docker search mysql

![image-20200403210938085](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200403210938085.png)

* docker images 查看所有的镜像
* docker pull 镜像名：标签（不填默认是最新的版本）
* docker rmi 镜像id/标签 
  * 删除镜像id，会先删除所有的 tag，在删除镜像本身
  * 删除标签，就只是删除标签而已
* docker run --name 名字 -d(后台运行)  redis:tags
* docker ps      查看运行中的容器
* docker stop 容器的id     停止正在运行的容器
* docker start 容器的id    重新启动运行的容器
* docker ps -a        查看所有的容器
* docker rm 容器的id      删除容器



* docker run -d -p 主机端口:容器内部端口 镜像名字
* docker logs 容器名称/容器id
* docker tag  镜像名 镜像别名





* 我们在使用 Docker 一段时间后，系统一般都会残存一些临时的、没有被使用的镜像文件  
  *  docker image prune





/mydata/mysql/log 是 linux 下的配置文件

/var/log/mysql 是容器中的配置文件

这样可以做一个映射，我们在 linux 下修改配置就可以了

-e 是修改 mysql 的一些参数

![image-20200404102514567](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200404102514567.png)

**docker run -p 6379:6379 --name redis -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf  -d redis redis-server /etc/redis/redis.conf**