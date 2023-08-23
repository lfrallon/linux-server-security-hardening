## Detect DDoS attack on Linux System 

### 1. Log in to the Linux server using SSH.

### 2. Run the below command to find the IP Addresses connected to your Linux system.

```
    netstat -anp |grep 'tcp\|udp' | awk '{print $5}' | cut -d: -f1 | sort | uniq -c
```

### 3. Run the below command to find the source IP address and the number of connections of the same IP Address to your Linux system.

```
    netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -n
```

### 4. Run the below command to check the active TCP connection number on port 80.

```
    netstat -n | grep :80 |wc -l
```

### 5. If there is an IP Address with too many connections, it could be an attacker's IP address. You can block it using the below command.

```
    route add ipaddress reject
```

### 6. You can also block it using the iptables on a Linux machine.


## Using Nginx to prevent DDoS attack

> Source: [DDoS attack prevention with Nginx](https://inmediatum.com/en/blog/engineering/ddos-attacks-prevention-nginx/)

### Nginx worker connections

```
    events {
       worker_connections 50000;
    }
```

### Limiting requests rate

```
    limit_req_zone $binary_remote_addr zone=one:1m rate=30r/m;

    server {
        location /wp_login.php {
            limit_req zone=one;
        }
    }
```

### Limiting number of connections

```
    limit_conn_zone $binary_remote_addr zone=two:1m;

    server {
        location / {
            limit_conn two 10;
        }
    }
```

### Timeout parameters

```
    server {
        client_body_timeout 5s;
        client_header_timeout 5s;
    }
```

### Limit requests size

```
    client_body_buffer_size 200K;
    client_header_buffer_size 2k;
    client_max_body_size 200k;
    large_client_header_buffers 3 1k;
```

### Blacklist IP adresses

```
    location / {
        deny 123.123.123.3;
        deny 123.123.123.0/24;
    }
```

### Whitelist IP adresses

```
    location / {
        allow 123.123.123.3;
        deny all;
    }
```

### Blocking access to a file or location

```
    location /register.php {
        deny all;
        return 444;
    }
```

### Enable sysctl based protection

> Edit the file /etc/sysctl.conf, and set these two lines to 1 like this:

```
    net.ipv4.conf.all.rp_filter = 1
    net.ipv4.tcp_syncookies = 1
```

- The first parameter enables protection against IP spoofing, and the second allows TCP SYN cookie protection.

### Nginx as load balancer

```
    upstream domain {
        server 123.123.123.1:80 max_conns=100;
        server 123.123.123.2:80 max_conns=100;
        queue 20 timeout=10s;
    }
```

## Additional Nginx DDoS protection

> [Nginx: Mitigate DDoS Attack](https://www.nginx.com/blog/mitigating-ddos-attacks-with-nginx-and-nginx-plus/)