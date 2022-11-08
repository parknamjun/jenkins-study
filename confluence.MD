### local pc에서 사용

* 공유 볼륨 : c:\docker-volume
* 네트워크 생성
  * docker network create --subnet=172.20.0.0/16 mynetwork

* https://hub.docker.com/r/atlassian/confluence-server
* 보유하고 있는 라이선스가 아래 7.13.3 버전이라...
  * docker run -m=2g -v /C/docker-volume/confluence:/var/atlassian/application-data/confluence --net mynetwork --ip 172.20.0.4 --name="confluence" -d -p 8090:8090 -p 8091:8091 atlassian/confluence:7.13.3-jdk11
  * cp mysql-connector-java-5.1.49.jar /opt/atlassian/confluence/confluence/WEB-INF/lib
  * docker restart confluence
  * CREATE DATABASE confluence CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
* DB연결 설정
  * jdbc:mysql://172.17.0.3/confluence?useSSL=false&allowPublicKeyRetrieval=true&sessionVariables=transaction_isolation='READ-COMMITTED'
  * 
* Mysql volume을 사용
* docker run --net mynetwork --ip 172.20.0.3 --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -v /C/docker-volume/mysql:/var/lib/mysql -d mysql

 메모리를 얼마로 해야 하나...
 
