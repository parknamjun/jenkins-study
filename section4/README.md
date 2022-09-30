## Jenkins + Infrastructure as Code 와의 연동

### jenkins 
 docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-server --restart=on-failure jenkins/jenkins:lts-jdk11
 
### docker 
* -p 8081:8080 삭제, tomcat deploy test 끝
 docker run --privileged --name docker-server -itd -p 10022:22 -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup:rw --cgroupns=host edowon0623/docker-server:m1 /usr/sbin/init
 
### ansible
 docker run --privileged --name ansible-server -itd -p 20022:22 -p 8081:8080 -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup --cgroupns=host edowon0623/ansible-server:m1 /usr/sbin/init
 
### network 설정
* docker network inspect bridge
* 
