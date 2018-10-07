
[TOC]

# 启动程序

## 镜像名

rabbitmq:3-management

spring-boot-cloud/registry

spring-boot-cloud/config

spring-boot-cloud/monitor

spring-boot-cloud/zipkin

spring-boot-cloud/gateway

spring-boot-cloud/auth-service

spring-boot-cloud/svca-service

spring-boot-cloud/svcb-service

## 拉取脚本

cd ..
```
docker build --rm -t spring-boot-cloud/registry spring-boot-cloud/registry
docker build --rm -t spring-boot-cloud/config spring-boot-cloud/config
docker build --rm -t spring-boot-cloud/monitor spring-boot-cloud/monitor
docker build --rm -t spring-boot-cloud/zipkin spring-boot-cloud/zipkin
docker build --rm -t spring-boot-cloud/gateway spring-boot-cloud/gateway
docker build --rm -t spring-boot-cloud/auth-service spring-boot-cloud/auth-service
docker build --rm -t spring-boot-cloud/svca-service spring-boot-cloud/svca-service
docker build --rm -t spring-boot-cloud/svcb-service spring-boot-cloud/svcb-service
```
