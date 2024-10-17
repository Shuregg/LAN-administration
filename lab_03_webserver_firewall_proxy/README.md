# Web-server: nginx

## Prerequirement

```bash
sudo apt install netcat
sudo apt install nginx
sudo apt install squid
```

## 1. HTTPS request

1. Make an HTTP request for a page with information about TCP/IP network located at `lib.ru/unixhelp/network.txt`. Save the HTTP response.

    ```bash
    echo -e "GET http://lib.ru/unixhelp/network.txt HTTP/1.1\n\n" | nc lib.ru 80 >> ~/network.html
    ```

2. Open the network.html file using browser and make sure that the text is formatted correctly.

    ```bash
    firefox ~/network.html
    ```
    
    ![html via browser](images/img_01_html_via_browser.png)

## 2. Nginx web server

0. Check that Nginx is active

    ```bash
    sudo systemctl status nginx.service
    firefox localhost
    ```
    
    ![alt text](images/img_02_welcome_to_nginx.png)

    Creating configuration backup:
    
    ```bash
    sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.orig
    ```

1. Create an HTML file in the directory /var/www/site/.

    ```bash
    sudo mkdir /var/www/site 
    sudo nano /var/www/site/site.html
    ```

2. Install the Nginx web server and configure it. When making an HTTP request, it should send the HTML file you created.


    ```bash
    sudo nano /etc/nginx/nginx.conf
    ```

    ```plaintext
    user www-data;
    worker_processes auto;
    pid /run/nginx.pid;
    
    events {
        worker_connections 10;
    }
    
    http {
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 60;
        access_log /var/www/site/access.log;
        error_log /var/www/site/error.log;
        
        server {
            listen 80 default_server;
            root /var/www/site;
            index site.html;
            server_name site.iav.miet.stu;
        }
    }
    ```

    You should replace `www-data` by your username!


    Checking the config for errors:
    
    ```bash
    sudo nginx -t
    ```

    You will see the following if there are no errors:
    ```plaintext
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

    Now restart nginx daemon:
    ```bash
    sudo systemctl restart nginx
    ```

3. Add the created page to the list of DNS records and open it from the client's machine using the site domain name `site.iav.miet.stu`.

    On the server:
    ```bash
    sudo nano /etc/bind/zones/db.iav.miet.stu
    ```

    Insert `site IN A 192.168.122.13` to the config:
    
    ```plaintext
    $TTL    604800
    iav.miet.stu.        IN       SOA     srv.iav.miet.stu. admin.iav.miet.stu (
        2024100601
        3h
        1h
        1w
        1h
    )
    iav.miet.stu.        IN       NS      srv.iav.miet.stu.
    cli                  IN       A       192.168.122.12
    srv                  IN       A       192.168.122.13
    cli2                 IN       A       192.168.122.14
    site                 IN       A       192.168.122.13
    ```

    ```bash
    sudo systemctl restart bind9
    ```


    On the client:

    Trying to make a request:
    ```bash
    curl http://site.iav.miet.stu
    ```
    
    You will see:
    ```html
    <!DOCTYPE html>
    <html>
        <head>
            <title>
                My first page!
            </title>
        </head>
    
        <body>
            Hello everyone!
        </body>
    </html>
    ```

    Or open this URL via browser:
    
    ```bash
    firefox site.iav.miet.stu
    ```

    ![Connecting to our html page from the client](images/img_03_check_site_from_client.png)

## 3. Firewall

1. Turn on integrated firewall on the server and reset the current state.

2. Add a rule, that prohibiting connection to the server via the `sftp` procotol (`port 22`).

3. Add a rule allowing all incoming packets from the client.

4. Check the connetction to the server via `sftp` and `ftp` protocols.

5. Delete rule that prohibiting connection via the sftp (without reset). Check the connection.

6. Turn of the firewall.

## 4. Squid proxy server

1. Install the squid on the server.

2. Configure the proxy server so that access to the `vk.com` is restricted during business hours, and the rest of the sites were launched freely.

3. Make sure that the client has access to the site you created. Prohibit the client from connecting to the site using a proxy server.  