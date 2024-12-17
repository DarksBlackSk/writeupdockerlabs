## Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-17 10:38 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 aa:df:30:8b:17:c5:3c:80:1c:88:f1:f8:c0:ac:cc:fa (ECDSA)
|_  256 aa:6a:33:65:fc:54:b7:8f:98:ff:1f:3d:79:a3:05:3c (ED25519)
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-12-17T13:38:33
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.09 seconds
```

observaos 2 servicios levantados, `ssh` y `smb`, por lo que comienzo con `smb`

```ruby
enum4linux-ng 172.17.0.2
```
```ruby
ENUM4LINUX - next generation (v1.3.4)

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 172.17.0.2
[*] Username ......... ''
[*] Random Username .. 'wmpkjecr'
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
NetBIOS computer name: DF1AC7002957
NetBIOS domain name: ''
DNS domain: ''
FQDN: df1ac7002957
Derived membership: workgroup member
Derived domain: unknown

 =======================================
|    RPC Session Check on 172.17.0.2    |
 =======================================
[*] Check for null session
[+] Server allows session using username '', password ''
[*] Check for random user
[+] Server allows session using username 'wmpkjecr', password ''
[H] Rerunning enumeration with user 'wmpkjecr' might give more results

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
Server type string: Wk Sv PrQ Unx NT SNT df1ac7002957 server (Samba, Ubuntu)

 ===================================
|    Users via RPC on 172.17.0.2    |
 ===================================
[*] Enumerating users via 'querydispinfo'
[+] Found 0 user(s) via 'querydispinfo'
[*] Enumerating users via 'enumdomusers'
[+] Found 0 user(s) via 'enumdomusers'

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
  comment: IPC Service (df1ac7002957 server (Samba, Ubuntu))
  type: IPC
darkshare:
  comment: ''
  type: Disk
print$:
  comment: Printer Drivers
  type: Disk
[*] Testing share IPC$
[+] Mapping: OK, Listing: NOT SUPPORTED
[*] Testing share darkshare
[+] Mapping: OK, Listing: OK
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

Completed after 5.76 seconds
```
en principio no observo enumeracion de usuarios, lo que si observo es el directorio `darkshare` en los recursos compartidos

![image](https://github.com/user-attachments/assets/1b1af0ed-18d1-44a0-b109-9b85a9ca61f0)

asi que intento acceder a dichos recursos compartidos

```ruby
smbclient //172.17.0.2/darkshare
```

![image](https://github.com/user-attachments/assets/b26e5f22-3f50-4511-a71d-598db07d749a)

me descargo todos los archivos a mi maquina

```ruby
get drugs.txt
get ilegal.txt
get archivesDatabases.txt
get credentials.txt
```

ahora chequeamos que informacion contienen

![image](https://github.com/user-attachments/assets/6b0e2f59-afde-4be0-af1f-767fac7b7610)

![image](https://github.com/user-attachments/assets/ad4a0ca3-1b46-495b-a115-93195064e6da)

aqui puede que `Top Hackers` sea un user, continuo

![image](https://github.com/user-attachments/assets/7e229148-982c-42bb-8b20-dad6351442f7)

![image](https://github.com/user-attachments/assets/7b464db6-2f01-454b-a538-e9da3fe4755c)

![image](https://github.com/user-attachments/assets/2ace6e62-2bda-47d5-8124-97ad80303196)

la pista de que use 5 me hace pensar que se trata de un mensaje con cifrado cesar, asi que chequeo a ver que resulta

![image](https://github.com/user-attachments/assets/ef8363e5-7ae8-4a2c-ad3f-e6ea6fecccf6)

en efecto se trataba de un mensaje con cifrado cesar y un desplazamiento de 5, aqui nos estan dejando una url en la `darkweb` asi que accedo a dicho dominio

```ruby
nordvpn c Onion_Over_VPN
```

en este caso estoy haciendo uso de un servidor VPN `Onion_Over_VPN` de `nordvpn`para ser exactos, este tiepo de servidores me permite conectarme primero
a un servidor de `nordvpn` y luego conectarme a la red `tor`, esto con el fin de obtener un grado mas de seguridad en la `darkweb`, ademas que me facilita 
el acceso a la `darkweb` a traves de cualquier navegador, no requiero descargar ningun navegador especial ni nada por el estilo


![image](https://github.com/user-attachments/assets/5e26beb1-19cd-4fad-8531-e78000e6a378)

un detalle importante es que este servicio es de paga, asi que en caso de no contar con este tipo de servicios pueden hacer uso de `proxychains` + `firefox` o descargar
un navegar especializado para acceder a la `darkweb`... continuamos


![image](https://github.com/user-attachments/assets/24618f11-d37c-46d5-8eaa-a95f1d353e3e)

por lo visto solo es funcional `Access the Darkest Web`

![image](https://github.com/user-attachments/assets/5dcf15f3-03a7-4406-beac-82b5cc2516fc)

el buscador no funciona asi que comeinzo a chequear las diferentes opciones y el primer enlace es funcional `Hidden Marketplace`

![image](https://github.com/user-attachments/assets/2213c1a4-f985-4cdb-affd-74ef26acd1a8)

aqui si bajamos un poco mas nos conseguimos con un `Confidential List's Passwords`

![image](https://github.com/user-attachments/assets/f7094a72-ed17-40a2-ac08-53c84343ff29)

que al acceder a este obtengo una wordlist de password, asi que me las copio

![image](https://github.com/user-attachments/assets/7b14c547-f758-44da-a086-70ade4daedc8)





