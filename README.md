[TOC]
# saasi-demo
# 实验步骤:

dony的虚拟机ip：

192.168.109.128

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
  


  
## 查看docker log
sudo docker logs -f -t --tail 行数 容器名
  
## linux批量查找文件内容
https://blog.csdn.net/ilove737/article/details/48372281
  
  
-----------------------------------
## 重命名docker容器名
docker rename old容器名  new容器名


## 清空集群

参考链接: http://dockone.io/question/1076

kubectl delete services --all

kubectl delete pods --all

kubectl delete deployments --all

注意必须删除所有的deployments才可以防止重新生成

无论各种方式生成的pod, 均可以使用如下命令强制删除: kubectl delete pods <pod> --grace-period=0 --force 

## 状态查看
kubectl get nodes 查看你集群的机器的状态

kubectl get pods --all-namespaces 查看你集群中pod状态

kuebctl get svc --all-namespaces 查看集群中的服务的状态

## docker 链接spring boot和mysql容器

第一种方式:

1.修改application.yml:把localhost改为mysql

2.通过--link把两个容器链接起来，

参考链接：

https://blog.csdn.net/happyyear1/article/details/72314147

docker run -d -p 8080:8080 --name spring-demo --link mymysql:mysql spring-demo

第二种方式:

添加 --net=host

参考链接: 

https://blog.csdn.net/begin1013/article/details/80860224

采用host的网络方式将容器与宿主机共用一个Network Namespace,这样容器内localhost就是宿主机的localhost了


注意，此处应该加 --net = host，因为docker会给启动的容器自动配置ip，但是项目里配置的mysql地址是localhost，在容器内连接localhost是连接容器本身而不是宿主机的localhost，所以一直找不到mysql。所以采用host的网络方式将容器与宿主机共用一个Network Namespace,这样容器内localhost就是宿主机的localhost了。


docker run -p --net=host 8080:8080 --name springboot-mybatis-demo -d springboot-mybatis-demo:base

总结:

两种都是可行的，推荐使用第一种，因为后面服务很多的话，第一种更安全。

## 虚拟机更新yum源为阿里镜像

link：

https://blog.csdn.net/inslow/article/details/54177191

mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

cd /etc/yum.repos.d/

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

yum makecache

yum -y update

## 虚拟机安装k8s集群
### 关闭防火墙

systemctl stop firewalld

systemctl disable firewalld
### 禁用SELINUX

#### 临时禁用
setenforce 0

#### 永久禁用 
vim /etc/selinux/config    # 或者修改/etc/sysconfig/selinux

SELINUX=disabled

### 修改k8s.conf文件

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
    
### 关闭swap

# 临时关闭
swapoff -a

修改 /etc/fstab 文件，注释掉 SWAP 的自动挂载（永久关闭swap，重启后生效）

# 注释掉以下字段
/dev/mapper/cl-swap     swap                    swap    defaults        0 0

安装Docker
卸载老版本的Docker

如果有没有老版本Docker，则不需要这步

```
yum remove docker \
           docker-common \
           docker-selinux \
           docker-engine
```

使用yum进行安装

每个节点均要安装，目前官网建议安装17.03版本的docker，官网链接

# step 1: 安装必要的一些系统工具

sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# Step 2: 添加软件源信息

sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# Step 3: 更新并安装 Docker-CE

sudo yum makecache fast

sudo yum -y install docker-ce docker-ce-selinux

```
# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
#
# 安装指定版本的Docker-CE:
# Step 3.1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
```
# Step 3.2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)

sudo yum -y --setopt=obsoletes=0 install docker-ce-[VERSION] \
docker-ce-selinux-[VERSION]

sudo yum -y --setopt=obsoletes=0 install docker-ce-17.03.0.ce-1.el7.centos \
docker-ce-selinux-17.03.0.ce-1.el7.centos

# Step 4: 开启Docker服务

sudo systemctl enable docker && systemctl start docker

docker安装问题小结

错误信息

```
Error: Package: docker-ce-17.03.2.ce-1.el7.centos.x86_64 (docker-ce-stable)
           Requires: docker-ce-selinux >= 17.03.2.ce-1.el7.centos
           Available: docker-ce-selinux-17.03.0.ce-1.el7.centos.noarch (docker-ce-stable)
               docker-ce-selinux = 17.03.0.ce-1.el7.centos
           Available: docker-ce-selinux-17.03.1.ce-1.el7.centos.noarch (docker-ce-stable)
               docker-ce-selinux = 17.03.1.ce-1.el7.centos
           Available: docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch (docker-ce-stable)
               docker-ce-selinux = 17.03.2.ce-1.el7.centos
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```

## 解决办法

要先安装docker-ce-selinux-17.03.2.ce，否则安装docker-ce会报错
注意docker-ce-selinux的版本 要与docker的版本一致

yum install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm

或者

yum -y install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm

# Docker 配置国内镜像源加速

cat << EOF >> /etc/docker/daemon.json
{
        "registry-mirrors": ["https://registry.docker-cn.com"]
}
EOF
