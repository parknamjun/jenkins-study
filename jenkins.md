### jenkins install
* docker pull jenkins/jenkins
* docker run -d --name jenkins-server --net mynetwork --ip 172.20.0.5 -p 9080:8080 -p 50000:50000 -v /D/docker-volume/jenkins:/var/jenkins_home jenkins/jenkins

### SonarCube
* docker run -d -p 9000:9000 --name sonarqube sonarqube
* 초기계정 : admin/admin
