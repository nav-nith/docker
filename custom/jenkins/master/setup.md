# README
## Pre-Requisite
1. Set Proxy
If you are behind corporate proxy please ensure that proxy environment variables for http_proxy, https_proxy, ftp_proxy, socks_proxy and no_proxy are set.
You can set these environment variables using following commands.
```
export http_proxy=http://<username>:<password>@<proxy>:<port>
export https_proxy=https://<username>:<password>@<proxy>:<port>
export ftp_proxy=ftp://<username>:<password>@<proxy>:<port>
export socks_proxy=socks://<username>:<password>@<proxy>:<port>
export no_proxy=localhost,127.0.0.0/8,127.0.1.1,local.home
```
2. Persitant Storage
All Jenkins data and configurations required to be stored in a persistant drive to esure the data remains intact between restart of container.
This can be done by creating a folder in host PC and mounting it as a volume when starting the jenkins container.

For example:
 Create a folder ~/docker/jenkins/jenkins0 in user home which will be mounted with -v option in the command used for starting container.

## Start Container
```
docker run --name jenkins0 -d --restart=always -m 4G -p 127.0.0.1:8080:8080 -p 50000:50000 -e http_proxy=$http_proxy -e https_proxy=$https_proxy -e ftp_proxy=$ftp_proxy -e socks_proxy=$socks_proxy -e no_proxy=$no_proxy -e "JENKINS_SLAVE_AGENT_PORT=50000" -e "TZ=Asia/Calcutta" -e "JENKINS_OPTS=--prefix=/jenkins0" -e "JAVA_OPTS=-Dgroovy.use.classvalue=true -Dhudson.plugins.active_directory.ActiveDirectorySecurityRealm.forceLdaps=true -XX:+AlwaysPreTouch -XX:NumberOfGCLogFiles=5 -XX:+UseGCLogFileRotation -XX:GCLogFileSize=20m -Djava.awt.headless=true -server -Xmx2048m -Xms1024m -XX:MaxPermSize=512m -XX:PermSize=256m -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled" -v ~/docker/jenkins/jenkins0:/var/jenkins_home docker pull jenkins/jenkins
```
## SSL Certificate Generation
Following instructions will enable you to generate Self Signed Cerificate files

## Configuring Host NGINX for Proxy-pass
NGINX site configuration files are located at */etc/nginx/site-available/*. Modify *default* file as follows to proxy pass traffic to jenkins listening on localhost port 8080.
```
upstream jenkins00 {
  server localhost:8080 fail_timeout=0;
}

server {
  listen 80;
  listen [::]:80;

  server_name <fqdn server name>;
  return 301 https://$server_name$request_uri;
}

server {
  listen 443 ssl;
  listen [::]:443 ssl;
  
  server_name <fqdn server name>
  #error_page 497 301 =307 https://$server_name:443$request_uri;
  
  #SSL Configuration
  ssl on;
  ssl_protocol TLSv1 TLSv1.1 TLSv1.2;
  ssl_certificate /etc/nginx/.ssl/<certificate file name>.crt;
  ssl_certificate_key /etc/nginx/.ssl/<key file name>.key;
  #ssl_clinet_certificate /etc/nginx/.ssl/<client certificate file name>.crt;
  
  location ^~ /jenkins0/ {
    # convert inbount WAN
    proxy_pass http://jenkins00/jenkins0/;
    
    # Rewrite HTTPS requests from WAN to HTTP requests on LAN
    proxy_redirect http:// https://;
    
    senfile off;
    
    proxy_set_header Host $host:$server_port;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header
    
  }
  
}
```
