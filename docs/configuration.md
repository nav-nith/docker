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

### Creating Override.conf
Override.conf file is created in path /etc/systemd/system/docker.service.d/override.conf
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
Remote access to Docker daemon is configured in Docker daemon statup options. Default Startup options are available at /lib/systemd/system/docker.service or /usr/lib/systemd/system/docker.service depending on docker installation.
Standard overrides can be configured to /etc/systemd/system/docker.service. But recommended way to override daemon config is by creating a /etc/systemd/system/docker.service.d/override.conf file and edit only relevant option
