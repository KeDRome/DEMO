# ISP

## Настройка хостнеймов

```bash
hostnamectl set-hostname ISP
```

## [Настройка сети]

`nmtui` -- интерактивно \
`nmcli` -- командный интерфейс

> [1] Настройка интерйфесов:

```bash
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

```bash
## Forwarding 
iptables -A FORWARD -d 10.0.0.2 -s 10.0.0.1 -j ACCEPT
iptables -A FORWARD -d 10.0.0.1 -s 10.0.0.2 -j ACCEPT
```

Добавляем дополниельные роуты:

```bash
ip route add 10.0.0.2 dev ens4
ip route add 10.0.0.1 dev ens5
```

> [7] Настройка DNS службы

Установка `dnsmasq`:

```bash
apt install -yq dnsmasq
```

Редактирование конфига `/etc/dnsmasq.conf`:

```bash
listen-address=3.3.3.1,4.4.4.1,5.5.5.1
domain=demo.wsr
server=/int.demo.wsr/192.168.100.200

address=/isp.demo.wsr/3.3.3.1
address=/www.demo.wsr/4.4.4.100
address=/www.demo.wsr/5.5.5.100
```

Перезагрузка службы:

```bash
systemctl restart dnsmasq
systemctl enable dnsmasq
```

> [8] Настройка NTP Службы:

Установка службы

```bash
apt install ntp -yq
```

Настройка службы `/etc/ntp.conf`:

```bash
<Удалите всё что начинается с pool>
restict 192.168.100.0/24
restict 172.16.100.0/24

server 127.127.1.0
fudge 127.127.1.0 stratum 4
```
