# Docker Linux Configuration

Docker daemon in linux can be configured in three ways.
- Systemd Configuration
- SysVinit configuration
- Daemon.json settings

By default the systemd configuration files controlling the service are under the folder/usr/lib/systemd/system. This is also evident in the Loaded: line in the output of the systemctl status command.
```
sudo systemctl status docker.service | grep Loaded
```
The docker.service file contains all the configuration options for the docker process. The options are organized under sections [Unit], [Service] and [Install]. The format of the file is very similar to an init file, with name=value pairs in each section.
To override the daemon options, it is recommended not to edit this file. Instead, an override file should be used. The override file would contain just the options that need to be changed along with the section heading it is under.
