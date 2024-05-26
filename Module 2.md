# Module 2
### Task 1. HQ-SRV
```
apt install bind9 bind9-utils -y
nano /etc/bind/named.conf.options
```
```
acl internal-network {
	20.0.0.0/26;
	30.0.0.0/28;
};
	
options {

	directory "/var/cache/bind";
	allow-query { localhost; internal-network; branch; };
	allow-transfer { localhost; };
	recursion yes;
	dnssec-validation auto;
	listen-on { any; };
	listen-on-v6 { any; };
};
```
```
nano /etc/bind/named.conf.local
```
```
zone "hq.work" IN {
	type master;
	file "/etc/bind/hq.lan";
	allow-update { none; };
};
zone "0.0.20.in-addr.arpa" IN {
	type master;
	file "/etc/bind/20.db";
	allow-update { none; };
};
zone "branch.work" IN {
	type master;
	file "/etc/bind/branch.lan";
	allow-update { none; };
};
zone "0.0.30.in-addr.arpa" IN {
	type master;
	file "/etc/bind/30.db";
	allow-update { none; };
};
```
```
cp db.empty hq.lan
cp db.empty branch.lan
cp db.empty 20.db
cp db.empty 30.db
```

Добавить строки в файлы
```
nano /etc/bind/hq.lan
```
```
hq-srv  IN      A       20.0.0.2
hq-r    IN      A       20.0.0.1
```
```
nano /etc/bind/branch.lan
```
```
br-srv  IN      A       30.0.0.2
br-r    IN      A       30.0.0.1
```
```
nano /etc/bind/20.db
```
```
2       IN      PTR     hq-srv.hq.work.
1       IN      PTR     hq-r.hq.work.
```
```
nano /etc/bind/30.db
```
```
2       IN      PTR     br-srv.branch.work.
1       IN      PTR     br-r.branch.work.
```
```
systemctl restart bind9
```
All machines:
```
nano /etc/resolv.conf
```
```
domain hq.work (or branch.work)
search hq.work (or branch.work)
nameserver 20.0.0.2
nameserver 8.8.8.8
```
HQ-R: nslookup hq-srv <br />
HQ-SRV: nslookup br-r <br />
BR-R: nslookup 30.0.0.2 <br />
BR-SRV: nslookup 20.0.0.1 <br />


