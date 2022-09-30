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

<pre>
----------------------------------------
iptables-config
# Load additional iptables modules (nat helpers)
#   Default: -none-
# Space separated list of nat helpers (e.g. 'ip_nat_ftp ip_nat_irc'), which
# are loaded after the firewall rules are applied. Options for the helpers are
# stored in /etc/modprobe.conf.
IPTABLES_MODULES=""

# Save current firewall rules on stop.
#   Value: yes|no,  default: no
# Saves all firewall rules to /etc/sysconfig/iptables if firewall gets stopped
# (e.g. on system shutdown).
IPTABLES_SAVE_ON_STOP="no"

# Save current firewall rules on restart.
#   Value: yes|no,  default: no
# Saves all firewall rules to /etc/sysconfig/iptables if firewall gets
# restarted.
IPTABLES_SAVE_ON_RESTART="no"

# Save (and restore) rule and chain counter.
#   Value: yes|no,  default: no
# Save counters for rules and chains to /etc/sysconfig/iptables if
# 'service iptables save' is called or on stop or restart if SAVE_ON_STOP or
# SAVE_ON_RESTART is enabled.
IPTABLES_SAVE_COUNTER="no"

# Numeric status output
#   Value: yes|no,  default: yes
# Print IP addresses and port numbers in numeric format in the status output.
IPTABLES_STATUS_NUMERIC="yes"

# Verbose status output
#   Value: yes|no,  default: yes
# Print info about the number of packets and bytes plus the "input-" and
# "outputdevice" in the status output.
IPTABLES_STATUS_VERBOSE="no"

# Status output with numbered lines
#   Value: yes|no,  default: yes
# Print a counter/number for every rule in the status output.
IPTABLES_STATUS_LINENUMBERS="yes"

# Reload sysctl settings on start and restart
#   Default: -none-
# Space separated list of sysctl items which are to be reloaded on start.
# List items will be matched by fgrep.
#IPTABLES_SYSCTL_LOAD_LIST=".nf_conntrack .bridge-nf"

# Set wait option for iptables-restore calls in seconds
#   Default: 600
# Set to 0 to deactivate the wait.
#IPTABLES_RESTORE_WAIT=600

# Set wait interval option for iptables-restore calls in microseconds
#   Default: 1000000
# Set to 100000 to try to get the lock every 100000 microseconds, 10 times a
# second.
# Only usable with IPTABLES_RESTORE_WAIT > 0
#IPTABLES_RESTORE_WAIT_INTERVAL=1000000

----------------------------------------
ip6tables-confif
# Load additional ip6tables modules (nat helpers)
#   Default: -none-
# Space separated list of nat helpers (e.g. 'ip_nat_ftp ip_nat_irc'), which
# are loaded after the firewall rules are applied. Options for the helpers are
# stored in /etc/modprobe.conf.
IP6TABLES_MODULES=""

# Save current firewall rules on stop.
#   Value: yes|no,  default: no
# Saves all firewall rules to /etc/sysconfig/ip6tables if firewall gets stopped
# (e.g. on system shutdown).
IP6TABLES_SAVE_ON_STOP="no"

# Save current firewall rules on restart.
#   Value: yes|no,  default: no
# Saves all firewall rules to /etc/sysconfig/ip6tables if firewall gets
# restarted.
IP6TABLES_SAVE_ON_RESTART="no"

# Save (and restore) rule and chain counter.
#   Value: yes|no,  default: no
# Save counters for rules and chains to /etc/sysconfig/ip6tables if
# 'service ip6tables save' is called or on stop or restart if SAVE_ON_STOP or
# SAVE_ON_RESTART is enabled.
IP6TABLES_SAVE_COUNTER="no"

# Numeric status output
#   Value: yes|no,  default: yes
# Print IP addresses and port numbers in numeric format in the status output.
IP6TABLES_STATUS_NUMERIC="yes"

# Verbose status output
#   Value: yes|no,  default: yes
# Print info about the number of packets and bytes plus the "input-" and
# "outputdevice" in the status output.
IP6TABLES_STATUS_VERBOSE="no"

# Status output with numbered lines
#   Value: yes|no,  default: yes
# Print a counter/number for every rule in the status output.
IP6TABLES_STATUS_LINENUMBERS="yes"

# Reload sysctl settings on start and restart
#   Default: -none-
# Space separated list of sysctl items which are to be reloaded on start.
# List items will be matched by fgrep.
#IP6TABLES_SYSCTL_LOAD_LIST=".nf_conntrack .bridge-nf"

# Set wait option for ip6tables-restore calls in seconds
#   Default: 600
# Set to 0 to deactivate the wait.
#IP6TABLES_RESTORE_WAIT=600

# Set wait interval option for ip6tables-restore calls in microseconds
#   Default: 1000000
# Set to 100000 to try to get the lock every 100000 microseconds, 10 times a
# second.
# Only usable with IP6TABLES_RESTORE_WAIT > 0
#IP6TABLES_RESTORE_WAIT_INTERVAL=1000000

</pre>

