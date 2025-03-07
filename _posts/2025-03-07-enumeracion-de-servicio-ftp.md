---
layout : single
title: Enumeración de servicio FTP
date: 2025-03-07
classes: wide
header:
  teaser: /assets/images/slae32.png
categories:
  - Seguridad informática
  - Protocolo de Red
  - Servicio de transferencia de Archivos
tags:
  - FTP
  - Enumeración de servicio
  - Seguridad FTP
  - Protocolos de comunicación 
  - Puerto FTP
  - Explotación de red
  - Herramientas de enumeración
  - Servicios activos
---
Proceso de recopilar información sobre un servicio FTP(file transfer protocol) activos y sus características, como los usuarios, los permisos, las versiones del servicio y otros detalles relevantes. Esta técnica es común en el ámbito de la seguridad informática, especialmente en pruebas de penetración y auditorías de seguridad.

Durante la enumeración de servicios FTP, se puede obtener datos como:
1. Versión del servidor FTP: Saber que versión del software FTP está en ejecución puede ser útil para identificar vulnerabilidades conocidas asociadas con esa versión.

2. Usuarios y contraseñas: Algunos servidores FTP permiten la enumeración de usuarios, lo que significa que un atacante podria intentar listar los nombres de usuario válidos en el servidor FTP. Esto es más probable en servidores mal configurados.

3. Directorios y archivos accesibles: A través de la enumeración, se puede descubrir directorios y archivos a los que el usuario puede tener acceso, incluso si no está autenticado. Esto depende de la configuración del servidor.

4. Puertos abiertos y servicios activos: También se puede detectar puertos relacionados con FTP (por ejemplo el puerto 21, para conexiones estándar) y otros servicios asociados, FTPS o SFTP.

5. Configuración de seguridad: La enumeración también puede ayudar a identificar configuraciones de seguridad débiles, como la falta de cifrado (cuando FTP no es seguro) o el uso de contraseñas débiles.

### Herramientas comunes para la enumeración FTP:
- Nmap: escanear puertos y detectar servicios.
- Metasploit: Para realizar ataques o recopilar información detallada sobre el servicio FTP.
- Hydra o Medusa: Para intentar ataques de fuerza bruta y enumerar usuarios.
- Nikto: Para buscar vulnerabilidades conocidas en servidores FTP.

---
### enumeración de servicio FTP en entorno controlado
**Instalaciones necesarias**
- Debian
```bash
apt install ftp
apt install hydra
apt install nmap
```
 **Desplegamos en docker**
[ftp-server-auth](https://github.com/garethflowers/docker-ftp-server)

```bash
docker run \
	--detach \
	--env FTP_PASS=exe123 \
	--env FTP_USER=exequiel \
	--env PUBLIC_IP=192.168.0.1 \
	--name my-ftp-server \
	--publish 20-21:20-21/tcp \
	--publish 40000-40009:40000-40009/tcp \
	--volume /data:/home/user \
	garethflowers/ftp-server
```
*modificamos lo que es FTP_USER y el FTP_USER*

**Escaneo de puerto 21**
```bash
nmap -sCV -p21 172.0.0.1
```

- Esto es si hay autenticación
**Con hydra vamos a plicar fuerza bruta sabiendo cuál es el nombre de usario usando un diccionario**
```bash
hydra -l exequiel -P password.txt ftp://127.0.0.1 -t 4
```
`-l` especifica el nombre de usuario
`-P` para archivos de contraseñas
`-t` hasta cuanta tareas queremos que realicé

**Salida**
![ftp-hydra](~/damixn.gitgub.io/assets/images/ftp-service/ftp-hydra.png)

- Esto si es usuario anonymous
[fpt-server-anonimus](https://github.com/metabrainz/docker-anon-ftp)

```
docker run -d -p 20-21:20-21 -p 65500-65515:65500-65515 -v /tmp:/var/ftp:ro metabrainz/docker-anon-ftp
```

```bash
ftp 127.0.0.1
```
*Ingresamos usuario `anonymous` y damos enter vacio en cuando nos pide "password"*

> [!NOTE]
> Lo que primero que hay que hacer siempre es ver si como usuario anonymous podemos conectarnos vía FTP
