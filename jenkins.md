### jenkins install
* docker pull jenkins:2.60.3 latest가 안되므로 버전을 지정해야 함
* docker run --name jenkins-server -p 9080:8080 -p 50000:50000 -v /D/docker-volume/jenkins:/var/jenkins_home jenkins:2.60.3