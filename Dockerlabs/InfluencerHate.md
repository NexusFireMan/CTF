---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Fácil+
VectorInicial: Basic Auth con credenciales por defecto -> login bruteforce -> SSH
Privesc: Reutilización de credenciales + brute force local con `su`
Fecha: 2026-02-26
---

# InfluencerHate

## Reconocimiento inicial

Se enumeran puertos y servicios:

```bash
settarget 172.17.0.2
gomap -s $TARGET
```

Servicios detectados:

- `22/tcp` → SSH
- `80/tcp` → HTTP

## Basic Auth en la web

Al acceder a la web aparece autenticación básica HTTP. Se prueban credenciales por defecto con un script propio o con `hydra`.

### Opción 1: script específico

```bash
python3 basic_auth_tester.py http://$TARGET -w /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt
```

### Opción 2: `hydra`

```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt http-get://$TARGET
```

Credenciales válidas encontradas:

- `httpadmin:fhttpadmin`

## Enumeración adicional del sitio

Con las credenciales de Basic Auth se enumeran rutas:

```bash
gobuster dir -u http://$TARGET -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x html,php,txt,bak -U httpadmin -P fhttpadmin
```

Se detecta un `login.php`, así que se analiza la autenticación con Burp y se prepara fuerza bruta del formulario.

## Fuerza bruta del login web

```bash
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/wordlists/300-rockyou.txt 172.17.0.2 http-post-form "/login.php:username=^USER^&password=^PASS^:H=Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=:F=Credenciales incorrectas."
```

Credencial obtenida:

- `admin:chocolate`

Al acceder, aparece una referencia al usuario `balutin`.

## Acceso inicial por SSH

Se prueba fuerza bruta sobre SSH para `balutin`:

```bash
hydra -l balutin -P /usr/share/wordlists/rockyou.txt $TARGET -t 5 ssh
```

Credencial encontrada:

- `balutin:estrella`

Con esto se consigue acceso por SSH.

## Escalada de privilegios

No hay `sudo`, y los binarios SUID no ofrecen una vía clara inmediata. Se recurre entonces a enumeración adicional con `linpeas`, que revela una base de datos con credenciales reutilizadas.

Más adelante, al no funcionar directamente `su root`, se usa un script de brute force local contra `su`.

Tras probar contraseñas, se obtiene:

- `root:rockyou`

Y se accede como `root` con `su`.

## Conclusión

Cadena de ataque:

1. Credenciales por defecto en Basic Auth.
2. Enumeración del sitio tras autenticación.
3. Fuerza bruta del login web.
4. Descubrimiento del usuario `balutin`.
5. Fuerza bruta SSH.
6. Enumeración interna y obtención de credenciales reutilizadas.
7. Acceso final como `root`.

Laboratorio curioso porque mezcla varios eslabones débiles pequeños. Ninguno por sí solo parece espectacular, pero juntos revientan la máquina entera.

---
Si te gustó, puedes invitarme a un café.
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/C0C61UHTB1)
