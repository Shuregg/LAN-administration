# Local net, Samba, FTP, SSH

## 1. Initial machine configuration

### 1.1 Interfaces configuration

```bash
cat /etc/network/interfaces
```

### 1.2. Net settings

For Client:

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

For Server:

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

### 1.3. Net configuration

|Machine name | Server | Client |
|---|---|---|
|Host name| iavserver| iavclient |
|IP address |192.168.122.12| 192.168.122.13|
|Default gateway |192.168.122.1 | 192.168.122.1 |

Client's eth0 interface configuration

```bash
sudo nano /etc/network/interfaces
```

```bash
auto eth0
iface eth0 inet static
address 192.168.122.12
netmask 255.255.255.0
gateway 192.168.122.1
```

Adding server's domain

```bash
sudo nano /etc/hosts
```

```bash
# local domain
127.0.1.1       iavclient
# server domain
192.168.122.13  iavserver
```

```bash
sudo systemctl restart networking
ping iavserver
# 64 buytes from iavserver (192.168.122.13): icmp_seq=1 ttl=64 time=4.33 ms
```

Server's eth0 interface configuration

```bash
sudo nano /etc/network/interfaces
```

```bash
auto eth0
iface eth0 inet static
address 192.168.122.13
netmask 255.255.255.0
gateway 192.168.122.1
```

Adding client's domain

```bash
sudo nano /etc/hosts
```

```bash
127.0.1.1       iavserver
# client domain
192.168.122.12  iavclient
```

```bash
sudo systemctl restart networking
ping iavclient
# 64 buytes from iavclient (192.168.122.12): icmp_seq=1 ttl=64 time=1.07 ms
```

## 2. Shared space (FTP & Samba)

### 2.1. Create local FTP repo on Server and upload a file

Server's side config:

```bash
# install ftp daemon
sudo apt install vsftpd
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

```bash
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
sudo usermod -d /home/astra/ftp/ ftp
```

### 2.2. Download file from Server's FTP repo to Client

```bash
wget ftp://ftp@192.168.122.13/iav.txt
```

### 2.3. Create

Server:

```bash
sudo apt install samba
sudo mkdir /srv/share
sudo chown 777 /srv/share
sudo nano /etc/samba/smb.conf
sudo systemctl restart smbd
```

```bash
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
sudo mkdir ~/mnt
sudo mount -t cifs //192.168.122.13/share /home/adminstd/mnt -o users,sec=none
ls ~/mnt
# > iav.txt
```

### 2.4. Change date and time on client to 01.01.1970 18:12. Sync date and time with Server's date and time using NTP-server

```bash
# Client's machine
sudo apt install ntp
sudo cp /etc/ntp.conf /etc/ntp.conf.orig
sudo rm /etc/ntp.conf
sudo nano /etc/ntp.conf
```

```bash
server 192.128.122.13 prefer
```

```bash
sudo systemctl restart ntp
sudo systemctl status ntp
```

## 3. SSH

### 3.1. Run and add to autorun ssh daemon on Server

```bash
sudo apt install ssh
sudo systemctl start sshd
sudo systemctl enable sshd
sudo systemctl status sshd
```

### 3.2. Set authentication to Server via keys

```bash
ssh key-gen
ssh-copy-id adminstd@192.168.122.13
```

### 3.3. Connect to Server and make a dir

```bash
echo "Hello, world!" > /home/study/hello.txt
```

### 3.4. Copy a file from Server to Client this `hello.txt` file

```bash
# scp user@server_ip:path local_path
scp adminstd@192.168.13:/home/study/hello.txt /hello_received.txt
```
