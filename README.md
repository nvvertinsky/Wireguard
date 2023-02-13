# Wireguard

## Текстовая инструкция по настройке Wireguard к видео (https://www.youtube.com/watch?v=5Aql0V-ta8A).

### Команды:
```
apt update && apt upgrade -y                                                          # Обновляем сервер
apt install -y wireguard                                                              # Ставим wireguard
wg genkey | tee /etc/wireguard/privatekey | wg pubkey | tee /etc/wireguard/publickey  # Генерим ключи сервера
chmod 600 /etc/wireguard/privatekey                                                   # Проставляем права на приватный ключ
ip a                                                                                  # Проверим, как у вас называется сетевой интерфейс
```

### Полученный интефейс (eth0, ens3) инспользуем в /etc/wireguard/wg0.conf, который мы сейчас создадим:

```
nano /etc/wireguard/wg0.conf

[Interface]
PrivateKey = <privatekey>
Address = 10.0.0.1/24
ListenPort = 51830
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

### Настраиваем IP форвардинг:
```
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

### Включаем systemd демон с wireguard:
```
systemctl enable wg-quick@wg0.service
systemctl start wg-quick@wg0.service
systemctl status wg-quick@wg0.service
```

### Создаём ключи клиента:
```
wg genkey | tee /etc/wireguard/goloburdin_privatekey | wg pubkey | tee /etc/wireguard/goloburdin_publickey
```

### Добавляем в конфиг сервера клиента:
```
nano /etc/wireguard/wg0.conf

[Peer]
PublicKey = <nvvertinsky_publickey>
AllowedIPs = 10.0.0.2/32
```


### Перезагружаем systemd сервис с wireguard:
```
systemctl restart wg-quick@wg0
systemctl status wg-quick@wg0
```

### На локальной машине создаём текстовый файл с конфигом клиента:
```
nano nvvertinsky_wb.conf

[Interface]
PrivateKey = <CLIENT-PRIVATE-KEY>
Address = 10.0.0.2/32
DNS = 8.8.8.8

[Peer]
PublicKey = <SERVER-PUBKEY>
Endpoint = <SERVER-IP>:51830
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 20
```

### Этот файл открываем в Wireguard клиенте — и жмем в клиенте кнопку подключения.