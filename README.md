
## 인프런강의
https://www.inflearn.com/course/젠킨스-ci-cd-파이프라인/dashboard

docker file

https://github.com/joneconsulting/docker-files

## Section3. Jenkins + Infrastructure as Code 와의 연동

### jenkins-server
* 아래 docker를 사용해도 문제 없음
* docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-server --restart=on-failure jenkins/jenkins:lts-jdk11

### ansible-server 
* image download
* Docker 설정에서 WSL2를 사용하지 않게 수정하고, 아래 docker 파일을 이용해서 image를 생성하면 됨
![image](https://user-images.githubusercontent.com/4444533/196851489-995cab12-2337-4985-89f7-ad5347a6a297.png)

* ansible-server Dockerfile
* docker build -t ansible-server .
```
# FROM centos:7
FROM centos:8

RUN yum -y update; yum clean all
RUN yum -y install systemd; yum clean all; 
# (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
# rm -f /lib/systemd/system/multi-user.target.wants/*;\
# rm -f /etc/systemd/system/*.wants/*;\
# rm -f /lib/systemd/system/local-fs.target.wants/*; \
# rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
# rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
# rm -f /lib/systemd/system/basic.target.wants/*;\
# rm -f /lib/systemd/system/anaconda.target.wants/*;

VOLUME ["/sys/fs/cgroup"]

RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-Linux-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-Linux-*

# install sshd
RUN yum install -y epel-release
RUN yum install -y openssh-server \
&& sed 's/#PermitRootLogin yes/PermitRootLogin yes/' -i /etc/ssh/sshd_config \
&& echo 'root:P@ssw0rd' | chpasswd \
&& ssh-keygen -f /etc/ssh/ssh_host_rsa_key -N '' -t rsa
RUN yum install -y openssh-clients

RUN yum update -y
RUN yum install -y iptables
# RUN yum install -y docker docker-registry
# RUN yum install -y docker
# RUN yum install -y docker-ce-cli 
# RUN yum install -y containerd.io
# RUN sed -i -e 's/overlay2/vfs/g' /etc/sysconfig/docker-storage

RUN yum install -y yum-utils
RUN yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
RUN yum install -y --allowerasing docker-ce docker-ce-cli containerd.io

# RUN yum install -y --skip-broke ansible
RUN yum install -y python38
RUN pip3 install --upgrade --ignore-installed pip setuptools
RUN pip3 install ansible

WORKDIR /root
COPY ./Dockerfile ./Dockerfile

EXPOSE 22

ENTRYPOINT ["/sbin/init", "systemctl", "start", "sshd"]
# CMD ["systemctl", "start", "sshd"]
```

* docker run --privileged -itd --name ansible-server -p 20022:22 -p 8082:8080 -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup --cgroupns=host ansible-server /usr/sbin/init


### docker-server
* docker build -t docker-server .
```
# FROM arm64v8/centos:7
FROM centos:8


RUN yum -y update; yum clean all
RUN yum -y install systemd; yum clean all;
# (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
# rm -f /lib/systemd/system/multi-user.target.wants/*;\
# rm -f /etc/systemd/system/*.wants/*;\
# rm -f /lib/systemd/system/local-fs.target.wants/*; \
# rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
# rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
# rm -f /lib/systemd/system/basic.target.wants/*;\
# rm -f /lib/systemd/system/anaconda.target.wants/*;

VOLUME ["/sys/fs/cgroup"]

RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-Linux-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-Linux-*

# install sshd
RUN yum install -y epel-release
RUN yum install -y openssh-server \
&& sed 's/#PermitRootLogin yes/PermitRootLogin yes/' -i /etc/ssh/sshd_config \
&& echo 'root:P@ssw0rd' | chpasswd \
&& ssh-keygen -f /etc/ssh/ssh_host_rsa_key -N '' -t rsa
RUN yum install -y openssh-clients

RUN yum update -y
RUN yum install -y iptables
# RUN yum install -y docker docker-registry
# RUN yum install -y docker
# RUN yum install -y docker-ce-cli 
# RUN yum install -y containerd.io
# RUN sed -i -e 's/overlay2/vfs/g' /etc/sysconfig/docker-storage

RUN yum install -y yum-utils
RUN yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
RUN yum install -y --allowerasing docker-ce docker-ce-cli containerd.io
# RUN systemctl start docker
RUN yum install -y ncurses
RUN yum install -y git

WORKDIR /root
COPY ./Dockerfile ./Dockerfile

EXPOSE 22

ENTRYPOINT ["/sbin/init", "systemctl", "start", "sshd"]
```
* docker run --privileged --name docker-server -itd -p 10022:22 -p 8081:8080 -e container=docker-server -v /sys/fs/cgroup:/sys/fs/cgroup docker-server /usr/sbin/init


## 이하 모든 실습은 apple m1에서 진행함
* 현재 업무용 PC에서는 docker는 2개 이상 사용하면 ssh접속이 안되네...

### ansible server 환경설정
* /etc/ansible/hosts
* ansible-server:172.17.0.3
* docker-server:172.17.0.4
```
[devops]
172.17.0.3
172.17.0.4
```

### ansible & docker 구성
![image](https://user-images.githubusercontent.com/4444533/199354302-9e39db7a-e697-4e64-8848-69d9794c8230.png)


## ssh key
* ssh 접속시 비밀번호 없이 접속하기 위함
* ansible 서버에서 생성한다.
```
172.17.0.3# ssh-keygen
172.17.0.3# ssh-copy-id root@172.17.0.4 --> 원격서버에 복사
172.17.0.3# ssh-copy-id root@172.17.0.3 --> localhost에서 사용 가능하게 추가
```

## Ansible 모듈 사용
### ansible option
* -i 적용될 호스트파일 정보
* -m 모듈선택
* -k 관리자 암호
* -k 관리자 권한 상승
* --list-hosts 적용되는 호스트 목록
* [root@ansible ~]# ansible all -m ping
```
172.17.0.3 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
172.17.0.4 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```
* file copy
  * [root@ansible ~]# ansible all -m copy -a "src=./test.txt dest=/tmp"
* sw install
  * [root@ansible ~]# ansible devops -m yum -a "name=httpd state=present"

## ansible playbook
### first-playbook.yml
```
---
- name: Add an ansible hosts
  hosts: localhost
  tasks:
    - name: add an ansible hosts
      blockinfile:
        path: /etc/ansible/hosts
        block: |
          [mygroup]
          172.17.0.5
```

### playbook-sample2.yml
* tomcat 버전 68로 변경, checksum 주소변경 
```
---
- name: Download Tomcat9 from tomcat.apache.org
  hosts: devops
  tasks:
   - name: Create a Directory /opt/tomcat9
     file:
       path: /opt/tomcat9
       state: directory
       mode: 0755
   - name: Download Tomcat using get_url
     get_url:
       url: https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.68/bin/apache-tomcat-9.0.68.tar.gz
       dest: /opt/tomcat9
       mode: 0755
       checksum: sha512:https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.68/bin/apache-tomcat-9.0.68.tar.gz.sha512

```

### Jenkins + Ansible 연동하기
* ansible 접속 ssh 
<img width="842" alt="image" src="https://user-images.githubusercontent.com/4444533/199842602-bc634d75-f7b0-4c05-8a2b-a4a81ea33652.png">

* 모든환경에서 docker가 기본적으로 사용가능해야 함
* anslble deploy playbook
* https://github.com/joneconsulting/jenkins_cicd_script/blob/master/playbook_script/exam5-devops-playbook1.yml
```
- hosts: all
#   become: true  

  tasks:
  - name: build a docker image with deployed war file
    command: docker build -t cicd-project-ansible .
    args: 
        chdir: /root
```

* hosts 파일을 만들고
```
172.17.0.3
```
* ansible-playbook -i hosts first-devops-playbook.yml
* docker container 추가작업
```
- hosts: all
#   become: true  

  tasks:
  - name: build a docker image with deployed war file
    command: docker build -t cicd-project-ansible .
    args: 
      chdir: /root
  - name : create a container using cicd-project-ansible image
    command: docker run -d --name my-cicd-project -p 8080:8080 cicd-project-ansible
```
* docker ps -a 상태 확인

```
- hosts: all
#  become: true
  tasks:
    - name: stop current running container
      command: docker stop my-cicd-project
      ignore_errors: yes
    - name: remove stopped container
      command: docker rm my-cicd-project
      ignore_errors: yes
    - name: remove current docker image
      command: docker rmi cicd-project-ansible
      ignore_errors: yes

    - name: build a docker image with deployed war file
      command: docker build -t cicd-project-ansible .
      args:
        chdir: /root

    - name : create a container using cicd-project-ansible image
      command: docker run -d --name my-cicd-project -p 8080:8080 cicd-project-ansible
```

### Ansible을 이용한 Docker image 관리
```
[root@ansible ~]# docker images
REPOSITORY                     TAG       IMAGE ID       CREATED      SIZE
lanovia/cicd-project-ansible   latest    2de09ae1c475   2 days ago   478MB
cicd-project-ansible           latest    2de09ae1c475   2 days ago   478MB
<none>                         <none>    7cc16cc9ed55   2 days ago   478MB
<none>                         <none>    0d02e04f32c0   2 days ago   478MB
<none>                         <none>    ccd207ca38c6   2 days ago   478MB
tomcat                         9.0       da0ee5569ad8   4 days ago   470MB
[root@ansible ~]# docker tag cicd-project-ansible lanovia/cicd-project-ansible
[root@ansible ~]# docker login
Username: lanovia
Password:
```
* https://hub.docker.com/ 업로드 내용 확인
* Image 생성 후 hub 배포, hosts 파일에 3, 4 IP가 있어 3번에서만 image를 생성하는 조건으로 실행
```
create-cicd-devops-image.yml 

- hosts: all
#  become: true
  tasks:
    - name: create a docker image with deployed war file
      command: docker build -t lanovia/cicd-project-ansible .
      args:
        chdir: /root
    - name: push the image on Docker hub
      command: docker push lanovia/cicd-project-ansible
      ignore_errors: yes

    - name : remove the docker image from the ansible server
      command: docker rmi lanovia/cicd-project-ansible
      ignore_errors: yes      
```
* 현재 ansible에서만 image 생성
* ansible-playbook -i hosts create-cicd-devops-image.yml --limit 172.17.0.3

* Image를 만든 후 배포하는 playbook
```
create-cicd-devops-container.yml

- hosts: all
#  become: true
  tasks:
    - name: stop current running container
      command: docker stop my-cicd-project
      ignore_errors: yes
    - name: remove stopped container
      command: docker rm my-cicd-project
      ignore_errors: yes

    - name: remove current docker image
      command: docker rmi lanovia/cicd-project-ansible
      ignore_errors: yes

    - name: pull the newest docker image from docker hub
      command: docker pull lanovia/cicd-project-ansible
      args:
        chdir: /root

    - name : create a container using cicd-project-ansible image
      command: docker run -d --name my-cicd-project -p 8080:8080 lanovia/cicd-project-ansible
```
* 172.17.0.4 서버만 적용하는 조건
* [root@ansible ~]# ansible-playbook -i hosts create-cicd-devops-container.yml --limit 172.17.0.4
```
[WARNING]: An error occurred while calling ansible.utils.display.initialize_locale (unsupported
locale setting). This may result in incorrectly calculated text widths that can cause Display to
print incorrect line lengths

PLAY [all] ****************************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [172.17.0.4]

TASK [stop current running container] *************************************************************
changed: [172.17.0.4]

TASK [remove stopped container] *******************************************************************
changed: [172.17.0.4]

TASK [remove current docker image] ****************************************************************
changed: [172.17.0.4]

TASK [pull the newest docker image from docker hub] ***********************************************
changed: [172.17.0.4]

TASK [create a container using cicd-project-ansible image] ****************************************
changed: [172.17.0.4]

PLAY RECAP ****************************************************************************************
172.17.0.4                 : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Ansible playbook으로 Docker container 생성하기
* jenkins new item : My-playbook-project를 My-ansible-pproject에서 복사
<img width="543" alt="image" src="https://user-images.githubusercontent.com/4444533/200217979-77410146-958a-4065-adfd-ca3bcb68dd1a.png">

* jenkins run


## Section 4. Jenkins + Ansible + Kubernetes 연동
### docker k8s 사용
* kubectl get nodes
```
 ~/I/S/je/k8s  kubectl get nodes 
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   10h   v1.25.2
```

### Kubernetes 기본 명령어
* sample1.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
* kubectl get deployments
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/2     2            0           11s
</pre>
```
* kubectl get pods
```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7fb96c846b-449mp   1/1     Running   0          30s
nginx-deployment-7fb96c846b-vzp8w   1/1     Running   0          30s
```

### kubernetes script 파일
* cicd-devops-deployment.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cicd-deployment
spec:
  selector:
    matchLabels:
      app: cicd-devops-project
  replicas: 2

  template:
    metadata:
      labels:
        app: cicd-devops-project
    spec:
      containers:
      - name: cicd-devops-project
        image: lanovia/cicd-project-ansible
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
```

* cicd-devops-service.yml
```
apiVersion: v1
kind: Service
metadata:
  name: cicd-service
  labels:
    app: cicd-devops-project
spec:
  selector:
    app: cicd-devops-project
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32000
```

* kubectl apply -f cicd-devops-deployment.yml
* kubectl apply -f cicd-devops-service.yml
* kubectl get deployment
* kubectl get services
* kubectl get pods -o wide
* kubectl get services
```
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
cicd-service   NodePort    10.107.203.83   <none>        8080:32000/TCP   16s
```
* http://localhost:32000/hello-world/ 정상화면이 나오는지 확인


### Kubernetes + Ansible 연동

### Ansible에서 Kubernetes 제어하기
* ansible docker
* k8s 디렉토리를 만들고 hosts 파일을 생성한다
```
[root@ansible ~]# cat k8s/hosts
[ansible-server]
localhost

[kubernetes]
150.74.201.75
````
* k8s 서버와 ssh 연결을 확인한다.
* [root@ansible ~]# ansible -i k8s/hosts kubernetes -m ping
```
[root@ansible ~]# ansible -i k8s/hosts kubernetes -m ping
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
150.74.201.75 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: root@150.74.201.75: Permission denied (publickey,password,keyboard-interactive).",
    "unreachable": true
}
```
* k8s로 ansible ssh key를 복사해야 한다.
````
 [root@ansible ~]# ssh-copy-id parknamjun@150.74.201.75
 ~/IdeaProjects/Study/jenkins  cat ~/.ssh/authorized_keys                                   ✔  07:48:38
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC/gPFNhp+Y6WiHzrm/J+gy7M0UgkTvPSGxREOP0IwRPdiDjPKBHYaoE29DsAmt8wUnpGd1E6+62TDFPhtoaBo3byafvCV4BFn/4o0i54TxMDJuJxXYK9XqpAHY5WqAerFbOdwqWoUIE6HVqmEtIwM2oNIVFoya42eY3cP1ttHAcRw3rihuDq0IiirACjukWaEahIqfRW9q3YnJEu8gW15B0+lsZENtCibjZJE1/J9Iz3nx2jqAKFBPoR6+9w2EDReuZ4DMeclKo6vGoSNy/kb6b+CG5V4zBto4DNj36TPLEHDCTn2qaJsqiZwn8sd8D5mG4ZCNJTLknw9uChDdLBjLHSRfP5OXOQjrQKAjpJ3HhRHy2rbUyGx251YAKyChZqjAFZqeMulWce0u6hBDJph73L1sPzjhdvBUnr+T2ZEcGm1FKI5/9LsHLoZgNGtEow1Wzl2I3N9I4nACn97YIdAkeeCu+U/7S6tLqPHlvGpTXucKaSG8c3o9LD2q6Wn0mVE= root@93cdc1d35c55
````
* root 계정으로 복사되므로 원격에서 접속하려면 Mac 사용자 계정을 이용해야 한다.
```
[root@ansible ~]# ansible -i k8s/hosts kubernetes -m ping -u parknamjun
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
[WARNING]: Platform darwin on host 150.74.201.75 is using the discovered Python interpreter at
/usr/bin/python3, but future installation of another Python interpreter could change the meaning of that
path. See https://docs.ansible.com/ansible-core/2.13/reference_appendices/interpreter_discovery.html for
more information.
150.74.201.75 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```


### 실습8) Jenkins를 이용한 CI/CD 자동화 파이프라인 구축하기 ①
* ansible docker에서 확인
```
[root@ansible ~]# ls -l create*
-rw-r--r-- 1 root root 683 Nov  6 22:32 create-cicd-devops-container.yml
-rw-r--r-- 1 root root 457 Nov  6 22:42 create-cicd-devops-image.yml
```
* Dockerfile
```
FROM tomcat:9.0

LABEL org.opencontainers.image.authors="eahn.park@gmail.com"

COPY ./hello-world.war /usr/local/tomcat/webapps
```

* k8s 서버 
  *  ~/I/S/jenkins  kubectl get all
```
 ~/I/S/jenkins  kubectl get all                                          ✔  docker-desktop ⎈  07:24:57
NAME                                   READY   STATUS    RESTARTS   AGE
pod/cicd-deployment-5f96747f9c-htq5t   1/1     Running   0          19h
pod/cicd-deployment-5f96747f9c-w9nwp   1/1     Running   0          19h

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/cicd-service   NodePort    10.107.203.83   <none>        8080:32000/TCP   19h
service/kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP          2d11h

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cicd-deployment   2/2     2            2           19h

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/cicd-deployment-5f96747f9c   2         2         2       19h
```

 * 필요없는 deployment, service 삭제
 *  ~/I/S/jenkins  kubectl delete deployment.apps/cicd-deployment service/cicd-service

* ansible애서 k8s로 ssh 접속을 할 수 있어야 함, Mac은 설정에서 원격로그인이 가능하게 설정하면 ssh로 원격에서 접속가능하다.
* 

