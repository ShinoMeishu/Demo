# Module 1
### Task 1.

### Part A. Setting hostnames (All machines)

```
hostnamectl set-hostname <hostname>
```

### Part C-D. Netmasks

x = Количество хостов, x = 2^y <br />
255.255.255.256-x = netmask <br />
/32-y

### Part B. Setting IPs (All machines)

| Имя устройства | IPv4        | IPv6           |
|----------------|-------------|----------------|
| CLI            | 10.0.0.2/24 | 2001:10::2/124 |
| ISP            | DHCP        | DHCP           |
|                | 10.0.0.1/24 | 2001:10::1/124 |
|                | 1.0.0.1/30  | 2000:1::1/124  |
|                | 2.0.0.1/30  | 2000:2::1/124  |
| HQ-R           | 1.0.0.2/30  | 2000:1::2/124  |
|                | 20.0.0.1/26 | 2001:20::1/124 |
| HQ-SRV         | 20.0.0.2/26 | 2001:20::2/124 |
| BR-R           | 2.0.0.2/30  | 2000:2::2/124  |
|                | 30.0.0.1/28 | 2001:30::1/124 |
| BR-SRV         | 30.0.0.2/28 | 2001:30::2/124 |
| HQ-CLI         | 20.0.0.3/26 | 2001:20::3/124 |
| HQ-AD          | 20.0.0.4/26 | 2001:20::4/124 |


```
nano /etc/network/interfaces
```
```
auto <interface>
iface <interface> inet static/dhcp
address 0.0.0.0/32
gateway 0.0.0.0

iface enp0s3 inet6 static
address 2001:3::2 
netmask 124
gateway 2001:3::1 
```
### Task 2. (Routers)
```
apt install frr -y
nano /etc/frr/daemons
```
> ospfd=yes
```
nano /etc/frr/frr.conf
```
```
> log syslog informational <br />
>router ospf <br />
> ospf router-id 10.10.10.1 <br />
> redistribute connected <br />
> network 10.10.10.0/28 area 0.0.0.0 <br />
> network 1.1.1.0/30 area 0.0.0.0 <br />
> network 2.2.2.0/30 area 0.0.0.0 <br />
```
```
> systemctl restart frr
> vtysh -c "show run"
> vtysh -c "show ip ospf neighbor"
```
ISP:
```
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQURADE
apt install iptables-persistent -y
```
### Part A. Unknown

### Task 3. (HQ-R)
```
apt install isc-dhcp-server -y
nano /etc/default/isc-dhcp-server 
```
```
INTERFACESv4="enp0s8"
```
```
nano /etc/dhcp/dhcpd.conf
```
```
default-lease-time 600; 
max-lease-time 7200;
option domain-name "hq.work";
option domain-name-servers 20.20.20.2, 8.8.8.8;

subnet 20.20.20.0 netmask 255.255.255.192 {
 range 20.20.20.2 20.20.20.18;
 option routers 20.20.20.1;
}

host hqserver {
hardware ethernet <MAC-адресс HQ-SRV>;
fixed-address 20.20.20.2;
}
```
```
systemctl restart isc-dhcp-server
```
На HQ-SRV поставить интерфейс на режим dhcp

### Task 4. (All machines)
```
adduser <name>
getent passwd
```
Чтобы сделать судо-юзера
```
usermod –aG sudo <name> 
```

### Task 5. 
ISP:
```
apt install iperf3 -y
iperf3 -s
```
HQ-R:
```
apt install iperf3 -y
iperf3 -c 1.1.1.1
```


### Task 6. (HQ-R, BR-R)

```
cd /home
mkdir backup
cd /backup
nano backup_router.sh
```
```
#!/bin/bash

cp -r /etc/frr /home/backup/frr_backup
cp -r /etc/network /home/backup/network_backup
cp -r /etc/iptables /home/backup/iptables_backup
# Для HQ-R
cp -r /etc/dhcp /home/backup/dhcp_backup

echo "backup complete"
exit 0
```
```
ls
bash backup_router.sh
ls
```



### Task 7-8. (HQ-SRV)
```
apt install ssh
nano /etc/ssh/sshd_config
```
> Port 2222
```
iptables -t nat -A PREROUTING -p tcp --dport 22 -j NAT --to-destination 20.20.20.2:2222
iptables -I INPUT -s 10.0.0.2 -p tcp --dport 2222 -j REJECT
apt install iptables-persistent -y
```








