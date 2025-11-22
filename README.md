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
hostnamectl set-hostname soc
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

```
mkdir -p /soc/volumes/jenkins && chown -R 1000 /soc/volumes/jenkins
```

### vim /soc/docker-config/docker_compose_jenkins.yml

```
version: '3.8'
services:
  jenkins:
    image: jenkins/jenkins:jdk21
    ports:
      - 50000:50000
    environment:
      - "JENKINS_OPTS=--prefix=/jenkins"
    deploy:
      labels:
        - traefik.enable=true
        - traefik.docker.network=traefik-public
        - traefik.http.routers.jenkins.rule=Host(`soc`) && PathPrefix(`/jenkins`)
        - traefik.http.routers.jenkins.entrypoints=https
        - traefik.http.routers.jenkins.tls=true
        - traefik.http.services.jenkins-svc.loadbalancer.server.port=8080
    volumes:
      - "/soc/volumes/jenkins:/var/jenkins_home"
      - "/etc/localtime:/etc/localtime:ro"
    networks:
      - traefik-public
networks:
  traefik-public:
    external: true
```

```
docker stack deploy -c /soc/docker-config/docker_compose_jenkins.yml jenkins
```

- Trivy 

- Nikto 

