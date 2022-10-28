### local pc에서 사용

* 보유하고 있는 라이선스가 아래 7.13.3 버전이라...
  * docker -m=2g run -v /C/docker-volume/confluence:/var/atlassian/application-data/confluence --name="confluence" -d -p 8090:8090 -p 8091:8091 atlassian/confluence:7.13.3-jdk11
  * cp mysql-connector-java-5.1.49.jar /opt/atlassian/confluence/confluence/WEB-INF/lib
  * docker restart confluence
  * CREATE DATABASE <database-name> CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
