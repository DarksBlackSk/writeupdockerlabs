### Enumeracion de Puertos, Servisios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 2000 172.17.0.2 -oN nmap
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-28 22:54 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 20:c2:39:55:4e:1e:5f:4e:1d:e7:f1:17:39:1a:4e:7a (ECDSA)
|_  256 a5:b5:c0:f2:ce:3e:1a:e4:eb:f8:c3:dd:b5:c6:f3:b3 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: P\xC3\xA1gina Web Profesional
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.45 seconds
```

observo un servicio web y ssh, asi que chequeo el servicio web

![image](https://github.com/user-attachments/assets/26b4c37c-9a19-46ce-9ce3-8490f086a248)

![image](https://github.com/user-attachments/assets/76be943c-e967-4a21-af52-12e50ea922fe)


observo por aca lo que podria ser un dominio, asi que lo agrego al archivo `/etc/hosts`

```ruby
echo '172.17.0.2  cracker.dl' >> /etc/hosts
```
ahora si accedo al dominio observo otro sitio web

![image](https://github.com/user-attachments/assets/ebaede91-d86e-4792-bb9e-27a094375245)

esta web parece que aun esta en construccion ya que nada funciona, asi que hare fuzzing a ver si consigo algo

![image](https://github.com/user-attachments/assets/e7acff2f-cb44-4d50-9483-35450783c6ff)

no veo nada asi que tirare de subdominios a ver

![image](https://github.com/user-attachments/assets/43a97337-cd32-4bce-9a10-6b5186e65ab6)

pruebo `japan` ya que es el unico con codigo de estado `200`; no sin antes agregarlo en `/etc/hosts`

asi nos quedaria en `/etc/hosts`
```
172.17.0.2  cracker.dl japan.cracker.dl
```

![image](https://github.com/user-attachments/assets/0b5a6008-848a-40b4-9153-f179907ce885)

veo que puedo descargar un `SOFTWARE` asi que lo descargo

![image](https://github.com/user-attachments/assets/3928d07e-fd06-44e2-bb06-4653f805994e)

le doy permisos de ejecucion

```ruby
chmod +x PanelAdmin
```
ejecuto el binario y observo lo siguiente

![image](https://github.com/user-attachments/assets/b36af7f8-3aac-4115-b429-14907ea9f321)

solicita un serial para el acceso asi que vere que saco de hacerle `reversing al binario` asi que ejecuto `ghidra`

![image](https://github.com/user-attachments/assets/7b0c8969-fd0d-40ed-83f6-60f03c95b12b)

![image](https://github.com/user-attachments/assets/103c2c04-aac4-421e-8ca4-d2c82b563a7b)

ya en este punto comienzo examinar el codigo decompilado y las funciones encontrando el serial 

![image](https://github.com/user-attachments/assets/0b092d34-5359-431c-a12d-88d1a27942c5)

asi que vuelvo ejecutar el binario y le paso el serial

![image](https://github.com/user-attachments/assets/037ca74f-0082-4d87-ba28-f127a7ee9c4e)

![image](https://github.com/user-attachments/assets/8f7f9285-1bfd-45f1-9192-087329cd5fc4)

![image](https://github.com/user-attachments/assets/79d08d3f-4215-4e26-86bd-93eb8ecf5dfc)

obtengo entonces la contrasena, sin embago no tengo un user con el cual loguearme via `ssh` por lo que aplicare un ataque de diccionario contra el servicio

```ruby
hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p '#P@$$w0rd!%#S€c7T' ssh://172.17.0.2
```
![image](https://github.com/user-attachments/assets/5eed46ce-e437-4393-956e-b2b8e633900d)

lo que observo son errores por alguna razon, asi que usare `crackmapexec`

```ruby
crackmapexec ssh 172.17.0.2 -u /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p '#P@$$w0rd!%#S€c7T' |grep -v 'Authentication failed.'
```

![image](https://github.com/user-attachments/assets/52b2dbbc-cdfd-4e26-9ea1-2c3fd920ca68)

accedo via `ssh`

### Escalada de Privilegios

### cracker

aqui enumere todo el sistema, busque en el ultimo archivo y no daba con nada hasta que probe el serial del binario como password y logre escalar!

```ruby
su
pass: 47378-10239-84236-54367-83291-78354
```

![image](https://github.com/user-attachments/assets/59702908-a3a0-4c5a-b28a-2b12aedd4578)


### PWNED - ROOT








