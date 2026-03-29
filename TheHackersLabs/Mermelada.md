---
Estado: Completado
Plataforma: TheHackersLabs
SO: Linux
Dificultad: Principiante
VectorInicial: Webshell descubierta en uploads de WordPress
ServicioInicial: HTTP
PuertoInicial: 80
Credenciales:
  - debian: 12345
  - mermeladita: pepitU
  - mysql root: 12345
Usuarios:
  - debian
  - mermeladita
  - root
Privesc: sudo find
Tecnicas:
  - Network Discovery
  - Directory Enumeration
  - WordPress Enumeration
  - Webshell Discovery
  - Command Execution
  - User Enumeration
  - Password Brute Force
  - Credential Reuse
  - Database Enumeration
  - Privilege Escalation
Herramientas:
  - arp-scan
  - gomap
  - gobuster
  - wpscan
  - hydra
  - mysql
Fecha: 2026-03-05
---

# Mermelada

## Reconocimiento inicial

Se detecta la IP del host y se enumeran puertos:

- `22/tcp` → SSH
- `80/tcp` → HTTP

## Enumeración web

La web principal incluye un parámetro `zona` en URL, pero antes de profundizar ahí se enumeran directorios:

```bash
gobuster dir -u http://$TARGET/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x html,php,txt
```

Se descubren:

- `login.php`
- `uploads/`
- `wordpress/`

La instalación de WordPress requiere resolver el dominio `mermelada.thl`, que se añade a `/etc/hosts`.

## Enumeración de WordPress

Con `wpscan` se detectan:

- usuario: `mermeladita`
- plugin vulnerable: `wpdiscuz`
- `directory listing` activado en `wp-content/uploads`

Mientras se prueban credenciales sobre WordPress, se inspecciona manualmente la carpeta de uploads y se encuentran varios archivos `.php` con apariencia de imagen falsa y parámetros de ejecución por `cmd`.

Esto permite ejecutar comandos directamente y leer `/etc/passwd`, revelando usuarios como `debian` y `mermeladita`.

## Acceso inicial

Se lanza fuerza bruta SSH y se obtiene:

- `debian:12345`

Se accede por SSH con esa cuenta.

## Enumeración local

Sin privilegios sudo útiles para `debian`, se continúa explorando el sistema y aparece un archivo interesante:

```text
/opt/.credenciales
```

Este archivo contiene datos de base de datos. Después se revisa `wp-config.php` y se obtiene:

- `DB_USER=root`
- `DB_PASSWORD=12345`

Con esas credenciales se accede a MariaDB.

## Movimiento lateral

Dentro de la base de datos `mermelada`, la tabla `users` contiene:

- `mermeladita:pepitU`

Con ello se cambia al usuario `mermeladita` usando `su`.

## Escalada final

Permisos sudo de `mermeladita`:

```bash
sudo -l
```

Salida relevante:

```text
(ALL : ALL) NOPASSWD: /usr/bin/find
```

Escalada a root:

```bash
sudo find . -exec /bin/sh \; -quit
```

## Conclusión

Cadena de ataque:

1. Enumeración web y WordPress.
2. Descubrimiento manual de webshells en uploads.
3. Lectura de `/etc/passwd`.
4. Fuerza bruta SSH sobre `debian`.
5. Enumeración de base de datos y reutilización de credenciales.
6. Pivoting a `mermeladita`.
7. Escalada a root con `sudo find`.

Muy buen ejemplo de por qué dejar archivos ejecutables en uploads es directamente pedir que te revienten el sistema.
