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
# local domen
127.0.1.1       iavclient
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
127.0.1.1       iavserver
# client domen
192.168.122.12  iavclient
```

```bash
sudo systemctl restart networking
ping iavclient
# 64 buytes from iavclient (192.168.122.12): icmp_seq=1 ttl=64 time=1.07 ms
```

## Задание 2
### 2.1. На сервере создайте локальный FTP-репозиторий и загрузите на него файл, содержащий в названии ваши ФИО.
Настройка серверной стороны:
```bash
# install ftp daemon
sudo apt install svftpd
```

```bash
sudo systemctl start vsftpd
sudo systemctl enable vsftpd
```

```bash
# open config file
sudo nano /etc/vsftpd.conf
```
Should be next params:
```
listen=YES
listen_ipv6=NO
anonymous_enable=YES

local_enable=YES
write_enable=YES
```
```bash
sudo systemctl restart vsftpd
```
```bash
echo "test file" > ~/ftp/iav.txt
sudo chown ftp:ftp /home/ftp/iav.txt
```

### 2.2. Выгрузите файл из созданного репозитория на машину Client.
```bash
sftp adminstd@iavserver # time out
```
```
> get ftp/iav.txt
```
### 2.3. Создайте на сервере общую папку smb и примонтируйте её на машине клиента в директорию с вашим именем.
Server:
```bash
sudo apt install samba
sudo mkdir /srv/share
sudo chown nobody:nogroup /srv/share
sudo chown 0777 /srv/share
sudo nano /etc/samba/smb.conf
sudo systemctl restart smbd
```
```
    ...
    map to guest = bad user
[share]
    comment = hello_everyone
    guest ok = yes
    force user = nobody
    force group = nogroup
    path = /srv/share
    read only = no
```
Client:
```bash
sudo apt install cifs-utils
sudo mount -t cifs //192.168.122.13/share /mnt -o users,sec=none
```

### 2.4. На Client, используя графический интерфейс, поменяйте дату и время на 01.01.1970 и 18:12. Синхронизируйте время Server и Client по сети, установив NTP-сервер на машину Server.
```bash
sudo apt install ntp
sudo cp /etc/ntp.conf /etc/ntp.conf.orig
sudo rm /etc/ntp.conf
sudo nano /etc/ntp.conf
```
```
server 192.128.122.13 prefer
```
```bash
sudo systemctl restart ntp
sudo systemctl status ntp
```

## Задание 3.
### 3.1. На сервере запустите службу ssh и добавьте её в автозагрузку.
```bash
sudo apt install sshd
sudo systemctl start sshd
sudo systemctl enable sshd
sudo systemctl status sshd
ssh-keygen
sudo ssh-copy-id adminstd@192.168.122.12
```
### 3.2. На клиенте настройте аутентификацию по ключам с сервером.
```bash
ssh key-gen
ssh-copy-id adminstd@192.168.122.13 # time out
```
### 3.3. Подключитесь к серверу с машины клиента и создайте в директории /home/study файл с содержимым «Hello world!».
```bash
echo "Hello, world!" ? /home/study/hello.txt
```
### 3.4. Скопируйте с сервера на клиент (командой scp) файл, созданный в предыдущем пункте.
```bash
# scp user@server_ip:path local_path
scp adminstd@192.168.13:/home/study/hello.txt /hello_received.txt
```