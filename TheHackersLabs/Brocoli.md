---
Estado: Completado
Plataforma: TheHackersLabs
SO: Linux
Dificultad: Principiante
VectorInicial: Webshell expuesta en `uploads`
ServicioInicial: HTTP
PuertoInicial: 80
Credenciales:
  - brocoli: megustalafruta
Usuarios:
  - www-data
  - brocoli
  - brocolon
  - root
Privesc:
  - sudo find
  - sudo java
Tecnicas:
  - Network Discovery
  - Directory Enumeration
  - Directory Listing
  - Webshell Discovery
  - Command Execution
  - Reverse Shell
  - Credential Discovery
  - User Pivoting
  - Privilege Escalation
Herramientas:
  - arp-scan
  - gomap
  - gobuster
  - hashcat
  - hydra
  - penelope
Fecha: 2026-03-10
---

# Brocoli

## Reconocimiento inicial

Se identifica la IP mediante `arp-scan` y después se enumeran puertos con `gomap`.

Servicios relevantes:

- `22/tcp` → SSH
- `80/tcp` → HTTP

## Enumeración web

La web principal solo muestra la página por defecto de Apache. Se enumeran directorios:

```bash
gobuster dir -u http://$TARGET/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x html,php,txt
```

Aparece la ruta `/uploads/`, con `directory listing` activado. Dentro se encuentran dos archivos interesantes:

- `brocoli.php`
- `informebrocoli.txt`

## Webshell y credenciales

El archivo `informebrocoli.txt` contiene un hash MD5 que, tras romperse con `hashcat`, da el valor `mecmec`, aunque esa contraseña no resulta útil para SSH.

El archivo `brocoli.php` actúa como webshell y permite ejecutar comandos mediante el parámetro `cmd`.

Ejemplo:

```text
http://10.0.11.15/uploads/brocoli.php?cmd=cat+/etc/passwd
```

Esto revela usuarios interesantes como `brocoli` y `brocolon`.

## Reverse shell y movimiento lateral

Como las credenciales iniciales no funcionan para SSH, se lanza una reverse shell desde la propia webshell y se captura con `penelope`.

Una vez dentro como `www-data`, se localiza un archivo en `/opt/credenciales.txt` que contiene:

- `brocoli:megustalafruta`

Con esa contraseña se hace `su brocoli`.

## Pivoting a `brocolon`

Revisando sudoers para `brocoli`:

```bash
sudo -l
```

Salida relevante:

```text
(brocolon) NOPASSWD: /usr/bin/find
```

Escalada lateral:

```bash
sudo -u brocolon find . -exec /bin/sh \; -quit
```

## Escalada final a root

Como `brocolon`, se revisan permisos sudo:

```bash
sudo -l
```

Salida:

```text
(ALL : ALL) NOPASSWD: /usr/bin/java
```

Se crea un archivo Java que lanza `/bin/bash`, y se ejecuta con `sudo java`, aprovechando la capacidad de ejecución directa de la versión instalada.

Resultado: shell como `root`.

## Conclusión

Cadena de ataque:

1. Descubrimiento de webshell expuesta.
2. Obtención de reverse shell como `www-data`.
3. Hallazgo de credenciales en `/opt/credenciales.txt`.
4. Pivoting de `brocoli` a `brocolon` con `find`.
5. Escalada final a `root` mediante `sudo java`.

Laboratorio simpático porque combina mala exposición web, credenciales mal guardadas y varios saltos de privilegio hasta root.
