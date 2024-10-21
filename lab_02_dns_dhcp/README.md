
# DNS, DHCP

## Setting up DNS with BIND9

### Step 1: Installing BIND9 on both virtual machines

install BIND9 on both the server and client machines:

```bash
sudo apt install bind9
```

### Step 2: Configuring BIND9 on the server

#### 1. Defining the DNS zone

1. Open the BIND9 configuration file:

   ```bash
   sudo nano /etc/bind/named.conf.local
   ```

2. Add the zone configuration for the domain `iav.miet.stu`:

   ```bash
   zone "iav.miet.stu" {
       type master;
       file "/etc/bind/zones/db.iav.miet.stu";
   };
   ```

#### 2. Reverse zone for the 192.168.122 subnet

1. In the same file, add the reverse zone:

   ```bash
   zone "122.168.192.in-addr.arpa" {
       type master;
       file "/etc/bind/zones/db.192.168.122";
   };
   ```

#### 3. Verifying the configuration

After making the changes, verify the configuration:

```bash
sudo named-checkconf
```

#### 4. Creating zone files

1. Create a directory for the zone files:

   ```bash
   sudo mkdir /etc/bind/zones
   ```

2. Create the resource record file for the zone `iav.miet.stu`:

   ```bash
   sudo nano /etc/bind/zones/db.iav.miet.stu
   ```

   Fill the file with the following content:

   ```plaintext
   $TTL    604800
   iav.miet.stu.        IN       SOA     srv.iav.miet.stu. admin.iav.miet.stu. (
                             2024100601   ; Serial number
                             3h           ; Reresh rate
                             1h           ; Retry of refresh 
                             1w           ; Expire
                             1h           ; Negative Cache TTL
   )           
   iav.miet.stu.        IN       NS      srv.iav.miet.stu.
   srv                  IN       A       192.168.122.13
   cli                  IN       A       192.168.122.12
   cli2                 IN       A       192.168.122.14
   ```

3. Create the reverse zone file:

   ```bash
   sudo nano /etc/bind/zones/db.192.168.122
   ```

   Fill the file with the following content:

   ```plaintext
   TTL 604800
   122.168.192.in-addr.arpa.       IN      SOA srv.iav.miet.stu. admin.iav.miet.stu. (
           2024100601 ;
           3h ;
           1h ;
           1w ;
           1h ;
   )
   122.168.192.in-addr.arpa.       IN      NS  srv.iav.miet.stu.
   12                              IN      PTR cli.iav.miet.stu.
   13                              IN      PTR srv.iav.miet.stu.
   14                              IN      PTR cli2.iav.miet.stu.
   ```

#### 5. Checking the zone files

Check the zone files for errors:

```bash
sudo named-checkzone iav.miet.stu /etc/bind/zones/db.iav.miet.stu
sudo named-checkzone 122.168.192.in-addr.arpa /etc/bind/zones/db.192.168.122
```

#### 6. Restarting BIND9 and testing

```bash
sudo nano /etc/resolv.conf
```

```plaintext
domain iav.miet.stu
nameserver 192.168.122.13
nameserver 10.0.2.3
```

Restart the service:

```bash
sudo systemctl restart bind9
```

Test the availability of the machines by pinging their names:

```bash
ping srv.iav.miet.stu
```

```plaintext
PING srv.iav.miet.stu (192.168.122.13) 56(84) bytes of data.
64 bytes from srv.iav.miet.stu (192.168.122.13): icmp_seq=1 ttl=64 time=0.022 ms
64 bytes from srv.iav.miet.stu (192.168.122.13): icmp_seq=2 ttl=64 time=0.086 ms
64 bytes from srv.iav.miet.stu (192.168.122.13): icmp_seq=3 ttl=64 time=0.069 ms
64 bytes from srv.iav.miet.stu (192.168.122.13): icmp_seq=4 ttl=64 time=0.082 ms
```

```bash
ping cli.iav.miet.stu
```

```plaintext
PING cli.iav.miet.stu (192.168.122.12) 56(84) bytes of data.
64 bytes from cli.iav.miet.stu (192.168.122.12): icmp_seq=1 ttl=64 time=0.896 ms
64 bytes from cli.iav.miet.stu (192.168.122.12): icmp_seq=2 ttl=64 time=1.11 ms
64 bytes from cli.iav.miet.stu (192.168.122.12): icmp_seq=3 ttl=64 time=1.03 ms
64 bytes from cli.iav.miet.stu (192.168.122.12): icmp_seq=4 ttl=64 time=1.73 ms
```

```bash
ping cli2.iav.miet.stu
```

```plaintext
ping: cli2.iav.miet.stu: Unknown name or service
```
or

```plaintext
PING cli2.iav.miet.stu (192.168.122.14) 56(84) bytes of data.
From srv.iav.miet.stu (192.168.122.13) icmp_seq=1 Destination Host Unreachable
From srv.iav.miet.stu (192.168.122.13) icmp_seq=2 Destination Host Unreachable
From srv.iav.miet.stu (192.168.122.13) icmp_seq=3 Destination Host Unreachable
```

### Step 3: Configuring the client to send DNS queries by domain names

1. Open the `/etc/resolv.conf` file:

   ```bash
   sudo nano /etc/resolv.conf
   ```

2. Add the DNS server:

   ```plaintext
   domain iav.miet.stu
   nameserver 192.168.122.13
   ```

Now you can send DNS queries by domain names, for example:

```bash
ping srv.iav.miet.stu
ping cli.iav.miet.stu
```

## DHCP

### Step 1:  Install the DHCP server on the server machine

install fly-admin-dhcp on the server machine (for Astra Linux OS):

```bash
sudo apt-get install fly-admin-dhcp
```

### Step 2: Select the range 192.168.122.(N) - 192.168.122.(N+20) to allocate dynamic addresses

Set dhcp interface
```bash
sudo nano /etc/default/isc-dhcp-server
```

```plaintext
INTERFACESv4="eth0"
INTERFACESv6=""
```
 
```bash
sudo nano /etc/dhcp/dhcpd.conf
```

```plaintext
option domain-name "iavdhcp";
option domain-name-servers iavdhcp;

subnet 192.168.122.0 netmask 255.255.255.0 {
   range 192.168.122.91 192.168.122.111;
}
```

### Step 3: Start the DHCP service and make sure it is working correctly

```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```

Attention! The eth0 of Server machine should be static like this:

```plaintext
auto eth0
iface eth0 inet static
address 192.168.122.13
netmask 255.255.255.0
gateway 192.168.122.1
```

but, of course, the client's interface should be controled by dhcp:

```plaintext
auto eth0
iface eth0 inet dhcp
```

On the client machine:
1. Reset dynamic address
2. Get new address
3. Check it

```bash
sudo dhclient -r
sudo dhclient
sudo ifconfig
```

```plaintext
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.122.91  netmask 255.255.255.0  broadcast 192.168.122.255
```
The dhcp server allocated the first available address to the client (192.168.122.91).

Check resolv.conf:

```bash
sudo nano /etc/resolv.conf
```

```plaintext
domain iavdhcp
search iavdhcp
nameserver 192.168.122.13
```

### Step 4: Configure DHCP with a static address for client

For example: 192.168.122.20

First, you need to find out the client's MAC address:

```bash
ip a
```

Then we configure dhcp on the server:

```bash
sudo nano /etc/dhcp/dhcpd.conf

```

```plaintext
subnet 192.168.122.0 netmask 255.255.255.0 {
   range 192.168.122.91 192.168.122.111;
}

host static_client {
   hardware ethernet 08:00:27:d1:2e:7a;
   fixed-address 192.168.122.20;
}
```

where is 08:00:27:d1:2e:7a - MAC address

Restart DHCP service

```bash
sudo systemctl restart isc-dhcp-server
```

On the client machine:

```bash
sudo ifconfig
```

```plaintext
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.122.20  netmask 255.255.255.0  broadcast 192.168.122.255
```

The dhcp server allocated a fixed address to the client (192.168.122.20).
