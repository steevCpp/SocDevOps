# SocDevOps
Soc d'analyse de vulnérabilité des FS, Docker images, apache...
## Infra 
- VM centos7, sudo su -

```
yum update && upgrade -y
```

```
yum install -y yum-utils
```

```
mkdir -p /soc/docker-config
```

- Docker

Installation

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

```
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

```
yum systemctl start docker
```

- Docker-swarm 

```
docker swarm init
```

- Docker Network Traefik

```
docker network create --driver=overlay --scope=swarm traefik-public
```

- Jenkins 

- Trivy 

- Nikto 

