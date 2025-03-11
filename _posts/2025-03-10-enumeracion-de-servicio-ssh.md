---
layout : single
title: Enumeración de servicio SHH
date: 2025-03-07
classes: wide
header:
  teaser: /assets/images/slae32.png
categories:
  - Seguridad informática
  - Protocolo de Red
tags:
  - SSH
  - Enumeración de servicio
  - Seguridad SSH
  - Protocolos de comunicación 
  - Puerto SSH
  - Explotación de red
  - Herramientas de enumeración
  - Servicios activos
---
La enumeración de servicio **SSH** se refiere al proceso de identificar y obtener información sobre los servicios **SSh** (Secure Shell) que están activos en una red o sistema. Este proceso es común en auditorías de seguridad, prueba de penetración y evaluaciones de vulnerabilidades. El objetivo es detectar configuraciones erróres, vulnerabilidades conocidas o debilidades que un atacante podría aprovechar. 

El servicio **SHH** es un proctocolo utilizado para acceder de manera remota a un sistema, enciprtando la comunicación para proteger los datos durante la tramisión.


### Probando Enumeración de servicio SHH en entorno controlado
**docker hub**
[linuxserver](https://hub.docker.com/r/linuxserver/openssh-server)

```
docker run -d \
  --name=openssh-server \
  --hostname=openssh-server `#optional` \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e PUBLIC_KEY=yourpublickey `#optional` \
  -e PUBLIC_KEY_FILE=/path/to/file `#optional` \
  -e PUBLIC_KEY_DIR=/path/to/directory/containing/_only_/pubkeys `#optional` \
  -e PUBLIC_KEY_URL=https://github.com/username.keys `#optional` \
  -e SUDO_ACCESS=false `#optional` \
  -e PASSWORD_ACCESS=false `#optional` \
  -e USER_PASSWORD=password `#optional` \
  -e USER_PASSWORD_FILE=/path/to/file `#optional` \
  -e USER_NAME=linuxserver.io `#optional` \
  -e LOG_STDOUT= `#optional` \
  -p 2222:2222 \
  -v /path/to/openssh-server/config:/config \
  --restart unless-stopped \
  lscr.io/linuxserver/openssh-server:latest
```

**Modificamos a nuestra manera**
```bash
sudo docker run -d \
  --name=openssh-server \
  --hostname=machine-learning \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e PASSWORD_ACCESS=true \
  -e USER_PASSWORD=exe123  \
  -e USER_NAME=exequiel \
  -p 2222:2222 \
  -v /path/to/openssh-server/config:/config \
  --restart unless-stopped \
  lscr.io/linuxserver/openssh-server:latest;
```

**Nos conectamos**
```bash
ssh exequiel@127.0.0.1 -p 2222
```

**Con hydra podemos aplicar fuerza bruta**
```bash
hydra -l exequiel -P diccionario_de_password ssh://127.0.0.1 -s 2222 -t 4
```

**Cuando hay un `ssh` o `ftp` podemos averiguar cuál es el `CODENAME`**
El `CODENAME` es para ocultar la verdadera identidad.

[oficial ubuntu](https://hub.docker.com/_/ubuntu)
**Elegimos cualquier versión dentro de la web y creamos un Dockerfile**
```Dockerfile
FROM ubuntu:20.04

EXPOSE 22

RUN apt update && apt install ssh -y

ENTRYPOINT service ssh start && /bin/bash
```

**Corremos el contenedor**
```bash
sudo docker build -t my_ssh_server .
sudo docker run -dit -p22:22 --name mySSHServer my_ssh_server
```

**Verificar que conecte**
```bash
ssh exequiel@1270.0.1 
```

**Ahora podemos hacer el reconocimiento**
```bash
nmap -sCV -p22 127.0.0.1
```

**Salida**
```bash
nmap -sCV -p22 127.0.0.1
Starting Nmap 7.93 ( https://nmap.org ) at 2025-03-10 23:27 -03
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00018s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0) #En esta linea Sacamos el CODENAME
| ssh-hostkey: 
|   3072 384108aaf788b9f8ed49a0d7de29fbe7 (RSA)
|   256 e9d4ed395643a2fa1c6d984936925b8d (ECDSA)
|_  256 a62db7e50e0932b40179d05e4b74b5c1 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.96 seconds
```
*En nuestro navegador ponemos lo siguiente `OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 launchpad` para que te muestre los resultados* 
