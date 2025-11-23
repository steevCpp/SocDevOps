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


```
version: '3.8'
services:
  traefik:
    image: traefik:v2.9.6
    ports:
      - 80:80
      - 443:443
    deploy:
      placement:
        constraints:
          - node.role == manager
      labels:
        - traefik.enable=true
        - traefik.docker.network=traefik-public
        - traefik.http.routers.traefik.middlewares=admin-auth
        - traefik.http.middlewares.admin-auth.basicauth.users=<user>:<hashed_password>
        - traefik.http.routers.traefik.rule=Host(`soc`) && (PathPrefix(`/dashboard`) || PathPrefix(`/api`))
        - traefik.http.routers.traefik.entrypoints=https
        - traefik.http.routers.traefik.tls=true
        - traefik.http.routers.traefik.service=api@internal
        - traefik.http.services.traefik-svc.loadbalancer.server.port=8080
    command:
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --providers.docker.swarmmode=true
      - --entrypoints.http.address=:80
      - --entrypoints.https.address=:443
      - --providers.file.directory=/certificates/
      - --providers.file.watch=true
      - --log=true
      - --log.level=DEBUG
      - --api
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /soc/volumes/traefik/certificates/:/certificates/
    networks:
      traefik-public: {}
networks:
  traefik-public:
    external: true
```

```
mkdir -p /soc/volumes/traefik/certificates/ && cd /soc/volumes/traefik/certificates/
```

```
openssl req -x509 -newkey rsa:4096 -keyout soc.key -out soc.crt -sha256 -days 3650 -nodes
```

```
vim /soc/volumes/traefik/certificates/certificates.yaml
```

```
docker stack deploy -c /soc/docker-config/docker_compose_traefik.yml traefik
```

```
docker stack ls
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

