* yum install docker   安装docker
* 启动 systemctl start docker
* 查看版本号 docker -v
* docker search mysql

![image-20200403210938085](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200403210938085.png)

* docker images 查看所有的镜像
* docker pull 镜像名：标签（不填默认是最新的版本）
* docker rmi 镜像id 删除镜像
* docker run --name 名字 -d(后台运行)  redis:tags
* docker ps      查看运行中的容器
* docker stop 容器的id     停止正在运行的容器
* docker start 容器的id    重新启动运行的容器
* docker ps -a        查看所有的容器
* docker rm 容器的id      删除容器



* docker run -d -p 主机端口:容器内部端口 镜像名字
* docker logs 容器名称/容器id

