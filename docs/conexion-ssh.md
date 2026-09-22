# Conexión remota y auditoria de red

Esta documentacion esta orientada no unicamente a resolver el paso detallado, sino tambien a resolver preguntas que pueden ir surgiendo en el aprendizaje. Por ejemplo 
previo a abrir un puerto para establecer una conexion externa. ¿Qué se encuentra abierto en mi servidor?

## Auditoria inicial de Puertos 

 Se realiza una auditoría de la superficie de ataque base del sistema operativo mediante el comando `sudo ss -tulpn`.

Este comando nos permite verificar qué sockets están escuchando tráfico (`LISTEN` en TCP) o activos en modo no conectado (`UNCONN` en UDP).

Notaremos mediante el comando ejecutado que el puerto 53 de DNS está abierto en IPv4 e IPv6 en escucha (TCP), y además está activo en IPv4 e IPv6 en UDP. Cabe destacar que el protocolo DNS mayoritariamente resuelve dominios de IPs por UDP; sin embargo, cuando la consulta supera el límite de los 512 bytes, es cuando utiliza el protocolo TCP de respaldo. 

Veremos también en UDP el puerto abierto de DHCP para la resolución de la IP privada del servidor y, por último, el puerto 22 abierto tanto en IPv4 como en IPv6 únicamente en TCP (OpenSSH).

## Configuración del Puente de Conexión: Reenvío de Puertos (Port Forwarding)

Sabiendo que el servidor ya tiene el puerto `22` abierto y escuchando en modo TCP, nos enfrentamos a un problema: el servidor se encuentra aislado en la red NAT de VirtualBox, por lo que nuestra máquina anfitriona no puede llegar a él directamente de forma nativa. 

Para solucionar esto y simular una administración remota real, configuramos una regla de **Reenvío de Puertos (Port Forwarding)** en el hipervisor. Esto creará un puente seguro que redirigirá el tráfico desde nuestra PC física hacia el entorno virtualizado.

### Paso 1: Configuración en VirtualBox
Con el servidor encendido o apagado, realizamos los siguientes pasos en la interfaz de VirtualBox:
1. Ir a **Configuración (Settings)** de la máquina virtual ➡️ sección **Red (Network)**.
2. Desplegar las opciones de **Avanzado (Advanced)** y hacer clic en **Reenvío de puertos (Port Forwarding)**.
3. Añadir una nueva regla con el ícono `+` y completar los campos con los siguientes parámetros técnicos:
   * **Nombre:** `SSH`
   * **Protocolo:** `TCP`
   * **IP Anfitrión (Host IP):** `127.0.0.1` *(Dirección de loopback de nuestra PC física).*
   * **Puerto Anfitrión (Host Port):** `2232` *(Puerto libre elegido en la máquina real para evitar conflictos).*
   * **Puerto Invitado (Guest Port):** `22` *(El puerto nativo donde OpenSSH escucha dentro del servidor).*
   * *Nota: Los campos de "IP Invitado" se dejan completamente en blanco.*

  <br>

<img src="../docs/img/ssh1.png" alt = "Regla en VirtualBox" width="500">

   <br>

### Paso 2: Conexión Remota desde la Máquina Anfitriona
Una vez aplicada la regla, el puente queda activo. Abrimos una terminal en nuestra computadora física (en mi caso Bash) y ejecutamos el comando de conexión remota apuntando a nuestro puerto puenteado:

```bash
ssh usuario-del-servidor@127.0.0.1 -p 2232
```
<br>

<img src="../docs/img/ssh2.png" alt = "Primer comando" width="500">

   <br>

#### ¿Qué sucede en el primer inicio?
La primera vez que establezcamos la conexión, OpenSSH nos mostrará una advertencia criptográfica sobre la autenticidad del servidor (`The authenticity of host... can't be established`). 
* Escribimos **`yes`** para confirmar y guardar la firma del servidor en nuestra PC física.

<br>

<img src="../docs/img/ssh3.png" alt = "Confirmación" width="500">

   <br>

* Introducimos la contraseña de nuestro usuario del servidor y ya habremos tomado el control del servidor de forma remota por SSH.

* En mi caso la primera conexion  despues de ingresar `yes` fallo, reinicie el servidor, volvi a intentar y funciono, automaticamente solicito usuario y contraseña e ingreso.   

<br>

<img src="../docs/img/ssh4.png" alt = "Regla en Conexion establecida en terminal" width="500">

   <br>


## Configuración de VirtualBox: Creación de una Red NAT

Para permitir la comunicación entre dos o más máquinas virtuales, procedemos a crear una **Red NAT** en VirtualBox. 

Por defecto, el modo **NAT** estándar aísla a cada máquina virtual en su propia subred, impidiendo que se conecten entre sí. En cambio, una **Red NAT** funciona como un **switch (conmutador) virtual compartido**: agrupa a ambos dispositivos dentro de la misma subred interna, permitiendo que se comuniquen entre sí (por ejemplo, para administración SSH o auditorías) mientras mantienen su acceso a Internet de forma segura y aislada de la red física local.

### Paso 1: Creación de la Red NAT en el Hipervisor

Estando en el panel general de VirtualBox, nos dirigimos al menú superior y hacemos clic en `Archivo` > `Herramientas` y seleccionamos la opción `Red`.

Una vez adentro del Administrador de Redes de VirtualBox, veremos tres pestañas. Debemos dirigirnos a la pestaña **Redes NAT** y luego hacer clic en el botón superior **Crear (+)**. Automáticamente se generará una red con un nombre por defecto, tal como se muestra en la siguiente imagen:

<br>

<img src="../docs/img/ssh5.png" alt = "Creación Red NAT" width="500">

   <br>

### Paso 2: Asociar las Máquinas Virtuales a la Red NAT

Para que los dispositivos puedan comunicarse entre sí, debemos cambiar la configuración de red tanto en la máquina cliente (**Linux Mint**) como en el servidor (**Ubuntu Server**):

1. Seleccionamos la máquina virtual en el panel de VirtualBox y hacemos clic en **Configuración** (ícono de engranaje).
2. En el menú lateral izquierdo, nos dirigimos al apartado de **Red**.
3. En la pestaña **Adaptador 1**, nos aseguramos de que la casilla *Habilitar adaptador de red* esté marcada.
4. En el desplegable *Conectar a*, cambiamos el modo de **NAT** a **Red NAT**.
5. En el campo *Nombre*, seleccionamos la red que creamos previamente (`NatNetwork`).

*Nota: Realizar este mismo procedimiento en ambas máquinas virtuales para introducirlas en la misma subred.*


<br>

<img src="../docs/img/ssh6.png" alt = "Cambio de red en dispositivos" width="500">

   <br>

## Conexión SSH Directa mediante Red NAT

Una vez que ambas máquinas virtuales están encendidas dentro de la misma **Red NAT**, ya no es necesario realizar un reenvío de puertos desde el host. La conexión se realiza de forma directa entre los sistemas virtuales.

### Paso 1: Identificar la IP del Servidor
En la consola de tu **Ubuntu Server**, ejecuta el siguiente comando para conocer su dirección IP privada interna:
```bash
ip a
```
*(Debes buscar la IP que se encuentra en el rango `10.0.2.X`)*

### Paso 2: Verificar la conectividad (Ping)
Desde la terminal de tu máquina cliente (**Linux Mint**), verifica que ambos sistemas se comuniquen correctamente ejecutando un ping hacia la IP del servidor (limitado a 4 paquetes con el parámetro `-c`):

```bash
ping -c 4 <IP_DEL_SERVIDOR>
```
<br>

<img src="../docs/img/ssh7.png" alt = "Prueba de conectividad" width="500">

   <br>


### Paso 3: Establecer la conexión SSH
Con la conectividad confirmada, ejecuta el comando de conexión SSH desde tu Linux Mint:

```bash
ssh tu_usuario_servidor@<IP_DEL_SERVIDOR>
```
  <br>

<img src="../docs/img/ssh8.png" alt = "conexión" width="500">

   <br>


 ## Hardening: Autenticación Avanzada mediante Llaves SSH

Para robustecer la seguridad del servidor y eliminar la necesidad de autenticarse mediante contraseñas tradicionales (vulnerables a ataques de fuerza bruta), se procedió a configurar un esquema de autenticación basado en **llaves criptográficas (pública/privada)** utilizando el algoritmo moderno **ED25519**.

### Paso 1: Generación de Llaves Personalizadas en la Máquina Cliente (Linux Mint)

Dado que la máquina cliente ya cuenta con un par de llaves asignadas para interactuar con **GitHub** (`id_ed25519`), es una buena práctica de seguridad y administración generar una llave exclusiva con un **nombre personalizado** para evitar sobreescribir las credenciales existentes.

Desde la terminal del cliente (Linux Mint), ejecutamos:
```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_ubuntu_lab -C "lab-mint"
```

* *Nota 1: El parámetro `-f` especifica la ruta y el nombre del nuevo archivo para que no colisione con el de GitHub.*
* *Nota 2: El parámetro `-C` añade un comentario descriptivo al final de la llave pública para identificar su origen en el servidor.*
* *Nota 3: En el asistente de la terminal, se presionó **Enter** en todas las solicitudes para dejar la clave (passphrase) en blanco, permitiendo un acceso automatizado.*

<br>

<img src="../docs/img/ssh9.png" alt = "generacion de llaves" width="500">

   <br>

### Paso 2: Transferencia del "Candado" (Llave Pública) al Servidor

Para inyectar de forma segura nuestra nueva llave pública en el servidor Ubuntu, utilizamos el comando automatizado `ssh-copy-id`. Al contar con múltiples llaves en nuestro directorio local, es obligatorio usar el parámetro `-i` (Identity) para indicarle explícitamente al sistema cuál archivo transferir:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_ubuntu_lab bluebeard@10.0.2.4
```
*Se solicitará la contraseña tradicional del usuario del servidor por última vez para autorizar la instalación del componente público.*

<br>

<img src="../docs/img/ssh10.png" alt = "copia" width="500">

   <br>

### Paso 3: Auditoría y Verificación en el Servidor (Mentalidad de Ciberseguridad)

Para validar que el servidor almacenó de forma íntegra y correcta el "candado" de nuestra máquina de laboratorio (y no otra llave por error), realizamos una verificación en dos pasos:

 **Lectura directa del almacén de llaves autorizadas (en el servidor):**
   ```bash
   cat ~/.ssh/authorized_keys
   ```
   *Se constató que la línea finalizaba con la etiqueta `lab-mint`, confirmando la procedencia exacta de la llave.*

<br>

<img src="../docs/img/ssh11.png" alt = "verificacion" width="500">

   <br>   



### Paso 4: Conexión SSH Directa y Automatizada

Con la arquitectura validada, la conexión SSH hacia el servidor se realiza apuntando de forma explícita a nuestra llave privada asignada, logrando iniciar sesión de forma instantánea sin requerir contraseñas:

```bash
ssh -i ~/.ssh/id_ed25519_ubuntu_lab bluebeard@10.0.2.4
```  

<br>

<img src="../docs/img/ssh12.png" alt = "conexión con llaves" width="500">

   <br> 

##  Automatización de la Conexión (Uso de SSH Config)

Escribir comandos largos que incluyan la ruta de la llave, el usuario y la dirección IP cada vez que se requiere administrar el servidor es ineficiente. Para solucionar esto, configuramos un **alias o atajo permanente** en la máquina cliente mediante el archivo de configuración de SSH.

### Paso 1: Configurar el archivo en el Cliente (Linux Mint)

Abrimos o creamos el archivo de configuración local con el editor Nano:
```bash
nano ~/.ssh/config
```

Dentro del archivo, ingresamos la estructura con los parámetros de nuestro servidor. Es una buena práctica aplicar **indentación (un Tab o 4 espacios)** en las líneas secundarias para indicar que esas propiedades pertenecen al Host declarado arriba:

```text
Host ubuntulab
    HostName 10.0.2.4
    User bluebeard
    IdentityFile ~/.ssh/id_ed25519_ubuntu_lab
```

<br>

<img src="../docs/img/ssh13.png" alt = "archivo nano" width="500">

   <br> 

### Paso 2: Conexión Simplificada mediante Alias

Una vez guardado el archivo (`Ctrl + O` y `Ctrl + X`), el sistema mapea los parámetros automáticamente. Ahora es posible conectarse al entorno de laboratorio utilizando únicamente el alias definido, resolviendo el usuario, la IP y la llave criptográfica por detrás:

```bash
ssh ubuntulab
```

<br>

<img src="../docs/img/ssh14.png" alt = "log con automatización" width="500">

   <br> 