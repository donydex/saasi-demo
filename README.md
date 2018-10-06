# saasi-demo
# 实验步骤:
cd ~

git clone https://github.com/donydex/saasi-demo.git
# 配置mvn阿里云镜像
路径:

/usr/local/webserver/apache-maven-3.3.9/conf
添加:

<mirror>  
    <id>nexus-aliyun</id>  
    <mirrorOf>central</mirrorOf>    
    <name>Nexus aliyun</name>  
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>  
</mirror> 
------------------------------------------
# 数据库安装部分：
查找docker镜像
docker search 镜像关键字

docker 安装mysql

http://www.runoob.com/docker/docker-install-mysql.html

docker pull mysql:5.7

(最新的8会报错)

docker run -p 3306:3306 --name mymysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

docker exec -it mymysql /bin/bash

mysql -uroot -p123456

从宿主机拷贝文件到容器
https://blog.csdn.net/dongdong9223/article/details/71425077

docker cp $PWD/springboot.sql mymysql:/home/

注意容器名后没有空格

/home/springboot.sql

mysql运行命令行
http://database.51cto.com/art/201107/277687.htm

第一种方式：在未连接数据库的情况下，输入 mysql -h localhost -u root -p 123456  < /home/springboot.sql 回车即可；

第二种方式：在已连接数据库的情况下，此时命令提示符为mysql>，输入 source /home/springboot.sql。

# Demo 镜像构建&运行容器部分

cd springboot-mybatis-demo

mvn clean install

写dockerfile

使用dockerfile：
https://www.jianshu.com/p/93a678d1bde6

```
FROM java:8-jre

RUN mkdir -p /app/
ADD ./target/springboot-mybatis-demo-0.0.1-SNAPSHOT.jar /app/
CMD ["java", "-Xmx200m", "-jar", "/app/springboot-mybatis-demo-0.0.1-SNAPSHOT.jar"]

EXPOSE 8080
```
## 创建镜像:
docker build --rm -t springboot-mybatis-demo:base .
  
## 运行容器：
docker run -p 8080:8080 --name springboot-mybatis-demo -d springboot-mybatis-demo:base

注意，此处应该加 --net = host，因为docker会给启动的容器自动配置ip，但是项目里配置的mysql地址是localhost，在容器内连接localhost是连接容器本身而不是宿主机的localhost，所以一直找不到mysql。所以采用host的网络方式将容器与宿主机共用一个Network Namespace,这样容器内localhost就是宿主机的localhost了。

参考链接: https://blog.csdn.net/begin1013/article/details/80860224

docker run -p --net=host 8080:8080 --name springboot-mybatis-demo -d springboot-mybatis-demo:base


  
## 查看docker log
sudo docker logs -f -t --tail 行数 容器名
  
## linux批量查找文件内容
https://blog.csdn.net/ilove737/article/details/48372281
  
  
-----------------------------------
## 重命名docker容器名
docker rename old容器名  new容器名


