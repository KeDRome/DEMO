# ISP
## Настройка хостнеймов
```bash
hostnamectl set-hostname ISP
```
## [Настройка сети]

`nmtui` -- интерактивно \
`nmcli` -- командный интерфейс

> [1] Настройка интерйфесов:
```
---------------------
| ens5 | 5.5.5.1/24 |
---------------------
| ens3 | 3.3.3.1/24 |
---------------------
| ens4 | 4.4.4.1/24 |
---------------------
```

> [2] Включаем форвардинг пакетов между интерфейсами:
```bash
sysctl net.ipv4.ip_forward=1
sysctl -p
```

> [3] Настраиваем маршрутизацию через iptables:
```bash
iptables -F
# Allow Network ..
## 5.5.5.0/24 to 3.3.3.0/24 ...
iptables -A FORWARD -s 5.5.5.0/24 -d 3.3.3.0/24 -j ACCEPT
## 3.3.3.0/24 to 5.5.5.0/24
iptables -A FORWARD -d 5.5.5.0/24 -s 3.3.3.0/24 -j ACCEPT
## 5.5.5.0/24 to 4.4.4.0/24
iptables -A FORWARD -s 5.5.5.0/24 -d 4.4.4.0/24 -j ACCEPT
## 4.4.4.0/24 to 5.5.5.0/24
iptables -A FORWARD -d 5.5.5.0/24 -s 4.4.4.0/24 -j ACCEPT
## 4.4.4.0/24 to 3.3.3.0/24 
iptables -A FORWARD -s 4.4.4.0/24 -d 3.3.3.0/24 -j ACCEPT
## 3.3.3.0/24 to 4.4.4.0/24
iptables -A FORWARD -s 3.3.3.0/24 -d 4.4.4.0/24 -j ACCEPT

# Block Network ..
## 192.168.100.0/24 to external networks
iptables -A FORWARD -s 192.168.100.0/24 -d 3.3.3.0/24 -j DROP
iptables -A FORWARD -s 192.168.100.0/24 -d 4.4.4.0/24 -j DROP
iptables -A FORWARD -s 192.168.100.0/24 -d 5.5.5.0/24 -j DROP
iptables -A FORWARD -d 192.168.100.0/24 -s 3.3.3.0/24 -j DROP
iptables -A FORWARD -d 192.168.100.0/24 -s 4.4.4.0/24 -j DROP
iptables -A FORWARD -d 192.168.100.0/24 -s 5.5.5.0/24 -j DROP
## 172.16.100.0/24 to external networks
iptables -A FORWARD -s 172.16.100.0/24 -d 3.3.3.0/24 -j DROP
iptables -A FORWARD -s 172.16.100.0/24 -d 4.4.4.0/24 -j DROP
iptables -A FORWARD -s 172.16.100.0/24 -d 5.5.5.0/24 -j DROP
iptables -A FORWARD -d 172.16.100.0/24 -s 3.3.3.0/24 -j DROP
iptables -A FORWARD -d 172.16.100.0/24 -s 4.4.4.0/24 -j DROP
iptables -A FORWARD -d 172.16.100.0/24 -s 5.5.5.0/24 -j DROP
##  172.16.100.0/24 to 192.168.100.0/24
iptables -A FORWARD -s 172.16.100.0/24 -d 192.168.100.0/24 -j DROP
##  192.168.100.0/24 to 172.16.100.0/24
iptables -A FORWARD -d 172.16.100.0/24 -s 192.168.100.0/24 -j DROP

```

> [4] Настраиваем GRE

Дополняем правила iptables, которые позволяют форвардить трафик с внешних интерфейсов RTR, до их GRE-тунельных интерфейсов. 
```
...

## Forwarding 
iptables -A FORWARD -d 10.0.0.2 -s 10.0.0.1 -j ACCEPT
iptables -A FORWARD -d 10.0.0.1 -s 10.0.0.2 -j ACCEPT

...
```
Добавляем дополниельные роуты:
```
ip route add 10.0.0.2 dev ens4
ip route add 10.0.0.1 dev ens5
```