---
layout: single
title: Escalada de privilegio
date: 2025-02-25
classes: wide
header:
  teaser: /assets/images/slae32.png
categories:
  - slae
  - infosec
tags:
  - escalada de privilegio
  - privilegio
  - linux
  - windows
---
La **escalada de privilegios** es el proceso mediante el cual un atacante obtiene un nivel de acceso mayor al que originalmente tenía en un sistema. Se divide en dos categorías principales:  

## Tipos de Escalada de Privilegios  

### 1. Escalada de Privilegios Horizontal  
- Ocurre cuando un usuario sin privilegios administrativos accede a los recursos o permisos de otro usuario con el mismo nivel de privilegios.  
- **Ejemplo:** Un atacante con una cuenta estándar accede a los archivos o configuraciones de otro usuario sin ser administrador.  

### 2. Escalada de Privilegios Vertical  
- Ocurre cuando un atacante con privilegios bajos (como un usuario normal) consigue acceso a una cuenta con mayores privilegios (como `root` o `Administrador`).  
- **Ejemplo:** Un atacante explota una vulnerabilidad en un programa para obtener acceso como superusuario en Linux o administrador en Windows.  

## Métodos Comunes de Escalada de Privilegios  

1. **Explotación de vulnerabilidades en software**  
   - Fallos en el kernel  
   - Binarios con `SUID` mal configurados  
   - Exploits de día cero  

2. **Errores en configuraciones**  
   - Permisos mal asignados en archivos críticos  
   - Credenciales almacenadas en texto plano  

3. **Ataques de fuerza bruta**  
   - Intentar contraseñas comunes para usuarios privilegiados  

4. **Uso de herramientas de enumeración**  
   - `linpeas`, `pspy`, `winPEAS` para descubrir posibles vectores de ataque  

## Herramientas Útiles  

- **Linux:** [`GTFOBins`](https://gtfobins.github.io/)  
- **Windows:** [`PowerUp`](https://github.com/PowerShellMafia/PowerSploit)
