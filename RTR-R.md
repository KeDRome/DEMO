# RTR-R
## Настройка хостнеймов
```bash
hostnamectl set-hostname RTR-R
```
## [Настройка сети]

`nmtui` -- интерактивно \
`nmcli` -- командный интерфейс

> [1] Настройка интерйфесов:
```
-------------------------------------------
| ens5   | 5.5.5.100/24      | 5.5.5.1/24 |
-------------------------------------------
| ens172 | 172.16.100.254/24 | ---------- |
-------------------------------------------
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
## 5.5.5.0/24 to 172.16.100.0/24 ...
iptables -A FORWARD -s 5.5.5.0/24 -d 172.16.100.0/24 -j ACCEPT
## 172.16.100.0/24 to 5.5.5.0/24 
iptables -A FORWARD -d 5.5.5.0/24 -s 172.16.100.0/24 -j ACCEPT
iptables -t nat -A POSTROUTING -s 172.16.100.0/24 -o ens5 -j SNAT --to-source 5.5.5.100

# Block Network
```

> [4] Настраиваем GRE

Материал взят [отсюда](https://wiki.astralinux.ru/pages/viewpage.action?pageId=144310560#id-НастройкатоннеляGREвAstraLinux-НастройкатоннеляGREcпомощьюграфическогоплагинаNetworkManagernm-connection-editor)
```bash
# Добавляем соединение ...
nmcli con add type ip-tunnel ip-tunnel.mode gre con-name gre1 ifname gre1 \
remote 10.0.0.1 local 10.0.0.2
# Добавляем статический адрес
nmcli con mod gre1 ipv4.addresses '10.0.0.2/30'
nmcli con mod gre1 ipv4.method manual
# Добавление статического маршрута в сеть ЗА тунелем
nmcli con mod gre1 +ipv4.routes "192.168.100.0/24 10.0.0.1"
# Поднимаем тунель
sudo nmcli con up gre1
```
> Лучше производить изменения в роутинге через nmtui, так как в примере ниже, настройки после ребута сбросятся.

И добавляем дополнительный роут который поведет наш трафик через NAT:
```bash
ip route add to 10.0.0.1 via 5.5.5.1 dev ens5
```