"# saasi-demo" 
实验步骤:
到目标目录,此处以根目录为例子
cd ~
git clone https://github.com/donydex/saasi-demo.git
配置mvn阿里云镜像
路径:
/usr/local/webserver/apache-maven-3.3.9/conf
添加:
<mirror>  
    <id>nexus-aliyun</id>  
    <mirrorOf>central</mirrorOf>    
    <name>Nexus aliyun</name>  
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>  
</mirror> 

写dockerfile
使用dockerfile：
https://www.jianshu.com/p/93a678d1bde6
```
FROM java:8-jre

RUN mkdir -p /app/
ADD ./target/springboot-mybatis-demo-0.0.1-SNAPSHOT.jar /app/
CMD ["java", "-Xmx200m", "-jar", "/app/springboot-mybatis-demo-0.0.1-SNAPSHOT.jar"]

EXPOSE 18000
```
  docker build --rm -t springboot-mybatis-demo:base .

查找docker镜像
docker search 镜像关键字

docker 安装mysql
http://www.runoob.com/docker/docker-install-mysql.html
docker pull mysql

docker run -p 3306:3306 --name mymysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql

docker exec -it mymysql /bin/bash

mysql -uroot -p123456

从宿主机拷贝文件到容器
https://blog.csdn.net/dongdong9223/article/details/71425077
docker cp $PWD/springboot.sql mymysql:/home/
注意容器后没有空格

/home/springboot.sql

mysql运行命令行
http://database.51cto.com/art/201107/277687.htm
第一种方式：在未连接数据库的情况下，输入 mysql -h localhost -u root -p 123456  < /home/springboot.sql 回车即可；

第二种方式：在已连接数据库的情况下，此时命令提示符为mysql>，输入 source /home/springboot.sql。
