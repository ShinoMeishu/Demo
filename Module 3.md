# Module 3
## Task 1
```
apt install rsyslog
nano /etc/rsyslog.conf
```
Remove TCP UDP comments
```
systemctl restart rsyslog
ss -tunlp | grep 514
ps -A | grep rsyslog 
cat /var/log/syslog | grep rsyslog
```


## Task 3
```
echo "Authorized access only!" > /etc/custom_banner
nano /etc/ssh/sshd_config
```
```
Banner /etc/custom_banner
PermitRootLogin no
PasswordAuthentication no
Port 2222
MaxAuthTries 4
PermitEmptyPasswords no
LoginGraceTime 5m
```

## Task 6
```
apt install cups
systemctl start cups
```
В настройках линукса нужно зайти в принтеры и разрешить добавление принтеров, 
где уже добавить cups и можно будет его вывести в консоли
```
sudo lpstat -p 
```
## Task 8
```
touch /var/log/warning.log
touch /var/log/warning_memory.log
touch /var/log/warning_disk.log
nano /etc/rsyslog.conf
```
Добавить в конце файла:
```
if ($msg contains ‘CPU load average is above 70’) then /var/log/warning.log
if (($msg contains ‘warning’ and) and ($msg contains ‘memory usage is greater than or >
action(type=”omfile” file=”/var/log/warning_memory.log”)
}
if (($msg contains ‘warning’ and) and ($msg contains ‘disk usage is greater than or eq>
action(type=”omfile” file=”/var/log/warning_disk.log”)
}
```
```
tail -f /var/log/messages
```
