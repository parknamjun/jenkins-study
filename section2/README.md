### git & maven
* maven 설치
* global setting
* Jenkins item : My-second-project
  * https://github.com/joneconsulting/cicd-web-project
  * Build/goals and options :clean compile package
  * build
  

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




