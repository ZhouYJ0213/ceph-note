## 时间同步
```
yum install -y chrony
systemctl start chronyd.service
systemctl restart chronyd.service
systemctl enable chronyd.service
#当服务器的话
allow 192.168/16

#开启防火墙
firewall-cmd --permanent --add-service=ntp
firewall-cmd --reload

#手动
chronyd -q 'server 0.asia.pool.ntp.org iburst'
chronyc sourcestats
chronyc sources -v
chronyc tracking


通过tcp同步时间
yum install -y rdate
proxychains rdate -s time-b.nist.gov

```
```
chronyc sources -v
date +%Y%m%d -s "20160616"
date +%T -s "8:55:30"
date -s "2016-06-21 16:42:00"
timedatectl set-timezone Asia/Shanghai
```

//设置ntp服务器本地
```
timedatectl set-timezone Asia/Shanghai

vi /etc/ntp.conf
restrict default nomodify
server  127.127.1.0     # local clock
fudge   127.127.1.0 stratum 8


watch -n1 ntpq -p  #watch util poll reach 77 above

//使用ntpdate客户端
server 192.168.10.5 iburst minpoll 4 maxpoll 4
```

## 查看硬盘使用量
```
df -lP | awk '{total+=$3} END {printf "%d G\n", total/2^20 + 0.5}'
for i in `seq 1 5`;do ssh ceph0$i df -lP | awk '{total+=$3} END {printf "%d G\n", total/2^20 + 0.5}';done
```

## 防火墙设置
```
systemctl disable firewalld
systemctl stop firewalld
sed -i  s'/SELINUX.*=.*enforcing/SELINUX=disabled'/g /etc/selinux/config
sed -i  s'/SELINUX.*=.*permissive/SELINUX=disabled'/g /etc/selinux/config

sed -i  s'/SELINUX.*=.*enforcing/SELINUX=disabled'/g /etc/sysconfig/selinux
sed -i  s'/SELINUX.*=.*permissive/SELINUX=disabled'/g /etc/sysconfig/selinux
sed -i  s'/Defaults    requiretty/#Defaults    requiretty'/g /etc/sudoers
setenforce 0


firewall-cmd --list-all
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload


firewall-cmd --list-all
firewall-cmd --zone=public --remove-port=8080/tcp --permanent
firewall-cmd --reload

#foward ip
#keepalived 防火墙
firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 --in-interface bond0 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
firewall-cmd --direct --permanent --add-rule ipv4 filter OUTPUT 0 --out-interface bond0 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
firewall-cmd --reload 


firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 --in-interface bond1 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
firewall-cmd --direct --permanent --add-rule ipv4 filter OUTPUT 0 --out-interface bond1 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
firewall-cmd --reload 

firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 --in-interface bond2 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
firewall-cmd --direct --permanent --add-rule ipv4 filter OUTPUT 0 --out-interface bond2 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
firewall-cmd --reload 

```

## 格式化磁盘
```
systemctl restart systemd-udevd.service
parted -a optimal --script /dev/sdk mklabel gpt
for sdd in `echo b c d e`;do parted -a optimal --script /dev/sd$sdd mklabel gpt ;done
parted -a optimal --script /dev/sdb mklabel gpt
```
## 添加用户
```
useradd -d /home/cephuser -m cephuser
passwd cephuser
echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
chmod 0440 /etc/sudoers.d/cephuser
sed -i 's/Defaults    requiretty/#Defaults    requiretty/g' /etc/sudoers
```
## 永久路由
```
[root@yuliyang ~]# cat /etc/sysconfig/network-scripts/route-ens33
10.139.11.0/24 via 192.168.100.2 dev ens33
```
### 清理无效的systemctl   sudo systemctl status ceph-radosgw@* 就不会显示了
```
sudo systemctl reset-failed
```


