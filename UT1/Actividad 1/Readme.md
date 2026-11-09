## Resumen de teoría

* **SSH:** Es un protocolo de red seguro que sirve para conectarte y controlar una máquina de forma remota a través de una consola de comandos. Toda la información va cifrada para que nadie pueda robar tus contraseñas ni lo que escribes.
* **Autenticación por contraseña vs claves:** La contraseña de toda la vida es menos segura porque te la pueden sacar por fuerza bruta y tienes que estar escribiéndola cada vez. El par de claves es mucho más seguro, usa criptografía y te deja entrar directamente de forma automática sin pedirte nada.
* **Clave pública y privada:** La clave pública es el candado y se copia en todos los servidores a los que te quieras conectar. La clave privada es tu llave, es secreta y solo se queda guardada en tu máquina cliente sin compartirse con nadie.
* **Endurecimiento del servicio:** Es cambiar la configuración para hacer el servidor más seguro una vez puestas las claves. Lo importante es desactivar el acceso al usuario root y prohibir que se pueda entrar usando contraseñas normales para cerrar la puerta a intrusos.

---

## Comandos utilizados en la práctica

### Paso 1: Comprobar la conectividad de red
```bash
# Se ejecuta en la máquina cliente (control)
ping -c 3 192.168.154.11
```
*Explicación:* Envía 3 paquetes de prueba a la IP del servidor nodo1 para comprobar que las dos máquinas se ven en la red y tienen conexión antes de empezar.

### Paso 2: Instalar y activar el servicio SSH en nodo1
```bash
# Si nodo1 es Ubuntu Server:
systemctl status ssh

# Si nodo1 es Windows Server:
Get-Service sshd
```
*Explicación:* Sirve para comprobar que el servicio de SSH está encendido, activo y listo en el servidor para recibir conexiones.

### Paso 3: Primera conexión por contraseña
```bash
# Conexión remota directa desde el cliente:
ssh hugo@192.168.154.11 "hostname; uptime; whoami"

# Crear un archivo de prueba y pasarlo por SCP:
echo "hola desde control" > prueba.txt
scp prueba.txt hugo@192.168.154.11:/home/hugo/
```
*Explicación:* El primer comando sirve para conectarte por contraseña a nodo1 y ejecutar comandos básicos sin abrir la sesión completa. El segundo comando crea un archivo de texto en local y lo copia de forma segura a la carpeta del usuario en el servidor.

### Paso 4: Generar el par de claves en control
```bash
# Se ejecuta en la máquina cliente (control)
ssh-keygen -t rsa -b 4096
```
*Explicación:* Genera tu clave pública y tu clave privada usando el algoritmo RSA con un tamaño seguro de 4096 bits. Le das a Enter para dejar la ruta por defecto y no le pones contraseña para la práctica.

### Paso 5: Desplegar la clave pública en nodo1
```bash
# Se ejecuta desde la máquina cliente hacia el servidor
ssh-copy-id hugo@192.168.154.11
```
*Explicación:* Copia tu clave pública de forma automática dentro del archivo de confianza del servidor nodo1 para que a partir de ahora te deje entrar directo sin pedir contraseña.

### Paso 6: Endurecer la configuración de nodo1
```bash
# 1. Abrir el archivo de configuración en el servidor:
sudo nano /etc/ssh/sshd_config

# 2. Cambiar estas líneas dentro del archivo:
PermitRootLogin no
PasswordAuthentication no

# 3. Comprobar que no nos hemos equivocado en nada:
sudo sshd -t

# 4. Reiniciar el servicio para aplicar los cambios:
sudo systemctl restart ssh
```
*Explicación:* Sirve para desactivar el acceso del usuario root y prohibir entrar con contraseñas normales. El comando `sshd -t` es muy importante porque comprueba si hay fallos antes de reiniciar para no quedarnos fuera del servidor.

### Paso 7: Prueba final
```bash
# 1. Probar a entrar desactivando las claves (Debe dar error):
ssh -o PubkeyAuthentication=no hugo@192.168.154.11

# 2. Entrar de forma normal (Debe entrar directo sin pedir contraseña):
ssh hugo@192.168.154.11
```
*Explicación:* El primer comando sirve para comprobar que el servidor ya rechaza las contraseñas normales de los usuarios. El segundo comando confirma que tu clave privada funciona bien y entras directo de forma segura.

---

## Reflexión: Ventajas de usar claves con muchos servidores

Si tienes que administrar muchos servidores a la vez, usar claves es mucho mejor por tres cosas:

1. **Automatización:** Te permite usar scripts o herramientas para hacer configuraciones en muchas máquinas a la vez de forma automática, porque no hay ninguna persona teniendo que escribir contraseñas una a una.
2. **Seguridad y comodidad:** Los administradores no tienen que estar recordando ni apuntando contraseñas difíciles para cada servidor, lo que evita que se apunten en sitios inseguros o se acaben filtrando.
3. **Control de usuarios:** Si un trabajador se va de la empresa, es tan fácil como borrar su clave pública de los servidores para quitarle el acceso rápido, sin tener que cambiar la contraseña de todos los demás ni molestar al resto del equipo.
