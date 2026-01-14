# main steps

## Version Matrix

| Component | Version | Release |
|----------|---------|---------------|
| Operating System | Ubuntu 24.04 LTS Server | - |
| NGINX Plus | 1.29.3 | nginx-plus-r36-p1 |
| Apache2 | 2.4.58 | - |
| WRK | 4.2.0 | - |

## Server Specification

| server_name | core | ram | IP |
|----------|---------|---------------|---------------|
| Load_Generator | 8 | 16GiB | 10.110.121.81 |
| NGINX_Plus_Server | 8 | 16GiB | 10.110.121.85 |
| Apache_Server | 8 | 16GiB | 10.110.121.175 |

## Deployment Diagram
![Request Flow](images/diagram.png)

## Installation 

### 1. WRK
Follow the steps to install WRK on Load Generator server.
#### Dependency installation
``` Log
# apt install -y git build-essential libssl-dev git zip unzip
```

#### Clone Repository and make
``` Log
# git clone https://github.com/wg/wrk.git
# cd wrk/
# make
```

#### Copy binary and verify
``` Log
# cp wrk /usr/local/bin/
# wrk --version
```
``` Log
wrk 4.2.0 [epoll] Copyright (C) 2012 Will Glozer
Usage: wrk <options> <url>                            
  Options:                                            
    -c, --connections <N>  Connections to keep open   
    -d, --duration    <T>  Duration of test           
    -t, --threads     <N>  Number of threads to use   
                                                      
    -s, --script      <S>  Load Lua script file       
    -H, --header      <H>  Add header to request      
        --latency          Print latency statistics   
        --timeout     <T>  Socket/request timeout     
    -v, --version          Print version details      
                                                      
  Numeric arguments may include a SI unit (1k, 1M, 1G)
  Time arguments may include a time unit (2s, 2m, 2h)
```

### Apache
Install and configure Apache HTTP Server on the server dedicated from Apache.

#### Install Apache HTTP Server
``` Log
# apt install apache2 -y
```
``` Log
# systemctl start apache2
```
``` Log
# systemctl status apache2
```
``` Log
# systemctl enable apache2
```

#### Configure Apache HTTP Server
``` Log
# vim /etc/apache2/sites-available/app.conf
```
``` Log
<VirtualHost *:80>
    ServerName 10.110.121.175 # change IP/domain as per need

    DocumentRoot /var/www/html

    <Directory /var/www/html>
        Options -Indexes
        AllowOverride None
        Require all granted
    </Directory>

    # Force JSON content type
    <Files "health.json">
        ForceType application/json
    </Files>

    DirectoryIndex health.json

    ErrorLog  ${APACHE_LOG_DIR}/health_error.log
    CustomLog ${APACHE_LOG_DIR}/health_access.log combined
</VirtualHost>
```
``` Log
# vim /var/www/html/health.json
```
``` Log
{
  "status": "OK",
  "service": "ApacheWebServer",
  "version": "1.0.0"
}
```
``` Log
# rm -f /etc/apache2/sites-available/000-default.conf
# a2ensite app
# apachectl configtest
# systemctl reload apache2
```
Verify from the Apache Server or from where the IP is reachable
``` Log
curl http://10.110.121.175/
```
``` Log
{
  "status": "OK",
  "service": "ApacheWebServer",
  "version": "1.0.0"
}
```


exporter


## Monitoring and Visulizaition

### Prometheus


### Grafana

## Results
### NGINX Plus [nginx/1.29.3 (nginx-plus-r36-p1)]
``` Log
wrk -t4 -c100 -d300s --latency http://10.110.121.85
Running 5m test @ http://10.110.121.85
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     4.31ms    3.59ms 131.65ms   98.23%
    Req/Sec     6.02k   627.62    10.28k    84.88%
  Latency Distribution
     50%    3.72ms
     75%    4.52ms
     90%    5.79ms
     99%    8.91ms
  7193149 requests in 5.00m, 2.12GB read
Requests/sec:  23972.00
Transfer/sec:      7.25MB
```

### Apache [2.4.58]
``` Log
wrk -t4 -c100 -d300s --latency http://10.110.121.175
Running 5m test @ http://10.110.121.175
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.93ms    1.19ms  58.85ms   77.59%
    Req/Sec     6.26k   501.40     9.76k    77.93%
  Latency Distribution
     50%    3.63ms
     75%    4.12ms
     90%    5.65ms
     99%    7.06ms
  7479000 requests in 5.00m, 2.15GB read
  Socket errors: connect 0, read 124210, write 0, timeout 0
Requests/sec:  24924.08
Transfer/sec:      7.35MB
```

