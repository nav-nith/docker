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

### Configure DNS
System DNS settings reflects in /etc/resolve.conf of host OS for containers starting in host network (docker run --network=host). 
For containers starting in private network, the DNS settings required to be configured for daemon in override file.

```
sudo vi /etc/systemd/system/docker.service.d/override.conf
[Service]
Type=notify
ExecStart=
ExecStart=/usr/bin/dockerd
Append following parameters to the ExecStart: 
--dns 8.8.8.8 --dns 8.8.8.4
```

