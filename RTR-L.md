# RTR-L
## Настройка хостнеймов
```bash
hostnamectl set-hostname RTR-L
```
## [Настройка сети]

`nmtui` -- интерактивно \
`nmcli` -- командный интерфейс

> [1] Настройка интерйфесов:
```
--------------------------------------------
| ens4   | 4.4.4.100/24       | 4.4.4.1/24 |
-------------------------------------------
| ens192 | 192.168.100.254/24 | ---------- |
--------------------------------------------
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
## 4.4.4.0/24 to 192.168.100.0/24 ...
iptables -A FORWARD -s 4.4.4.0/24 -d 192.168.100.0/24 -j ACCEPT
## 192.168.100.0/24 to 4.4.4.0/24 
iptables -A FORWARD -d 4.4.4.0/24 -s 192.168.100.0/24 -j ACCEPT
iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o ens4 -j SNAT --to-source 4.4.4.100

# Block Network
```

> [4] Настраиваем GRE

Материал взят [отсюда](https://wiki.astralinux.ru/pages/viewpage.action?pageId=144310560#id-НастройкатоннеляGREвAstraLinux-НастройкатоннеляGREcпомощьюграфическогоплагинаNetworkManagernm-connection-editor)
```bash
# Добавляем соединение ...
nmcli con add type ip-tunnel ip-tunnel.mode gre con-name gre1 ifname gre1 \
remote 10.0.0.2 local 10.0.0.1
# Добавляем статический адрес
nmcli con mod gre1 ipv4.addresses '10.0.0.1/30'
nmcli con mod gre1 ipv4.method manual
# Добавление статического маршрута в сеть ЗА тунелем
nmcli con mod gre1 +ipv4.routes "172.16.100.0/24 10.0.0.2"
# Поднимаем тунель
sudo nmcli con up gre1
```
> Лучше производить изменения в роутинге через nmtui, так как в примере ниже, настройки после ребута сбросятся.

И добавляем дополнительный роут который поведет наш трафик через NAT:
```bash
ip route add to 10.0.0.2 via 4.4.4.1 dev ens4
```

>[5] Настройка фаервола на RTR-L

Разрешить: 
- __DNS__ - 53 
- __HTTP__ - 80 
- __HTTPS__ - 443
- __ICMP__
- __SSH__ - 22

Блокировать: __всё остальное__
```
# Allow
iptables -t filter -A INPUT  ! -s 192.168.100.0/24 -p tcp --dport 22 -j ACCEPT
iptables -t filter -A INPUT  ! -s 192.168.100.0/24 -p tcp --dport 443 -j ACCEPT
iptables -t filter -A INPUT  ! -s 192.168.100.0/24 -p tcp --dport 80 -j ACCEPT
iptables -t filter -A INPUT  ! -s 192.168.100.0/24 -p udp --dport 53 -j ACCEPT
iptables -t filter -A INPUT  ! -s 192.168.100.0/24 -p icmp -j ACCEPT

# Block
iptables -t filter -A INPUT  ! -s 192.168.100.0/24 -j DROP
```

> [6] Проброс порта SSH
```
iptables -t nat -A PREROUTING -i ens5 -p tcp --dport 2222 -d 4.4.4.100 -j DNAT --to 192.168.100.100:22
iptables -t nat -A POSTROUTING -o ens172 -j SNAT --to 192.168.100.254
```

> [7] Проброс порта DNS
```
iptables -t nat -A PREROUTING -i ens5 -p udp --dport 53 -d 4.4.4.100 -j DNAT --to 192.168.100.200:53
```