## Задание 1
### 1.1. Проанализируйте файл /etc/network/interfaces. Что означает каждая строка?
```bash
cat /etc/network/interfaces
```
### 1.2. Выясните текущие сетевые настройки каждой машины. Проверьте соединение между клиентом и сервером.
Для клиента:
```bash
cat /etc/hostname
# > iavclient

cat /etc/hosts
# > 127.0.0.1   localhost
# > 127.0.1.1   ru01std00v

ping iavserver
# 64 buytes from iavserver (192.168.122.230): icmp_seq=1 ttl=64 time=4.33 ms
# ...
```
Для сервера:
```bash
cat /etc/hostname
# > iavserver

cat /etc/hosts
# > 127.0.0.1   localhost
# > 127.0.1.1   ru01std00v

ping iavclient
# 64 buytes from iavclient (192.168.122.235): icmp_seq=1 ttl=64 time=3.97 ms
# ...
```
### 1.3. Используя параметры, приведённые в таблице ниже, настройте сеть, дополнив необходимые конфигурационные файлы.
|Machine name | Server | Client |
|---|---|---|
|Host name| iavserver| iavclient |
|IP address |192.168.122.12| 192.168.122.13|
|Default gateway |192.168.122.1 | 192.168.122.1 |


Конфигурация eth0 для клиента
```bash
sudo nano /etc/network/interfaces
```
```
auto eth0
iface eth0 inet static
address 192.168.122.13
netmask 255.255.255.0
gateway 192.168.122.1
```

Добавление домена сервера
```bash
sudo nano /etc/hosts
```

```
# server domen
192.168.122.13  iavserver
```

```bash
sudo systemctl restart networking
ping iavserver
# 64 buytes from iavserver (192.168.122.13): icmp_seq=1 ttl=64 time=4.33 ms
```


Конфигурация eth0 для сервера
```bash
sudo nano /etc/network/interfaces
```
```
auto eth0
iface eth0 inet static
address 192.168.122.12
netmask 255.255.255.0
gateway 192.168.122.1
```

Добавление домена клиента
```bash
sudo nano /etc/hosts
```

```
# client domen
192.168.122.12  iavclient
```

```bash
sudo systemctl restart networking
ping iavclient
# 64 buytes from iavclient (192.168.122.12): icmp_seq=1 ttl=64 time=1.07 ms
```