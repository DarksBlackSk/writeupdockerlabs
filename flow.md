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

![image](https://github.com/user-attachments/assets/134c810e-bc2b-4022-97e2-c93cabae60d4)

continuo testeando pero esta como raro

![image](https://github.com/user-attachments/assets/70912cba-395b-4aa2-a1a4-afcf77bd1e56)

dice que la clave sera `root` para acceder al modo administrador, pero si coloco `root` no funciona asi que lo enviare a mi maquina para examinarlo mejor

![image](https://github.com/user-attachments/assets/2ed714bd-612e-4482-afb2-b120dca82f0c)

![image](https://github.com/user-attachments/assets/dd089e0e-1500-4099-b88e-484f2bfe6b63)

es un binario `x64` que no cuenta con 2 protecciones asi que chequeare si llega ser vulnerable a un desbordamiento de buffer

![image](https://github.com/user-attachments/assets/65904269-4d5b-46dd-8e5a-0387ec1ab2eb)

ocurrio un fallo de segmentacion, lo cual podria ser un indicio de un posible desbordamiento de buffer

## BUFFER OVERFLOW

comienzo desactivando el ASLR y ejecutando el binario a traves del depurador `gdb`

```ruby
sysctl -w kernel.randomize_va_space=0 # Desactivando temporalmente el ASLR
```

# Writeup en construccion, conuare ...














