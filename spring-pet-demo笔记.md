

```
mszarlinski/spring-petclinic-tracing-server
mszarlinski/spring-petclinic-api-gateway
mszarlinski/spring-petclinic-discovery-server
mszarlinski/spring-petclinic-config-server
mszarlinski/spring-petclinic-visits-service
mszarlinski/spring-petclinic-vets-service
mszarlinski/spring-petclinic-customers-service
mszarlinski/spring-petclinic-admin-server
```

```
docker tag mszarlinski/spring-petclinic-tracing-server regserv:5000/mszarlinski/spring-petclinic-tracing-server
docker tag mszarlinski/spring-petclinic-api-gateway regserv:5000/mszarlinski/spring-petclinic-api-gateway
docker tag mszarlinski/spring-petclinic-discovery-server regserv:5000/mszarlinski/spring-petclinic-discovery-server
docker tag mszarlinski/spring-petclinic-config-server regserv:5000/mszarlinski/spring-petclinic-config-server
docker tag mszarlinski/spring-petclinic-visits-service regserv:5000/mszarlinski/spring-petclinic-visits-service
docker tag mszarlinski/spring-petclinic-vets-service regserv:5000/mszarlinski/spring-petclinic-vets-service
docker tag mszarlinski/spring-petclinic-customers-service regserv:5000/mszarlinski/spring-petclinic-customers-service
docker tag mszarlinski/spring-petclinic-admin-server regserv:5000/mszarlinski/spring-petclinic-admin-server
```
```
docker push regserv:5000/mszarlinski/spring-petclinic-tracing-server
docker push regserv:5000/mszarlinski/spring-petclinic-api-gateway
docker push regserv:5000/mszarlinski/spring-petclinic-discovery-server
docker push regserv:5000/mszarlinski/spring-petclinic-config-server
docker push regserv:5000/mszarlinski/spring-petclinic-visits-service
docker push regserv:5000/mszarlinski/spring-petclinic-vets-service
docker push regserv:5000/mszarlinski/spring-petclinic-customers-service
docker push regserv:5000/mszarlinski/spring-petclinic-admin-server
```

push成功后, 可以调用registry API查看 registry中的镜像

```
curl regserv:5000/v2/_catalog
```

```
docker images -q
```

展示所有images的id

Swarm集群，部署单一服务:

```
docker service create  --replicas 1 --network  ingress  --name config-server -p 8888:8888 regserv:5000/mszarlinski/spring-petclinic-config-server

docker service create  --replicas 1 --network  ingress  --name config-server -p 8761:8761 regserv:5000/mszarlinski/spring-petclinic-discovery-server

docker service create  --replicas 1 --network  ingress  --name customers-service -p 8081:8081 regserv:5000/mszarlinski/spring-petclinic-customers-service

docker service create  --replicas 1 --network  ingress  --name visits-service -p 8082:8082 regserv:5000/mszarlinski/spring-petclinic-visits-service

docker service create  --replicas 1 --network  ingress  --name vets-service -p 8083:8083 regserv:5000/mszarlinski/spring-petclinic-discovery-server
```
