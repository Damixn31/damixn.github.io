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
La **escalada de privilegios** es una técnica utilizada en la ciberseguridad que permite a un atacante obtener mayores privilegios dentro de un sistema, de modo que pueda ejecutar comandos o acceder a recursos que normalmente estarían restringidos. Este tipo de ataque puede tener consecuencias graves si no se detecta a tiempo, ya que un atacante con privilegios elevados puede comprometer totalmente un sistema.

## Tipos de Escalada de Privilegios

Existen dos tipos principales de escalada de privilegios:

### 1. **Escalada de Privilegios Vertical**
Este tipo de escalada ocurre cuando un atacante obtiene privilegios superiores a los que ya posee. Por ejemplo, un usuario normal podría obtener privilegios de administrador (root en sistemas Unix o administrador en Windows). La escalada vertical es más común en sistemas donde los administradores no gestionan correctamente las configuraciones de acceso.

**Ejemplo:**
- Un atacante se infiltra en un sistema como usuario limitado.
- Utiliza vulnerabilidades o configuraciones incorrectas para ganar acceso a cuentas con privilegios más altos.

### 2. **Escalada de Privilegios Horizontal**
La escalada horizontal ocurre cuando un atacante se aprovecha de una vulnerabilidad para obtener acceso a otras cuentas de usuario, pero no a cuentas con más privilegios. Aunque este tipo de escalada no proporciona privilegios administrativos, sí permite a los atacantes comprometer otras cuentas y obtener acceso a datos sensibles.

**Ejemplo:**
- Un atacante se infiltra en la cuenta de un usuario y accede a los recursos o archivos que son privados para ese usuario.
- No puede obtener privilegios de administrador, pero sí puede robar información sensible de otros usuarios.

## Métodos Comunes de Escalada de Privilegios

A continuación, se describen algunos de los métodos más comunes que los atacantes utilizan para realizar una escalada de privilegios.

### 1. **Vulnerabilidades en Software**
Las vulnerabilidades de software, como errores de codificación o configuraciones incorrectas, son explotadas por los atacantes para escalar privilegios. Estas vulnerabilidades pueden ser locales o remotas y a menudo permiten la ejecución de código malicioso con privilegios más altos.

**Ejemplo:**
- **Buffer overflow**: Un atacante podría enviar datos excesivos a un programa vulnerable para sobrescribir la memoria y obtener control sobre el sistema con privilegios elevados.

### 2. **Sudo Misconfigurado (en sistemas Unix)**
En sistemas basados en Unix, si la configuración de `sudo` no está correctamente definida, los usuarios pueden ejecutar ciertos comandos con privilegios elevados sin restricciones. Los atacantes pueden aprovechar este malentendido para ejecutar comandos críticos que deberían estar restringidos.

**Ejemplo:**
- Un usuario con privilegios limitados puede tener acceso a `sudo` sin necesidad de autenticarse correctamente.

### 3. **Contraseñas Débiles**
La utilización de contraseñas débiles es una de las formas más simples de escalar privilegios. Los atacantes pueden adivinar o descifrar contraseñas mediante ataques de fuerza bruta, obteniendo acceso a cuentas de administrador o root.

**Ejemplo:**
- Un atacante utiliza un diccionario de contraseñas comunes para adivinar las credenciales de administrador.

### 4. **Exploits de Kernel (en sistemas Linux/Unix)**
Las vulnerabilidades en el **kernel** del sistema operativo pueden permitir a los atacantes ejecutar código malicioso a nivel de sistema. Un atacante con privilegios limitados puede usar estas vulnerabilidades para obtener acceso de root y tomar control total del sistema.

**Ejemplo:**
- Un atacante puede explotar una vulnerabilidad de kernel, como **Dirty COW** (CVE-2016-5195), para obtener acceso como root.

## Prevención de la Escalada de Privilegios

La mejor manera de prevenir la escalada de privilegios es asegurarse de que el sistema esté correctamente configurado y actualizado. Aquí algunas estrategias clave para mitigar este tipo de ataques:

1. **Actualizaciones de Seguridad Regulares**: Mantén el sistema y todas las aplicaciones actualizadas para corregir las vulnerabilidades conocidas.
   
2. **Revisión de Configuraciones de Sudo**: Asegúrate de que las configuraciones de `sudo` sean estrictas y que solo los usuarios autorizados puedan ejecutar comandos con privilegios elevados.
   
3. **Contraseñas Fuertes**: Implementa políticas de contraseñas seguras y usa autenticación de dos factores (2FA) siempre que sea posible.
   
4. **Seguridad del Kernel**: Implementa parches y medidas de seguridad para el kernel, y utiliza herramientas como **SELinux** o **AppArmor** para agregar capas de seguridad adicionales.
   
5. **Auditoría de Accesos y Registros**: Realiza auditorías regulares de acceso a las cuentas de usuario y de las acciones realizadas con privilegios elevados.

### Herrmientas de escalada de privilegios

- `linpeas`, `pspy`, `winPEAS` para descubrir posibles vectores de ataque

- [GTFOBins](https://gtfobins.github.io/) es una lista de binarios en sistemas Unix que pueden ser utilizados para ejecutar comandos con privilegios elevados. Permite buscar binarios susceptibles de ser utilizados para escalada de privilegios. 


- [PowerUp](https://github.com/PowerShellMafia/PowerSploit) es una herramienta de escalada de privilegios específica para sistemas de Windows.

---

**Referencias**:
- [OWASP - Escalada de Privilegios](https://owasp.org/)
- [CVE-2016-5195 - Dirty COW](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5195)



