

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

docker swarmd的情况:

参考链接:

https://www.kancloud.cn/docker_practice/docker_practice/469847

```
version: '3'
services:


  config-server:
    image: mszarlinski/spring-petclinic-config-server
    ports:
     - 8888:8888
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    networks:
     - dockercoins    

  discovery-server:
    image: mszarlinski/spring-petclinic-discovery-server
    links:
     - config-server
    depends_on:
      - config-server
    entrypoint: ["./wait-for-it.sh","config-server:8888","--timeout=60","--","java", "-XX:+UnlockExperimentalVMOptions", "-XX:+UseCGroupMemoryLimitForHeap", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
     - 8761:8761
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    networks:
     - dockercoins    

  customers-service:
    image: mszarlinski/spring-petclinic-customers-service
    links:
     - config-server
     - discovery-server
     - tracing-server
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./wait-for-it.sh","discovery-server:8761","--timeout=60","--","java", "-XX:+UnlockExperimentalVMOptions", "-XX:+UseCGroupMemoryLimitForHeap", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
    - 8081:8081
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    networks:
     - dockercoins    

  visits-service:
    image: mszarlinski/spring-petclinic-visits-service
    links:
     - config-server
     - discovery-server
     - tracing-server
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./wait-for-it.sh","discovery-server:8761","--timeout=60","--","java", "-XX:+UnlockExperimentalVMOptions", "-XX:+UseCGroupMemoryLimitForHeap", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
     - 8082:8082
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    networks:
     - dockercoins    

  vets-service:
    image: mszarlinski/spring-petclinic-vets-service
    links:
     - config-server
     - discovery-server
     - tracing-server
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./wait-for-it.sh","discovery-server:8761","--timeout=60","--","java", "-XX:+UnlockExperimentalVMOptions", "-XX:+UseCGroupMemoryLimitForHeap", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
     - 8083:8083
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    networks:
     - dockercoins    
    networks:
     - dockercoins    

  api-gateway:
    image: mszarlinski/spring-petclinic-api-gateway
    links:
     - config-server
     - discovery-server
     - customers-service
     - visits-service
     - vets-service
     - tracing-server
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./wait-for-it.sh","discovery-server:8761","--timeout=60","--","java", "-XX:+UnlockExperimentalVMOptions", "-XX:+UseCGroupMemoryLimitForHeap", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
     - 8080:8080
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    networks:
     - dockercoins    

  tracing-server:
    image: mszarlinski/spring-petclinic-tracing-server
    links:
     - config-server
     - discovery-server
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./wait-for-it.sh","discovery-server:8761","--timeout=60","--","java", "-XX:+UnlockExperimentalVMOptions", "-XX:+UseCGroupMemoryLimitForHeap", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
     - 9411:9411
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    networks:
     - dockercoins    

  admin-server:
    image: mszarlinski/spring-petclinic-admin-server
    links:
     - config-server
     - discovery-server
    depends_on:
     - config-server
     - discovery-server
    entrypoint: ["./wait-for-it.sh","discovery-server:8761","--timeout=60","--","java", "-XX:+UnlockExperimentalVMOptions", "-XX:+UseCGroupMemoryLimitForHeap", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    ports:
     - 9090:9090
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    networks:
     - dockercoins

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8999:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]   

networks:
  dockercoins: 

```

docker stats 看内存，cpu等情况
docker stack logs -f stack名 看stack
