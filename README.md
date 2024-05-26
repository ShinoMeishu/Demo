### Module 1
## Task 1.

# Part A. Setting hostnames (All machines)

> hostnamectl set-hostname <hostname>

# Part C-D. Netmasks

x = Количество хостов, x = 2^y
255.255.255.256-x = netmask
/32-y

# Part B. Setting IPs (All machines)

1.0.0.0 - ISP-HQ
2.0.0.0 - ISP-BR
10.0.0.0 - ISP-CLI
20.0.0.0 - HQ
30.0.0.0 - BRANCH

> nano /etc/network/interfaces

auto <interface>
iface <interface> inet static/dhcp
address 0.0.0.0/32
gateway 0.0.0.0 



## Task 2. (Routers)

R> apt install frr -y
R> nano /etc/frr/daemons

ospfd=yes

R> nano /etc/frr/frr.conf

log syslog informational
router ospf
 ospf router-id 10.10.10.1
 redistribute connected
 network 10.10.10.0/28 area 0.0.0.0
 network 1.1.1.0/30 area 0.0.0.0
 network 2.2.2.0/30 area 0.0.0.0

R> systemctl restart frr
R> vtysh -c "show run"
R> vtysh -c "show ip ospf neighbor"

ISP> iptables -t nat -A POSTROUTING -o enp0s3 -j MASQURADE
ISP> apt install iptables-persistent -y

# Part A. Unknown



## Task 3. (HQ-R)

> apt install isc-dhcp-server -y
> nano /etc/default/isc-dhcp-server 

INTERFACESv4="<ИНТЕРФЕЙС ПОДСЕТИ HQ>"

> nano /etc/dhcp/dhcpd.conf

default-lease-time 600;
max-lease-time 7200;
option domain-name "hq.work";
option domain-name-servers 20.20.20.2, 8.8.8.8;

subnet 20.20.20.0 netmask 255.255.255.192 {
 range 20.20.20.2 20.20.20.18;
 option routers 20.20.20.1;
}

host hqserver {
 hardware ethernet MAC-адресс HQ-SRV;
 fixed-address 20.20.20.2;
}

> systemctl restart isc-dhcp-server

# На HQ-SRV поставить интерфейс на режим dhcp



## Task 4. (All machines)

> adduser <name>
> getent passwd

# Чтобы сделать судо-юзера
> usermod –aG sudo <name> 



## Task 5. 

ISP> apt install iperf3 -y
ISP> iperf3 -s

HQ-R> apt install iperf3 -y
HQ-R> iperf3 -c 1.1.1.1



## Task 6. (HQ-R, BR-R)

> cd /home
> mkdir backup
> cd /backup
> nano backup_router.sh

#!/bin/bash

cp -r /etc/frr /home/backup/frr_backup
cp -r /etc/network /home/backup/network_backup
cp -r /etc/iptables /home/backup/iptables_backup
# Для HQ-R
cp -r /etc/dhcp /home/backup/dhcp_backup

echo "backup complete"
exit 0

> ls
> bash backup_router.sh
> ls



## Task 7-8. (HQ-SRV)

> apt install ssh
> nano /etc/ssh/sshd_config

Port 2222

> iptables -t nat -A PREROUTING -p tcp --dport 22 -j NAT --to-destination 20.20.20.2:2222
> iptables -I INPUT -s 10.0.0.2 -p tcp --dport 2222 -j REJECT
> apt install iptables-persistent -y









