---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Fácil
VectorInicial: WordPress login + edición de plugin para RCE
Privesc: SUID `env`
Fecha: 2026-02-16
---

# WalkingCMS

## Reconocimiento inicial

Se comienza con enumeración de la superficie:

```bash
settarget 172.17.0.2
gomap -s $TARGET
```

Solo aparece el puerto `80` con Apache.

## Descubrimiento de WordPress

La raíz muestra la página por defecto de Apache, así que se enumeran rutas:

```bash
dirb http://$TARGET
```

Aparece un WordPress en `/wordpress/`.

## Enumeración de WordPress

Visualmente solo se detecta al usuario `mario`, así que se recurre a `wpscan` para afinar:

```bash
wpscan --url http://$TARGET/wordpress/ -e u,p --plugins-detection aggressive
```

Se identifican:

- usuario: `mario`
- plugin: `theme-editor`
- varias vulnerabilidades conocidas del plugin

## Acceso al panel

Antes de intentar explotar el plugin, se busca la contraseña de `mario` con `wpscan`:

```bash
wpscan --url http://$TARGET/wordpress/ -U mario --passwords /usr/share/wordlists/rockyou.txt
```

Credencial válida:

- `mario:love`

Con ello se accede al panel de WordPress.

## Ejecución remota de código

Aunque el plugin `theme-editor` tiene vulnerabilidades conocidas, en este caso no hace falta explotarlas tal cual: al tener acceso al panel, basta con editar código del plugin e insertar una reverse shell en PHP.

Se deja `penelope` escuchando en el puerto `443`, se guarda el código malicioso y se activa la ejecución desde el plugin.

Resultado: shell como `www-data`.

## Escalada de privilegios

Se revisan binarios SUID:

```bash
find / -perm -4000 2>/dev/null
```

Aparece `env` como SUID, lo que permite escalar directamente:

```bash
env /bin/sh -p
```

Con esto se obtiene acceso como `root`.

## Conclusión

Cadena de ataque:

1. Descubrimiento de WordPress.
2. Enumeración de usuario y plugin.
3. Fuerza bruta de credenciales del panel.
4. Inserción de reverse shell en plugin editable.
5. Escalada mediante SUID en `env`.

Buen ejemplo de cómo unas credenciales flojas y permisos inseguros en el sistema convierten un WordPress mediocre en una invitación al desastre.

---
Si te gustó, puedes invitarme a un café.
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/C0C61UHTB1)
