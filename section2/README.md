### git & maven
* maven 설치
* global setting에서 maven 버전을 선택
* Jenkins item : My-second-project
  * https://github.com/joneconsulting/cicd-web-project
  * Build/goals and options :clean compile package
  * build
  

### Deploy plugin
* "deploy to container" plugin
* Jenkins item : My-third-project
* Post builder(빌드 후 조치)
  * "Deploy war/ear to a container" 선택
  * WAR/EAR files : **/*.war
* Container - Tomcat 9.x Remote
  * Credentials : deploy 권한이 있는 tomcat 사용자 추가
  * tomcat-user.xml에 "username="deployer" password="deployer" roles="manager-script"를 추가
  * "Add" jenkins 사용자를 선택 후 deployer 계정, 비밀번호, ID를 입력
  * Credential combo에서 추가한 deployer를 선택한다.
  * Tomcat URL : http://HOSTPC-IP:8088, docker-jenkins에서 HOST PC로 접속하여 manager-script 권한을 가진 사용자로 deploy 함 
  * server.xml 8080을 8088로 변경
  * webapps/manager/META-INF/context.xml, webapps/host-manager/META-INF/contex.xml
    * 2개 파일 아래 부분을 주석으로 처리한다. 관리자 권한은 127 대역에서만 접속하게 하는 제약 사항임
    * Valve className="org.apache.catalina.valves.RemoteAddrValve"
    *  allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1"


## docker deploy
* windows docker 이미지 설치
* docker run --privileged --name docker-server -itd -p 10022:22 -p 8081:8080 -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup edowon0623/docker:latest /usr/sbin/init
* MacOS apple silicon chip, m1) SSH 서버 (with 도커) 실행 명령어
* docker run --privileged --name docker-server -itd -p 10022:22 -p 8081:8080 -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup:rw --cgroupns=host edowon0623/docker-server:m1 /usr/sbin/init
* 접속 테스트 : ssh root@localhost -p 10022
* password : P@ssw0rd
* 기존에 ssh 정보가 있는 경우 에러가 발생할 수 있으므로 .ssh/known_hosts 파일에서 서버정보를 삭제한다.
* 접속 IP 확인 : docker inspect DOCKER_ID
* ![image](https://user-images.githubusercontent.com/4444533/192170117-cd694683-38d4-4eac-95c2-ffd56cb1b18c.png)
* 서버정보
* ![image](https://user-images.githubusercontent.com/4444533/192170131-a0fdc48b-27f4-42ce-a414-a4501650feb4.png)
* docker-server에 접속하여 docker 상태를 확인한다. 아직 실행중인 컨테이너는 없는 상태
```
C:\Users\USER>ssh root@localhost -p 10022
root@localhost's password:

Docker version 20.10.18, build b40c2f6
[root@7dd971023287 ~]# docker images
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
[root@7dd971023287 ~]# systemctl status docker --docker 서비스 상태확인
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://docs.docker.com
[root@7dd971023287 ~]# systemctl start docker

docker 시작시 iptable 등의 오류가 있어 다음장으로 넘어감

```

## apple m1으로 대체해서 진행
  * Dockerfile
    * FROM tomcat:9.0
    * COPY ./hello-world.war /usr/local/tomcat/webapps
<img width="793" alt="image" src="https://user-images.githubusercontent.com/4444533/193154559-bfb6867c-8925-4e4e-89b6-f4da047049c5.png">
<img width="793" alt="image" src="https://user-images.githubusercontent.com/4444533/193154598-ea177479-764e-4aed-a11f-2951a0b91cf4.png">


## 사내 네트워크 사용하면 오류가 있으므로 wifi guest를 사용해야 함


## docker login 정보확인
```
C:\Users\USER\.docker>type config.json
{
        "auths": {
                "https://index.docker.io/v1/": {}
        },
        "credsStore": "desktop"
}

C:\Users\USER\.docker>docker-credential-wincred.exe list
{"https://index.docker.io/v1/":"USER_ACCOUNT"}
```




