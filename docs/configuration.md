# Docker Linux Configuration

Docker daemon in linux can be configured in three ways.
- Systemd Configuration
- SysVinit configuration
- Daemon.json settings

With the recent Docker releases, its recommended to use systemd configuration for configuring Docker Daemon. But please esnure that the SysVint and Daemon.json configurations are not set when you are using Systemd.

## Systemd Configuration

By default the systemd configuration files controlling the service are under the folder/usr/lib/systemd/system. This is also evident in the Loaded: line in the output of the systemctl status command.
```
sudo systemctl status docker.service | grep Loaded
```
The docker.service file contains all the configuration options for the docker process. The options are organized under sections [Unit], [Service] and [Install]. The format of the file is very similar to an init file, with name=value pairs in each section.
To override the daemon options, it is recommended not to edit this file. Instead, an override file created at /etc/systemd/system/docker.service.d/override.conf should be used. The override file would contain just the options that need to be changed along with the section heading it is under.

Remember to reload the Docker Daemon configuration and restart Docker Daemon using following command after any modification to the configurations
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```
### Creating Override.conf
Override.conf file is created in path /etc/systemd/system/docker.service.d/override.conf. Please note that you may need to create the directories if the path is not available by default.
Add followig lines to override.conf to ensure previous ExecStart defined in default config file /usr/lib/systemd/system/docker.service is invalidated before setting new value:
```
[Service]
Type=notify
ExecStart=
ExecStart=/usr/bin/dockerd
```
### Configure DNS
System DNS settings reflects in /etc/resolve.conf of host OS for containers starting in host network (docker run --network=host). 
For containers starting in private network, the DNS settings required to be configured for daemon in override file by appending following parameters to the ExecStart line.
```
--dns <dns IP1> --dns <dns IP2>
```
Exampmple:
Your override file will look as follows when you add google DNS
```
[Service]
Type=notify
ExecStart=
ExecStart=/usr/bin/dockerd --dns 8.8.8.8 --dns 8.8.8.4
```
### Configure Proxy
If you are behind an HTTP proxy server, for example in corporate settings, you will need to add this configuration in the Docker systemd service file.
Create a file called /etc/systemd/system/docker.service.d/http-proxy.conf that adds environment variables to define proxy as follows:
```
[Service]
Environment="HTTP_PROXY=http://username:password@proxy:port" "HTTPS_PROXY=http://username:password@proxy:port" "FTP_PROXY=ftp://username:password@proxy:port" "NO_PROXY=localhost,127.0.0.0/8,127.0.1.1,local.home"
```
If you have special charecters in your password, please use HTML UTF-8 notation of the special charecters (Example: '@' is represented as %40)

### Remote Access
Remote access to Docker daemon is configured in Docker daemon statup options. You can override the daemon settings by adding relevant options to /etc/systemd/system/docker.service.d/override.conf.

The docker socket can be configured on any port with the dockerd -H option. Common docker ports are:
2375: unencrypted docker socket, remote root passwordless access to the host.
2376: tls encrypted socket, most likely this is your CI servers 4243 port as a modification of the https 443 port
2377: swarm mode socket, for swarm managers, not for docker clients
5000: docker registry service
4789 and 7946: overlay networking

Only the first two are set with dockerd -H, swarm mode can be configured as part of docker swarm init --listen-addr or docker swarm join --listen-addr.

To enable access to the docker daemon from remote you need to append following parameters to line containing ExecStart=/usr/bin/dockerd
```
-H tcp://0.0.0.0:4243 -H tcp://0.0.0.0:2375 -H fd://
```
I strongly recommend disabling or restricting 2375 port to specific clients and securing your docker socket. It's trivial to remotely exploit this port to gain full root access without a password from remote. The command to do so is as simple as:
```
docker -H $your_ip:2375 run -it --rm  --privileged -v /:/rootfs --net host --pid host busybox
````
That can be run on any machine with a docker client to give someone a root shell on your host with the full filesystem available under /rootfs, your network visible under ip a, and every process visible under ps -ef.

To setup TLS security on the docker socket, see these instructions. 
https://docs.docker.com/engine/security/https/
