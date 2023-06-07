# LimeSec Proxy

Use to run with rootless docker daemon --> install as follow 
https://docs.docker.com/engine/security/rootless/

Example configuration of Traefik2, DockerSockProxy, Authelia and Crowdsec.

### Cert resolver

Traefik as 2 cert resolver : 

cf : For cloudflard and wildcard domaine cert
le : Classic let's encrypt cert. 

By default, cf are enable.
Manual A, AAAA, or CNAME record ar needed, it's not a big deal for security reason.

## Cloudflare create 

## Rootless docker user configuration.

Socket docker : `/run/user/1000/docker.sock`
Creation network : `docker network create --driver=bridge --subnet=172.20.0.0/24 traefik_network`

### CPU io abd Mem control pour rootless user.

```s
sudo mkdir -p /etc/systemd/system/user@.service.d
sudo nano /etc/systemd/system/user@.service.d/delegate.conf
```

```s
[Service]
Delegate=cpu cpuset io memory pids
```

```s
sudo systemctl daemon-reload
cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/cgroup.controllers

>>> cpuset cpu io memory pids # OK
```


### <1024 for traefik

```s
sudo setcap cap_net_bind_service=ep $(which rootlesskit)
systemctl --user restart docker
```

### ip_unprivileged fwd for crowdsec :

```s
sudo nano /etc/sysctl.d/99-rootless.conf

<<<net.ipv4.ip_unprivileged_port_start=0>>>

sudo sysctl --system
```

### Set slirp4netns for Source IP fwd

```s
code ~/.config/systemd/user/docker.service.d/override.conf

<<< 
[Service]
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_PORT_DRIVER=slirp4netns"
>>>

systemctl --user daemon-reload
systemctl --user restart docker
```

