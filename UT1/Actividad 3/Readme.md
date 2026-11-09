##  Resumen de teoría 

* **Mínimo privilegio:** Es un principio de seguridad informática que consiste en dar a cada usuario únicamente los permisos y accesos estrictamente necesarios para realizar su trabajo, evitando que cuentas comunes tengan privilegios de administración o accesos a servicios que no necesitan.
* **AllowGroups en Linux y en Windows:** Es una directiva de configuración que sirve para restringir el acceso por SSH. Al usarla, solo los usuarios que pertenezcan a los grupos que pongas en esa lista podrán conectarse, rechazando automáticamente a todos los demás usuarios del sistema.
* **El SID del grupo de escritorio remoto:** El Identificador de Seguridad (SID) universal para el grupo de usuarios de escritorio remoto en Windows es `S-1-5-32-555`. Se usa en los comandos de PowerShell para añadir usuarios al grupo sin importar el idioma del sistema operativo, ya que el nombre del grupo cambia según si Windows está en inglés o en español.
* **Tsusers en xRDP:** Es el grupo de seguridad específico que se crea en Linux para controlar quién puede usar el entorno gráfico remoto. Si este grupo existe, xRDP restringe el acceso y solo permite iniciar sesión a los usuarios que formen parte de él, rechazando al resto.
* **Qué es un túnel SSH:** Es una técnica que permite crear un canal de comunicación cifrado y seguro entre tu ordenador y el servidor a través del protocolo SSH. Sirve para encapsular el tráfico de otros servicios (como el panel web Webmin) de manera que viajen protegidos por dentro de la conexión SSH ya existente.
* **Por qué no se expone un panel de administración a la red:** No se debe dejar el puerto de un panel como Webmin (puerto 10000) abierto a Internet porque se convierte en un objetivo directo para atacantes. Al cerrarlo en el firewall y acceder solo mediante un túnel SSH, obligas a cualquiera a tener que superar primero la seguridad y las claves de SSH antes de poder siquiera ver la pantalla de inicio del panel web.

---

##  Comandos utilizados en la práctica

### 1. Cuentas en Linux y restricción de SSH

* **Crear los usuarios normales:**
  ```bash
  sudo adduser soporte
  sudo adduser invitado
  ```
  *Explicación:* Crea dos usuarios estándar en el sistema operativo Linux llamados `soporte` e `invitado`.

* **Crear el grupo sshusers y añadir a los miembros:**
  ```bash
  sudo groupadd sshusers
  sudo usermod -aG sshusers soporte
  sudo usermod -aG sshusers hugo
  ```
  *Explicación:* Crea un grupo nuevo para centralizar el acceso SSH e incluye en él a `soporte` y al usuario administrador `hugo` para no perder el acceso a la máquina.

* **Configurar la restricción en SSH y reiniciar el servicio:**
  ```bash
  sudo nano /etc/ssh/sshd_config.d/10-acceso.conf
  # Añadir dentro: AllowGroups sshusers

  sudo sshd -t && sudo systemctl restart ssh
  sudo sshd -T | grep -i allowgroups
  ```
  *Explicación:* Crea el archivo para limitar el acceso SSH únicamente al grupo `sshusers`, comprueba que no haya fallos de sintaxis, reinicia el servicio SSH para aplicar los cambios y comprueba que se ha guardado bien la regla.

* **Ver el registro de accesos rechazados en SSH desde el cliente:**
  ```bash
  # Comando de prueba ejecutado desde el equipo cliente:
  ssh invitado@192.168.154.140

  # Comando ejecutado en el servidor para ver el rechazo:
  sudo journalctl -u ssh | grep "not allowed"
  ```
  *Explicación:* Intenta acceder con el usuario no autorizado desde el cliente para forzar el fallo y luego busca en el registro del servidor las líneas que confirman que `invitado` fue rechazado.

* **Quitar y devolver permisos (Prueba de revocación):**
  ```bash
  sudo gpasswd -d soporte sshusers
  sudo usermod -aG sshusers soporte
  ```
  *Explicación:* Expulsa temporalmente a `soporte` del grupo para comprobar que ya no puede entrar y lo vuelve a añadir sin tocar el fichero de configuración principal.

### 2. Restringir el escritorio remoto (xRDP) en Linux

* **Crear el grupo tsusers y asignar miembros autorizados:**
  ```bash
  sudo groupadd tsusers
  sudo usermod -aG tsusers soporte
  sudo usermod -aG tsusers hugo
  sudo systemctl restart xrdp xrdp-sesman
  ```
  *Explicación:* Al crear el grupo `tsusers` e introducir a los usuarios permitidos, xRDP empieza a filtrar las conexiones. Se reinician los servicios para aplicar los cambios.

* **Ver los registros de rechazo de xRDP:**
  ```bash
  sudo tail -n 30 /var/log/xrdp-sesman.log
  ```
  *Explicación:* Muestra las últimas líneas del registro de xRDP para localizar el motivo del rechazo del usuario `invitado` al intentar conectar por escritorio remoto.

### 3. Cuentas en Windows y escritorio remoto

* **Crear usuarios de forma segura mediante PowerShell:**
  ```powershell
  \$clave = Read-Host -AsSecureString "Contraseña"
  New-LocalUser -Name "soporte" -Password \$clave -FullName "Usuario de soporte"
  New-LocalUser -Name "invitado" -Password \$clave -FullName "Usuario invitado"
  ```
  *Explicación:* Solicita de manera oculta una contraseña en la consola y crea de forma segura los dos usuarios locales en Windows.

* **Conceder el permiso de Escritorio Remoto por el SID del grupo:**
  ```powershell
  Add-LocalGroupMember -SID S-1-5-32-555 -Member "soporte"
  Get-LocalGroupMember -SID S-1-5-32-555
  ```
  *Explicación:* Añade al usuario `soporte` al grupo de escritorio remoto utilizando su identificador universal (`S-1-5-32-555`) y verifica los miembros actuales.

* **Crear el grupo de SSH en Windows y añadir usuarios:**
  ```powershell
  New-LocalGroup -Name "sshusers" -Description "Usuarios con acceso SSH"
  Add-LocalGroupMember -Group "sshusers" -Member "soporte"
  Add-LocalGroupMember -Group "sshusers" -Member "hugo"
  ```
  *Explicación:* Crea el grupo local en Windows para restringir los accesos de OpenSSH y añade las cuentas autorizadas de `soporte` y `hugo`.

* **Restringir la directiva AllowGroups en la configuración de Windows:**
  ```powershell
  notepad C:\ProgramData\ssh\sshd_config
  # Añadir antes del bloque Match: AllowGroups sshusers

  & "\$env:windir\System32\OpenSSH\sshd.exe" -t
  Restart-Service sshd
  ```
  *Explicación:* Abre con el Bloc de notas y permisos de administrador el fichero de configuración, añade la restricción para el grupo, comprueba la sintaxis de OpenSSH y reinicia el servicio de Windows.

### 4. Instalación de Webmin y creación del Túnel SSH

* **Instalar Webmin mediante script oficial en Linux:**
  ```bash
  sudo apt update
  sudo apt install curl -y
  curl -o webmin-setup-repo.sh https://githubusercontent.com
  sudo sh webmin-setup-repo.sh
  sudo apt install webmin --install-recommends -y
  ```
  *Explicación:* Descarga las llaves y el script oficial de Webmin en Ubuntu, añade sus repositorios del sistema y realiza la instalación completa del panel web.

* **Comprobar el estado y los puertos de Webmin:**
  ```bash
  systemctl status webmin
  sudo ss -tlnp | grep 10000
  ```
  *Explicación:* Revisa si el panel de Webmin está activo y si se encuentra escuchando peticiones locales a través del puerto por defecto 10000.

* **Configuración y cierre del puerto en el Firewall (UFW):**
  ```bash
  sudo ufw status
  sudo ufw allow 10000/tcp        # Apertura temporal para la prueba inicial
  sudo ufw delete allow 10000/tcp # Cierre definitivo del puerto al exterior
  ```
  *Explicación:* Permite revisar el estado del firewall local, abrir el puerto de forma momentánea para la primera prueba desde el navegador cliente (`https://192.168.154.140:10000`) y finalmente eliminar la regla para blindar el panel.

* **Asegurar las reglas esenciales en UFW antes de activarlo:**
  ```bash
  sudo ufw allow OpenSSH
  sudo ufw allow 3389/tcp
  sudo ufw enable
  sudo ufw status
  ```
  *Explicación:* Permite los accesos obligatorios de SSH y RDP en el cortafuegos antes de encenderlo para evitar perder la comunicación con el servidor permanentemente.

* **Comando ejecutado EN EL CLIENTE para abrir el Túnel SSH:**
  ```bash
  # Ejecutar en PowerShell o Terminal de tu máquina cliente:
  ssh -L 10000:localhost:10000 soporte@192.168.154.140
  ```
  *Explicación:* Crea el túnel mapeando el puerto 10000 de tu máquina local hacia el puerto 10000 interno del servidor Ubuntu (`192.168.154.140`) usando el usuario autorizado `soporte`. Esto permite abrir el navegador web del cliente e ingresar de forma segura usando la dirección de bucle local: `https://localhost:10000`.
