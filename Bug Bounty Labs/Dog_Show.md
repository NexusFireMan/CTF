---
Estado: Completado
Plataforma: Bug Bounty Labs
SO: Linux
Dificultad: Fácil
VectorInicial: XSS
ServicioInicial: HTTP
PuertoInicial: 80
Credenciales:
 - No
Usuarios:
 - No aplica
Privesc: No
Tecnicas:
 - Service Enumeration
 - Reflected XSS
 - Stored XSS
Herramientas:
 - gomap
 - gobuster
 - navegador
Fecha: 2026-02-23
---
<img width="400" height="245" alt="Pasted image 20260223165234" src="https://github.com/user-attachments/assets/ef2cc6db-96dc-4eab-b5f8-9e1da42c4a18" />

Comenzaremos con una enumeración de puertos para hacernos una idea de lo que tenemos.

```bash
 settarget 172.17.0.2
TARGET establecido: 172.17.0.2

 gomap -s $TARGET
🎯 Scanning 172.17.0.2 (997 ports)

PORT    STATE  SERVICE         VERSION
80      open   http            Apache 2.4.62 (Debian)
5000    open   http            Werkzeug/3.1.3 Python/3.11.2

Host Exposure Summary
- 172.17.0.2 | open ports: 2 | critical: none | exposure: low

✓ Completed scan in 45ms | hosts: 1 | open ports: 2
```

Nos encontramos con 2 puertos abiertos:
- 80 - HTTP con servidor apache
- 5000 - HTTP con posible aplicación en Python.

Empecemos visitando la web directamente en el puerto 80 a ver qué hay.

Nos encontramos con la página por defecto de Apache, así que vamos a realizar una enumeración de directorios a ver si encontramos algo.

```bash
 gobuster dir -u http://$TARGET/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
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

Parece que en el directorio `/beta/` tenemos una web.

<img width="995" height="476" alt="Pasted image 20260223160032" src="https://github.com/user-attachments/assets/3deb43f5-8a86-4611-be81-bd501465d569" />

Por lo que vemos, es un formulario con una vulnerabilidad XSS, así que probaremos con lo más básico.

```javascript
<script>alert(1)</script>
```

Pero no ocurre nada, aunque al revisar el código mediante `CTRL+U` vemos que sí está ocurriendo algo:

```html
<p>
	<span title="<script>*****(1)</script>">
        Comentario reflejado
    </span>
</p>
```

Hay un filtro para no mostrar las alertas, así que intentaremos otro método.

```javascript
" onmouseover=confirm(1) "
```

Este payload permite el XSS y, por consiguiente, ya tenemos una vulnerabilidad confirmada. Esta parte finaliza aquí.

Ahora vamos a por el puerto *5000* el cual tiene una web de perritos.

<img width="528" height="487" alt="Pasted image 20260223162627" src="https://github.com/user-attachments/assets/be11ffe8-c1d7-4e33-87c3-67bc598921d8" />

Vamos a registrarnos a ver que podemos encontrar.

Solo tenemos una página para añadir nuestras mascotas; esto da a pensar que puede tratarse de un XSS almacenado.

Curiosamente, al ver el código de la página nos encontramos con esto:

```html
            <li>
                <!-- Aquí se elimina el escape, permitiendo la ejecución de JavaScript si se inyecta código -->
                <img src="http://www.ert.com" alt="pupy" style="width: 450px; height: 300px;">
                <p>Name: pupy</p> <!-- Uso de 'safe' hace que se renderice como HTML sin escape -->
                <p>Age: 12</p> <!-- Si hay algún intento de inyección de código, se ejecutará -->
                <p>Breed: 123</p> <!-- También se permite inyección en la raza -->
            </li>
        
            <li>
                <!-- Aquí se elimina el escape, permitiendo la ejecución de JavaScript si se inyecta código -->
                <img src="http://www.ert.com" alt="pupy" style="width: 450px; height: 300px;">
                <p>Name: pupy</p> <!-- Uso de 'safe' hace que se renderice como HTML sin escape -->
                <p>Age: 12</p> <!-- Si hay algún intento de inyección de código, se ejecutará -->
                <p>Breed: 123</p> <!-- También se permite inyección en la raza -->
            </li>
```

Lo cual ya nos indica dónde podremos inyectar el código.

Esto es un gran fallo, puesto que permite inyecciones sin necesidad de hacer ataques a ciegas, ya que nos está indicando exactamente el lugar donde tenemos que atacar.
