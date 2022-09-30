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
* docker run --privileged -itd --name ansible-server -p 20022:22 -p 8081:8080 -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup --cgroupns=host edowon0623/ansible:latest /usr/sbin/init

```
dockerd -D
NFO[2022-09-29T23:36:09.156939290Z] stopping event stream following graceful shutdown  error="<nil>" module=libcontainerd namespace=moby
DEBU[2022-09-29T23:36:09.157007530Z] Cleaning up old mountid : done.
failed to start daemon: Error initializing network controller: error obtaining controller instance: failed to create NAT chain DOCKER: iptables failed: iptables --wait -t nat -N DOCKER: iptables v1.8.4 (nf_tables):  CHAIN_ADD failed (No such file or directory): chain PREROUTING
 (exit status 4)
```
* MacOS silicon chip, m1) docker pull edowon0623/ansible-server:m1
* docker run --privileged --name ansible-server -itd -p 20022:22 -p 8081:8080 -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup --cgroupns=host edowon0623/ansible-server:m1 /usr/sbin/init
* systemctl start docker 정상확인...

