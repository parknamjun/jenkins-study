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
* docker pull edowon0623/ansible:latest
* MacOS silicon chip, m1) docker pull edowon0623/ansible-server:m1
* docker run --privileged -itd -p 20022:22 -p 8081:8080 -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup edowon0623/ansible:latest /usr/sbin/init

section2에서 발생한 systemctl start docker 오류가 계속됨....
