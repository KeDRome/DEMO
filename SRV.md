# SRV

## Настройка хостнеймов

```bash
hostnamectl set-hostname SRV
```

## [Настройка сети]

`nmtui` -- интерактивно \
`nmcli` -- командный интерфейс

> [1] Настройка интерйфесов:

``` bash
--------------------------------------------------
| ens4 | 192.168.100.200/24 | 192.168.100.254/24 |
--------------------------------------------------
```

> [2] Настройка DNS:

Установка пакетов:

```bash
apt install -yq network-manager dnsutils dnsmasq 
```

Настройка DNS `/etc/dnsmasq.conf:

```bash
listen-address=192.168.100.200,127.0.0.1

domain=int.demo.wsr
server=/int.demo.wsr/127.0.0.1

address=/web-r.int.demo.wsr/192.168.100.100
address=/web-l.int.demo.wsr/172.16.100.100
address=/rtr-r.int.demo.wsr/192.168.100.254
address=/rtr-l.int.demo.wsr/172.16.100.254
address=/srv.int.demo.wsr/192.168.100.200

cname=ntp,srv
cname=dns,srv

host-record=web-r.int.demo.wsr,192.168.100.100
host-record=web-l.int.demo.wsr,172.16.100.100
host-record=rtr-r.int.demo.wsr,192.168.100.254
host-record=rtr-l.int.demo.wsr,172.16.100.254
host-record=srv.int.demo.wsr,192.168.100.200
```

Перезагрузка службы:

```bash
systemctl restart dnsmasq
systemctl enable dnsmasq
```
