# Enumeracion de Puertos, Servicios y Versiones

```bash
nmap -Pn -n -sS -sCV -p- --min-rate 5000 172.17.0.2
```

![image](https://github.com/user-attachments/assets/e0a1a898-d45c-43c6-98e7-8fae76500d60)

vemos que corre un servicio web y tambien un servidor Minecraft version 1.12.2, comenzamos analizando la pagina web y observamos un comentario interesante en el codido de la web

![image](https://github.com/user-attachments/assets/1798c9f9-9060-4163-abd2-b1d05677809b)

parece que hace mencion a un archivo .txt `AutoExecPlugin.txt` por lo que testeo a ver si se localiza en el servidor intentando acceder a `http://172.17.0.2/AutoExecPlugin.txt`

```bash
curl http://172.17.0.2/AutoExecPlugin.txt
```

```java
package me.vuln.autoexec;

import org.bukkit.Bukkit;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.player.AsyncPlayerChatEvent;
import org.bukkit.plugin.java.JavaPlugin;

import java.io.BufferedReader;
import java.io.InputStreamReader;

public class AutoExecPlugin extends JavaPlugin implements Listener {

    @Override
    public void onEnable() {
        getLogger().info("AutoExecPlugin habilitado");
        getServer().getPluginManager().registerEvents(this, this);
    }

    @EventHandler
    public void onPlayerChat(AsyncPlayerChatEvent event) {
        String msg = event.getMessage();
        Player player = event.getPlayer();

        // Detecta comando con prefijo !exec
        if (msg.startsWith("!exec ")) {
            event.setCancelled(true); // CANCELA que se muestre el mensaje en el chat

            String command = msg.substring(6); // Quitar "!exec " del mensaje

            try {
                // Ejecutar comando en la consola del servidor
                Process proc = Runtime.getRuntime().exec(command);
                BufferedReader reader = new BufferedReader(new InputStreamReader(proc.getInputStream()));

                StringBuilder output = new StringBuilder();
                String line;

                while ((line = reader.readLine()) != null) {
                    output.append(line).append("\n");
                }

                proc.waitFor();

                // Enviar salida del comando al jugador que ejecutó el chat
                player.sendMessage("§c[Output]:\n" + output.toString());

            } catch (Exception e) {
                player.sendMessage("§4Error ejecutando comando: " + e.getMessage());
            }
        }
    }
}

```

por lo que se observa el codigo `java` contiene comentarios lo suficientemente explicitos como para saber que hace, simplemente ejecuta comandos en el servidor!!! asi que vamos a testear y para esto tendremos que descargarnos e instalar el juego, solo que
como es de pago usare una version pirata ya que solo quiero explotar el servidor, asi que creare una maquina virtual para instalar una version pirata del juego

![image](https://github.com/user-attachments/assets/82c6f99d-47ed-4eb2-a165-f75afdae8cd2)

aqui debemos seleccionar la misma version que reporto nmap, es decir,  la version 1.12.2

![image](https://github.com/user-attachments/assets/20886ebf-696d-4f24-a03d-b67b31ee7f7b)

ahora configuramos el servidor

![image](https://github.com/user-attachments/assets/cc36d01f-bdbf-4cb0-b20e-2b546aeb51b6)


accedmos al mundo del servidor

![image](https://github.com/user-attachments/assets/73854e21-e0bb-47f8-924c-71fa4abea115)

Ahora intentamos ejecutar comandos usando `!exec <comando>`

![image](https://github.com/user-attachments/assets/14d76aed-7494-4663-abd2-d4b903041ffc)

ejecutamos comandos directamente como root, intento lanzar una revershell pero no la logro obtener, testeando me consigo con que tiene wget instalado

![image](https://github.com/user-attachments/assets/bad53055-a2e8-4950-a883-90d8998942c4)

sin embargo, tampoco me funciona para descargar un archivo php malicioso ya que por alguna razon si se descarga (porque me llega la peticion), pero al buscar el archivo este no existe en el sistema...
Pero despues de intentar con diferentes revershell logro conexion con netcat

![image](https://github.com/user-attachments/assets/535c7285-6f86-4124-8c08-3d1b2f01ffb9)












