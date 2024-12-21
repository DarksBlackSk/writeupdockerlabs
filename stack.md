### Enumeracion de Puertos, Servicios y Versiones

comienzo realizando un escaneo de puertos y servicios con `nmap`

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 2000 172.17.0.2 -oN nmap
```

```ruby
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-21 12:16 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 85:7f:49:c5:89:f6:ce:d2:b3:92:f1:40:de:e0:56:c4 (ECDSA)
|_  256 6d:ed:59:b8:d8:cc:50:54:9d:37:65:58:f5:3f:52:e3 (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: Web en producci\xC3\xB3n
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.10 seconds
```

aqui observo 2 servicios, el puerto 80 (web) y el puerto 22 (ssh); asi que comienzo chequeando el servicio web

![image](https://github.com/user-attachments/assets/a156f80b-c443-48fc-9769-92292a92348a)

al chequear el codigo fuente observo un comentario peculiar

![image](https://github.com/user-attachments/assets/60f154c6-dffb-44c7-9c2b-614d880ec790)

al observar esto, lo primero que intente fue aplicar un ataque de diccionario contra el servicio `ssh` con el user `bob` pero no obtuve resultados asi que 
realice fuzzing web

```ruby
feroxbuster -u 'http://172.17.0.2' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 200 --random-agent --no-state -d 10
```

```ruby
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://172.17.0.2
 🚀  Threads               │ 200
 📖  Wordlist              │ /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ Random
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt, php, bak, db, py, html, js, jpg, png, git, sh]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 10
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        9l       28w      275c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        9l       31w      272c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       13l       32w      417c http://172.17.0.2/
200      GET        0l        0w        0c http://172.17.0.2/file.php
200      GET       13l       32w      417c http://172.17.0.2/index.html
301      GET        9l       28w      313c http://172.17.0.2/javascript => http://172.17.0.2/javascript/
200      GET        1l       17w      110c http://172.17.0.2/note.txt
301      GET        9l       28w      320c http://172.17.0.2/javascript/jquery => http://172.17.0.2/javascript/jquery/
200      GET    10907l    44549w   289782c http://172.17.0.2/javascript/jquery/jquery
200      GET    10907l    44549w   289782c http://172.17.0.2/javascript/jquery/jquery.js
[#############>------] - 86s  5068323/7474788 52s     found:8       errors:0      
[####################] - 68s  2491548/2491548 36674/s http://172.17.0.2/ 
[#############>------] - 86s  1673460/2491548 19461/s http://172.17.0.2/javascript/ 
[#######>------------] - 49s   901128/2491548 18421/s http://172.17.0.2/javascript/jquery/
```

si accedo en `http://172.17.0.2/note.txt` observo el siguiente texto

![image](https://github.com/user-attachments/assets/af09b911-98a8-419c-bae2-3d87eda2203b)

asi que ahora chequeo en `http://172.17.0.2/file.php` pero en principio la web esta en blanco y en el codigo fuente nada, por lo que testeo el `LFI` a ver si de verdad
fue parcheada... 

![image](https://github.com/user-attachments/assets/d2e9e6ef-47ff-49e6-9e09-34a1aa4f93a2)

En principio no encuentro nada, pero puedo intentar bypassear el `LFI` a ver si obtengo resultados

![image](https://github.com/user-attachments/assets/f0038024-e6d1-4fd2-ab4e-26a0b97aa799)

ahora si obtengo resultados, asi que testeo

![image](https://github.com/user-attachments/assets/34f8f712-60d1-440f-bcf5-754aba17c6c9)

y observamos que en efecto puedo leer archivos del sistema y si recordamos tenemos una ruta a una password por lo que intentare acceder a ella

```ruby
http://172.17.0.2/file.php?file=....//....//....//....//....//....//....//usr//share//bob//password.txt
llv6ox3lx300
```

he obtenido la credencial para el user `bob` por lo que la testeo en el servicio `ssh`

![image](https://github.com/user-attachments/assets/94e2c2b5-4a09-42af-a99a-9778c6d741ef)

obtengo acceso al sistema!!!!

## Escalada de privilegios

### bob

revisando el sistema me consigo con un binario

![image](https://github.com/user-attachments/assets/633a4b48-9e35-493a-8250-12dfcf411ebc)

ejecuto el biario y observo lo siguiente

![image](https://github.com/user-attachments/assets/9b6fb1bf-339c-4398-aa6a-622c85fcdbaa)

asi que me lo envio a mi maquina atacante para analizarlo

![image](https://github.com/user-attachments/assets/812b1336-0af8-4bb9-9d3b-1585043aad8f)

si le hago un `string` no observo informacion de interes por lo que chequeo si llega ser vulnerable a un desbordamiento de buffer

![image](https://github.com/user-attachments/assets/040d7bb4-4792-4466-aa6f-88fd13994eeb)

ocurrio un fallo de segmentacion, por lo que podria estar ante un desbordamiento de buffer, asi que lo primero que hare sera chequear las protecciones del binario y
la arquitectura

![image](https://github.com/user-attachments/assets/1fd63209-7ebe-4884-aacb-75c9c0dc1202)

![image](https://github.com/user-attachments/assets/418f1659-573e-451f-84b9-e7e593325df3)


veo que tiene activa las protecciones `NX` & `PIE`; por un lado no es posible la ejecucion de shellcode debido a la proteccion `NX` activa y por otro lado vemos que 
tendriamos una aleatorizacion en las direcciones de memoria de las funciones por la proteccion `PIE` que se encuentr activa lo que dificultaria un poco mas la explotacion
del buffer overflow, por tanto primero deshabilitare el `ASLR` y continuare con el analisis del binario desde el depurador

```ruby
sysctl -w kernel.randomize_va_space=0 # deshabilito el ASLR
```

ahora ejecuto el binario con el depurador `gdb` para su posterior analisis

```ruby
gdb ./command_exec -q
```

ahora creare una cadena especialmente disenada para el calculo del `offset`, esto lo hago con un exploit de `metasploit`, esto con el fin de pasarlo como dato al binario

![image](https://github.com/user-attachments/assets/61678d3d-5064-4874-9c09-2e2ff468fd5a)

primero creo la cadena de 500 caracteres con el exploit y luego se lo paso al binario cuando solicita la password, para luego obtener la direccion `rip` de retorno la cual
se la paso a un segundo exploit de metasploit para terminar calculando el `offset`

![image](https://github.com/user-attachments/assets/75a08ad4-b16f-430d-a39a-f03ad58d9520)

obtenemos el valor del `offset` con valor de 88 bytes, ahora probare si puedo obtener el control del registro `rip`

![image](https://github.com/user-attachments/assets/d77814b7-38bc-4000-97ab-4013b2beae54)

observo que en efecto tengo control sobre el registro `rip`, pero tambien observo un detalle mas y es que se puede observar que la key tiene valor `41414141`, y si vemos
el mensaje que arroja el binario `key debe valer 0xdead para entrar al modo administrador`, es decir, debo hacer que la `key` valga `0xdead` y por lo que veo actualmente
la key esta siendo sobreescrita con `A` (valor 41 en hexadecimal) asi que ire testeando hasta lograr setear la key con `0xdead`

![image](https://github.com/user-attachments/assets/f720bef7-fdba-4f31-b7e7-0e9e70ac8b5f)

he logrado llegar al modo administrador setenado la `key` con el valor correcto, tambien observo que me solicita un comando pero el programa termina de inmediato
por lo que intentare enviar un comando a ver como responde

![image](https://github.com/user-attachments/assets/598827c0-24ee-4405-9330-08bc22c0e717)

por lo visto a ejecuto mi comando asi que intentare ejecutar otro comando para validar 100%

![image](https://github.com/user-attachments/assets/c6c74967-73d1-4545-b293-157ef7d9fa6b)

he logrado la ejecucion de comandos asi que ahora intentare la escalada de privilegios ejecuando el comando `chmod u+s /bin/bash` en la maquina comprometida ya que
el binario tiene el bit suid activo

![image](https://github.com/user-attachments/assets/1f98184b-15f8-4803-b446-1bbc441f7315)

como la maquina comprometida no tiene instalado `python2` no puedo ejecutar los mismos comandos que en mi maquina atacante por lo que guardo el payload con `echo` y 
guardo tambien en un archivo el comando que le asignara el bit suid a `/bin/bash`, ahora ejecuto el binario `command_exec` pasandole el payload y el comando

![image](https://github.com/user-attachments/assets/dc822ac2-1bd8-42db-a5e3-501df40df016)

logre ejecutar correctamente el comando por lo que ya es posible la escalada

![image](https://github.com/user-attachments/assets/67feba5d-3b9f-49e3-9367-4a535b322248)


### PWNED - Root
