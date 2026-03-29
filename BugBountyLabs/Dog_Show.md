---
Estado: Completado
Plataforma: Bug Bounty Labs
SO: Linux
Dificultad: Fácil
VectorInicial: XSS reflejado y almacenado
Privesc: No
Fecha: 2026-02-23
---

# Dog Show

## Reconocimiento inicial

Se comienza enumerando puertos para identificar servicios expuestos.

```bash
settarget 172.17.0.2
gomap -s $TARGET
```

Servicios detectados:

- `80/tcp` → Apache
- `5000/tcp` → aplicación Python/Werkzeug

## Análisis del puerto 80

La web principal muestra la página por defecto de Apache. Se enumeran directorios:

```bash
gobuster dir -u http://$TARGET/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```

Aparece el directorio `/beta/`, que contiene un formulario vulnerable.

### Prueba inicial de XSS

```javascript
<script>alert(1)</script>
```

No se ejecuta directamente, pero al inspeccionar el código fuente se observa que la entrada acaba reflejada en un atributo HTML, por lo que es posible romper el contexto.

Payload funcional:

```javascript
" onmouseover=confirm(1) "
```

Con ello se confirma un XSS reflejado.

## Análisis del puerto 5000

En `5000` se encuentra una aplicación relacionada con mascotas. Tras registrarse y revisar el código HTML generado, se observa que varios campos se renderizan sin escape, usando contenido inseguro.

Esto sugiere claramente la posibilidad de XSS almacenado.

Ejemplo de pista visible en la propia aplicación:

```html
<p>Name: pupy</p>
<p>Age: 12</p>
<p>Breed: 123</p>
```

El comportamiento de renderizado indica que los datos proporcionados por el usuario pueden acabar ejecutándose si contienen HTML o JavaScript.

## Conclusión

Este laboratorio resulta interesante porque permite detectar dos escenarios distintos:

1. un XSS reflejado en la zona `/beta/`
2. un XSS almacenado en la aplicación del puerto `5000`

Más que un reto de explotación compleja, es una práctica útil para identificar contextos de inyección y analizar cómo se está renderizando realmente la entrada del usuario.

---
Si te gustó, puedes invitarme a un café.
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/C0C61UHTB1)

