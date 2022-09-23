
## jenkins docker 

### 설치 & 실행
* docker pull jenkins/jenkins
* docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-server --restart=on-failure jenkins/jenkins:lts-jdk11
* docker logs jenkins-server

### 저장공간 지정(-v)
* docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-server --restart=on-failure -v jenkins_home:/var/jenkins_home/jenkins/jenkins:lts-jdk11

### password 확인
* docker exec -it jenkins-server bash
* cat /var/jenkins_home/secrets/initialAdminPassword
