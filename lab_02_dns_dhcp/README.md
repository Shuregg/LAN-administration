
# DNS, DHCP

## Setting up DNS with BIND9

### Step 1: Installing BIND9 on both virtual machines

install BIND9 on both the server and client machines:

```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc -y
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
                             2024100601   ; Serial
                             3h           ; Refresh
                             1h           ; Retry
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
   $TTL    604800
   iav.miet.stu.        IN       SOA     srv.iav.miet.stu. admin.iav.miet.stu. (
                             2024100601   ; Serial
                             3h           ; Refresh
                             1h           ; Retry
                             1w           ; Expire
                             1h           ; Negative Cache TTL
   )
   iav.miet.stu.        IN       NS      srv.iav.miet.stu.
   13                   IN       PTR     srv.iav.miet.stu.
   12                   IN       PTR     cli.iav.miet.stu.
   14                   IN       PTR     cli2.iav.miet.stu.
   ```

#### 5. Checking the zone files

Check the zone files for errors:

```bash
sudo named-checkzone iav.miet.stu /etc/bind/zones/db.iav.miet.stu
sudo named-checkzone 122.168.192.in-addr.arpa /etc/bind/zones/db.192.168.122
```

#### 6. Restarting BIND9 and testing

Restart the service:

```bash
sudo systemctl restart bind9
```

Test the availability of the machines by pinging their names:

```bash
ping srv.iav.miet.stu
```

```plaintext
PING srv.iav.miet.stu (192.168.122.13) 56(84) bytes of data
64 bytes from 192.168.122.13 (192.168.122.13): icmp_seq=1 ttl=64 time=0.017 ms
64 bytes from 192.168.122.13 (192.168.122.13): icmp_seq=2 ttl=64 time=0.032 ms
64 bytes from 192.168.122.13 (192.168.122.13): icmp_seq=3 ttl=64 time=0.072 ms
64 bytes from 192.168.122.13 (192.168.122.13): icmp_seq=4 ttl=64 time=0.067 ms
```

```bash
ping cli.iav.miet.stu
```

```plaintext
PING cli.iav.miet.stu (192.168.122.12) 56(84) bytes of data
64 bytes from 192.168.122.12 (192.168.122.12): icmp_seq=1 ttl=64 time=0.054 ms
64 bytes from 192.168.122.12 (192.168.122.12): icmp_seq=2 ttl=64 time=0.037 ms
64 bytes from 192.168.122.12 (192.168.122.12): icmp_seq=3 ttl=64 time=0.064 ms
64 bytes from 192.168.122.12 (192.168.122.12): icmp_seq=4 ttl=64 time=0.082 ms
```

```bash
ping cli2.iav.miet.stu
```

```plaintext
ping: cli2.iav.miet.stu: Unknown name or service
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