### git & maven
* maven 설치
* global setting
* Jenkins item : My-second-project
  * https://github.com/joneconsulting/cicd-web-project
  

### Deploy plugin
* deploy to container plugin
* Jenkins item : My-third-project
* Post builder
  * Deploy war/ear to a container
  * WAR/EAR files : **/*.war
* Container - Tomcat 9.x Remote
  * Credentials : deploy 권한이 있는 tomcat 사용자 추가


### Tomcat
* 
* 

### poll scm
* 주기적으로 build, deploy


## docker deploy
* docker 이미지 설치
* docker run --privileged --name docker-server -itd -p 10022:22 -p 8081:8080 -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup edowon0623/docker:latest /usr/sbin/init
* 접속 테스트 : ssh root@localhost -p 10022
* password : P@ssw0rd
* 접속 IP 확인 : docker inspect DOCKER_ID
* ![image](https://user-images.githubusercontent.com/4444533/192170117-cd694683-38d4-4eac-95c2-ffd56cb1b18c.png)
* 서버정보
* ![image](https://user-images.githubusercontent.com/4444533/192170131-a0fdc48b-27f4-42ce-a414-a4501650feb4.png)


