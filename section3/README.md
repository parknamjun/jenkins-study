## jenkins + Infrastructure as Code

* Infrastructure as Code 개요와 Ansible의 이행
* Docker 컨테이너로 Ansible 실행하기
* Ansible 설정과 작동 과정
* Ansible 명령어
* Ansible 모듈 사용
* Ansible Playbook 사용하기
* Jenkins + Ansible 연동하기
*  Ansible을 이용한 Docker 이미지 관리


### image download
* window 노트북에서 안되는 이유는 ?? virtual box 
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

* docker run --privileged -itd --name ansible-server -p 20022:22 -p 8081:8080 -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup --cgroupns=host ansible-server /usr/sbin/init



