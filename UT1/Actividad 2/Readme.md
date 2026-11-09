##  Resumen de teoría 

* **RDP:** Es un protocolo de Microsoft que permite ver y controlar el ordenador sin estar frente a él. Usa el **puerto 3389** utilizando el protocolo **TCP**.
* **NLA:** El NLA es una capa de seguridad que es obligatoria en los nuevos equipos. Se utiliza en las conexiones de escritorio remoto y obliga a iniciar sesión antes de conectarse a través de escritorio remoto.
* **Quién puede conectarse:** Para poder conectarse a través de RDP, el sistema operativo del equipo destino debe ser **Windows**, hay que estar en la **misma red** y que el usuario con el que te vayas a conectar tenga **permiso de escritorio remoto**.
* **Diferencia con SSH:** La diferencia entre RDP y SSH es que funcionan en **puertos distintos**, RDP es **gráfico** y el SSH es más por **comandos**, y que el **consumo de red es mayor en RDP**.

---

##  Comandos utilizados en la práctica

###  En la máquina Ubuntu (`ubuntu-gui`)

* **Actualizar repositorios e instalar el entorno gráfico:**
  ```bash
  sudo apt update
  sudo apt install xfce4 xfce4-goodies dbus-x11 -y
  ```
  *Explicación:* Actualiza la lista de paquetes disponibles e instala el entorno de escritorio XFCE junto con `dbus-x11` para evitar fallos al arrancar la sesión remota.

* **Añadir el usuario de XRDP al grupo de seguridad:**
  ```bash
  sudo adduser xrdp ssl-cert
  ```
  *Explicación:* Añade el usuario `xrdp` al grupo `ssl-cert` para darle permisos de lectura sobre las claves y certificados de seguridad locales.

* **Comprobar el estado del servicio, puerto e IP:**
  ```bash
  sudo systemctl status xrdp
  sudo ss -tlnp | grep 3389
  ip a
  ```
  *Explicación:* Verifica que el servicio `xrdp` está activo, confirma que la máquina escucha por el puerto 3389 y muestra la dirección IP para poder conectarse a ella.

* **Arreglo para evitar la pantalla negra de la sesión:**
  ```bash
  echo "xfce4-session" > ~/.xsession
  ```
  *Explicación:* Configura de forma explícita el entorno XFCE como el escritorio predeterminado que debe iniciar la sesión remota.

* **Instalar el cliente RDP Remmina:**
  ```bash
  sudo apt update
  sudo apt install remmina remmina-plugin-rdp -y
  ```
  *Explicación:* Instala el programa Remmina junto con su complemento RDP para poder realizar conexiones hacia la máquina Windows.

* **Ver el registro de conexiones de XRDP:**
  ```bash
  sudo tail -n 30 /var/log/xrdp.log
  ```
  *Explicación:* Muestra las últimas 30 líneas del archivo log del servicio para auditar los accesos y comprobar quién se ha conectado.

###  En la máquina Windows

* **Comprobar conectividad y puerto abierto hacia Ubuntu:**
  ```powershell
  ping <ip_de_ubuntu-gui>
  Test-NetConnection <ip_de_ubuntu-gui> -Port 3389
  ```
  *Explicación:* Verifica la conexión de red básica con el comando `ping` y comprueba de forma específica que el puerto RDP de la máquina Ubuntu esté escuchando.

* **Añadir un usuario al grupo de escritorio remoto por su SID:**
  ```powershell
  Add-LocalGroupMember -SID S-1-5-32-555-Member "nombre_usuario"
  Get-LocalGroupMember -SID S-1-5-32-555
  ```
  *Explicación:* Da acceso al grupo de escritorio remoto a un usuario que no es administrador utilizando el identificador universal del grupo, evitando problemas con el idioma del sistema.

* **Habilitar la regla del Firewall para RDP:**
  ```powershell
  Enable-NetFirewallRule -Name "RemoteDesktop-UserMode-In-*"
  ```
  *Explicación:* Activa las reglas del cortafuegos nativo de Windows para permitir las solicitudes de escritorio remoto entrantes.

* **Ver la dirección IP local de Windows:**
  ```cmd
  ipconfig
  ```
  *Explicación:* Muestra la información de los adaptadores de red para poder conocer la dirección IP privada de la máquina Windows.

  ---

##  Reflexión: RDP vs SSH en la administración de servidores

Usaría **RDP** cuando sea obligatorio usar **interfaces gráficas (GUI)**, como al configurar aplicaciones  de Windows Server, gestionar herramientas de diseño, o dar soporte a usuarios que necesiten ver el escritorio en tiempo real. Usaria **SSH** para la administración de servidores Linux o Windows mediante **línea de comandos (CLI)**, ya que es la mejor opción para automatizar scripts, consumir el mínimo ancho de banda y garantizar conexiones rápidas, ligeras y altamente seguras en redes remotas.

