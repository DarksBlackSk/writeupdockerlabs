# Enumeracion de Puertos, Servicios y Versiones

```bash
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN report_nmap
```

![image](https://github.com/user-attachments/assets/a0c3f36f-1642-424a-9e6e-684bcf029396)


observamos 2 servicios, ftp (puerto 21) y http (puerto 80). Comienzo chequeando el servicio ftp

## Servicio FTP

me conecto al servicio

```bash
ftp 172.17.0.2
```

![image](https://github.com/user-attachments/assets/2afc2433-9298-43d8-8a23-e5ed9faee312)

aqui veo que es posible acceder con el usuario `anonymous` asi que accedemos como dicho usuario

```bash
ftp anonymous@172.17.0.2
```

![image](https://github.com/user-attachments/assets/1e32b65e-92fe-4d04-b528-6a993aada484)

despues de acceder, vemos un archivo llamado `alumno.txt` asi que lo descargamos, salimos y leemos el archivo

![image](https://github.com/user-attachments/assets/2fa33713-4f56-4a8b-ae51-f1a27a1f7294)

tenemos credenciales para el servicio web, asi que vamos por el servicio web a ver de que va esto

## Servicio web

Antes de poder acceder al website debemos agregar a nuestro archivo hosts el dominio reportado en `nmap`

![image](https://github.com/user-attachments/assets/e04d9195-dccc-47b9-97f1-0cd15b13faf1)

```bash
echo "172.17.0.2 chamilo.dl" >> /etc/hosts
```

ahora si accedemos sin problemas al website

![image](https://github.com/user-attachments/assets/5cdde8e1-ec8e-4696-a6d7-0ce8c2f13e83)

aqui accedemos con las credenciales que conseguimos anteriormente

![image](https://github.com/user-attachments/assets/8f4337f3-13a9-41d4-8da5-e9eef2c40156)

Por lo visto este usuario no tiene ninguna clase de privilegios, sin embargo es posible extraer usuarios y dar con el usuario administrador, una vez logueados
accedemos en `http://chamilo.dl/main/social/profile.php?u=1` y conseguimos el usuario administrador

![image](https://github.com/user-attachments/assets/2423379a-1604-48fb-a388-2c5e29ef4305)

incluso tambien esta en el panel de login en la parte inferior (footer)

![image](https://github.com/user-attachments/assets/b6c0af35-39c9-4550-8733-1e26814c8c33)


continuara...























