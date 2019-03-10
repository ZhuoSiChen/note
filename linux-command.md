firewall-cmd --zone=public --add-port=8040/tcp --permanent 
firewall-cmd --reload



#查看日志
docker-compose -f business-ms-compose.yml logs -t -f --tail='all' account

docker-compose -f basic-ms-compose.yml logs -t -f --tail='all' config_server

#启动服务
docker-compose -f business-ms-compose.yml up -d

docker-compose -f business-ms-compose.yml up -d

docker-compose -f monitor-ms-compose.yml  logs --tail='all' hystrix_dashboard

#停止服务
docker-compose -f infrastructure-compose.yml down
docker-compose -f monitor-ms-compose.yml down
docker-compose -f basic-ms-compose.yml down

#查看网络
docker network ls

docker network inspect   spring-cloud-rest-tcc_default

#进入容器
docker exec -it spring-cloud-rest-tcc_gateway_1  /bin/sh
docker exec -it hystrix_dashboard  /bin/bash



PS:
举个例子，假如一个应用程序在名为myapp的目录中，并且docker-compose.yml如下所示：

version: '2'
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
当我们运行docker-compose up时，将会执行以下几步：
创建一个名为myapp_default的网络；
使用web服务的配置创建容器，它以“web”这个名称加入网络myapp_default；
使用db服务的配置创建容器，它以“db”这个名称加入网络myapp_default。

#MYSQL
ALTER USER ‘root’@’localhost’ IDENTIFIED WITH mysql_native_password BY ‘mysql’; 


CREATE USER 'chris'@'%' ;

34.80.55.62
1045
update user set password=password("123123") where user="chris";
grant all on "mysql".* to 'chris'@'%';
flush privileges;
Alter user 'chris'@'%'  identified by '123123';

SHOW GRANTS FOR 'chris'@'%';