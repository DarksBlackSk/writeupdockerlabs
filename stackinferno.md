# Enumeracion de Puertos, Servicios y Versiones

```bash
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2 -oN report_nmap
```
![image](https://github.com/user-attachments/assets/33ce0c87-787c-479e-922d-da5ce4c6da78)

Se detectan 2 servicios expuestos, ssh (22) y http (80), tambien se observa una redireccion hacia el dominio `http://cybersec.dl`, agregamos el dominio a nuestro archivo hosts

```bash
echo "172.17.02 cybersec.dl" >> /etc/hosts
```

ahora chequeamos el website

https://github.com/user-attachments/assets/d12733d5-0386-49b3-8212-f0820d67b4fc

intentando acceder al codigo fuente observamos que nos limitan 

![image](https://github.com/user-attachments/assets/d51c537a-93a9-4b81-9b0b-e300098d16c9)

![image](https://github.com/user-attachments/assets/c37ff475-ba27-4281-a4c2-0e7362e73d16)

pero esta limitacion es facil de saltarse desactivando en el navegador el codigo js o con curl..

```bash
curl http://cybersec.dl
```

![image](https://github.com/user-attachments/assets/d4c69010-e9ab-4931-ac9f-3aa212526d29)

inspeccionando el codigo fuente puedo observar el uso de una api que se encarga de generar las credenciales que recomiendan usar en el website

![image](https://github.com/user-attachments/assets/fddd9d32-5771-4486-b1f7-5d0dbfe4bdd6)

![image](https://github.com/user-attachments/assets/da7962dc-04f4-4c69-9289-507563dcc88b)

solo genera las credenciales, sin embargo podemos intentar localizar mas puntos finales con fuzzing

```bash
feroxbuster -u http://cybersec.dl/api/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -t 200 --random-agent --no-state -d 5
```

![image](https://github.com/user-attachments/assets/006eb65c-ca62-419f-942a-45cbe5c9529b)

conseguimos un punto final mas, asi que lo testeamos

![image](https://github.com/user-attachments/assets/2ed55be0-9649-4d78-80d4-e9b9ee576d28)

esto nos indica que espera un tipo de peticion diferente, asi que interceptare la peticion con `caido` y asi modificarla. Primero configuramos el navegador para que pase
a traves de `caido`

![image](https://github.com/user-attachments/assets/2d0272af-39d7-47ec-a89d-96fc34396c82)

luego abrimos `caido` e interceptamos la peticion

![image](https://github.com/user-attachments/assets/692694a8-f007-46b1-95eb-6c8999988270)

pasamos la peticion de GET a POST 

![image](https://github.com/user-attachments/assets/e35dc18a-8b8e-4133-bb90-d15f82800dc4)

enviamos la peticion y observamos la respuesta en el navegador

![image](https://github.com/user-attachments/assets/274a4e0e-8f5c-49c1-bf57-3221b9c3df2e)

ahora observamos que pide la cabecera `Role` asi que volvemos interceptar la peticion y la mandamos al `automate` para testear los roles y para esto voy a preparar
una wordlist con posibles roles y luego automatizar la prueba con caido

https://github.com/user-attachments/assets/d1055a11-67d7-4a65-ad4b-bf5526b92a91

hemos logrado acceder a informacion de posible interes con el rol administrador, todos los subdominios que aqui consigo los agrego a mi archivo hosts y los testeo

https://github.com/user-attachments/assets/e5aa40fe-b3d1-496f-a265-57688c33bbdf

tenemos 3 subdominios activos, pero en los subdominios `http://mail.cybersec.dl/` & `http://soc_internal_operations.cybersec.dl/` despues de testearlos no tienes vulnerabilidades o informacion de interes
sin embargo el tercer subdominio `http://0internal_down.cybersec.dl/` vemos que indica falta la cabecera `X-UUID-Access` y si recordamos la informaacion extraida de /api/interest
vemos algo que puede estar relacionado con esta cabecera

![image](https://github.com/user-attachments/assets/3ff4768f-051b-48c4-8917-25390bdad67b)

asi que intercecto la peticion con `CAIDO`, le agrego la cabecera `X-UUID-Access` y pasandole los UUID de /api/interest

https://github.com/user-attachments/assets/2592b325-d56d-4be4-9320-d4d036255ed5
















