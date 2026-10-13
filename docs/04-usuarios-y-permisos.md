# Hardening Gestión de usuarios y permisos

## 1. Creación de Usuarios y Configuración de Sudo


El primer paso para asegurar nuestro servidor es aplicar el principio de menor privilegio y separación de funciones. Si bien Ubuntu Server bloquea al usuario root de fábrica y nos obliga a usar un usuario principal con capacidades de administración **(bluebeard)**, la buena práctica dicta que debemos segmentar los accesos. Creamos un usuario de administración específico para el laboratorio **(admin-01)** y le otorgamos privilegios de elevación mediante visudo para auditar y separar las tareas cotidianas del control general del sistema.

###  Diferencia técnica: `useradd` vs `adduser`
Al momento de crear un usuario en Ubuntu Server, existen dos comandos que suelen confundirse:
*   **`useradd`:** Es el comando nativo de bajo nivel. Crea el usuario de forma "cruda", lo que significa que no genera su contraseña, ni su directorio en `/home`, ni su entorno de terminal a menos que se lo indiquemos con parámetros manuales avanzados.
*   **`adduser`:** Es un script de alto nivel (asistente interactivo). Automatiza todo el proceso: solicita la contraseña de forma segura, crea la carpeta `/home/usuario`, configura el entorno por defecto (skel) y solicita datos informativos opcionales. Es la opción recomendada y la utilizada en este laboratorio.

###  Comandos ejecutados en el Laboratorio:

1. **Crear el nuevo usuario seguro:**
   ```bash
   sudo adduser admin-01
   ```
   *(El sistema nos pedirá ingresar y confirmar la nueva contraseña, y luego podemos presionar `ENTER` para saltar los campos de datos personales).*   

   <br>

<img src="../docs/img/usuario1.png" alt = "Creamos usuario con adduser" width="700">

   <br> 

   ###  Otorgar Privilegios de Administrador de forma Segura (`visudo`)

Para que nuestro usuario `admin-01` pueda ejecutar tareas administrativas, debemos registrarlo en el archivo de configuración de sudoers. La práctica recomendada de seguridad dicta que nunca debemos editar este archivo con editores comunes como `nano` o `vim` directamente, ya que un error de sintaxis podría bloquear el acceso `sudo` para todo el sistema.

En su lugar, utilizamos el comando seguro:
```bash
sudo visudo
```
*Este comando abre el archivo en un entorno seguro que verifica que la sintaxis sea correcta antes de guardar los cambios.*

#### Configuración aplicada:
Nos desplazamos hasta el final del archivo y añadimos la siguiente regla para darle control total a nuestro nuevo administrador:

```text
admin-01 ALL=(ALL:ALL) ALL
```

<br>
<img src="../docs/img/usuario2.png" alt="Configuración de visudo para admin-01" width="700">
<br>

Luego `ctrl o` para guardar y `ctrl x` para salir. Nuestro usuario admin-01 ya tiene permiso para utilizar el sudo en sus comandos. 

Por medio del comando `su admin-01` podemos ingresar a nuestro usuario, en la imagen de mi lab pruebo un comando simple de actualización de sistema para verficar que todo funciona de manera correcta.

<br>
<img src="../docs/img/usuario3.png" alt="Configuración de visudo para admin-01" width="700">
<br>

## 2. Gestión de Permisos sobre el Nuevo Usuario

Una vez creado el usuario, es fundamental comprender cómo administra Linux el acceso a los archivos y carpetas bajo su modelo de seguridad de tres capas (Dueño, Grupo, Otros).

###  Estructura de los Permisos en Linux
Cuando ejecutamos un comando de listado detallado (`ls -l`), el sistema nos devuelve una cadena de caracteres a la izquierda que define el tipo de elemento y sus privilegios.

#### 1. Primer carácter (Tipo de elemento):
*   **`-` (Guion):** Identifica que el elemento es un **archivo común** (un texto, una imagen, un script).
*   **`d`:** Identifica que es un **directorio** o carpeta.
*   **`l`:** Identifica que es un **enlace simbólico** (*symlink*), actuando como un acceso directo.

#### 2. Los 9 caracteres siguientes (Bloques de permisos):
Se dividen en tres conjuntos de tres letras (`r` = lectura, `w` = escritura, `x` = ejecución):
*   **Primeros 3:** Permisos para el **Dueño** (User).
*   **Segundos 3:** Permisos para el **Grupo** (Group).
*   **Últimos 3:** Permisos para **Otros** (Others - cualquier usuario fuera de los anteriores).

#### Ejemplo Teórico: El caso `761`
Si tenemos un archivo con la estructura `-rwxrw---x`, se traduce al sistema numérico (octal) sumando los valores asignados a cada propiedad: **Lectura (4), Escritura (2), Ejecución (1)**.

*   **Dueño (`rwx`):** 4 + 2 + 1 = **7** (Tiene control total).
*   **Grupo (`rw-`):** 4 + 2 + 0 = **6** (Puede leer y modificar, pero no ejecutar).
*   **Otros (`--x`):** 0 + 0 + 1 = **1** (Solo puede ejecutar el archivo, no leerlo ni editarlo).



###  Alta de Conexión SSH para el Nuevo Administrador

Una vez creado el nuevo usuario administrador y otorgados sus permisos en `visudo`, el siguiente paso crítico es permitir su acceso remoto mediante **SSH**. Si bien este laboratorio ya cuenta con un apartado general de SSH, considero una excelente práctica documentar cómo se gestionaría este proceso en un entorno corporativo real bajo estándares estrictos de seguridad.

En este escenario simulado, no compartiremos contraseñas. Le solicitaremos al nuevo usuario (**admin-01**) que genere su propio par de claves criptográficas (llave pública y privada) en su dispositivo local mediante el siguiente comando:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_admin-01 -C "admin-01-lab"
```

>**Nota de Infraestructura del Lab:** Para simular este comportamiento en nuestro entorno, utilizamos el mismo host físico (Linux Mint) desde donde opera el usuario principal **bluebeard**, actuando en la imaginación como si fuera la computadora externa del nuevo colega. Gracias a los parámetros `-f` (indica el nombre único del archivo) y `-C` (comentario identificatorio), podemos almacenar múltiples llaves de forma ordenada en un mismo cliente SSH sin pisar ni romper nuestras llaves personales previas de GitHub o del propio servidor.

<br>
<img src="../docs/img/usuario4.png" alt="Creación de llaves SSH para admin-01" width="700">
<br>

###  2. Inyección de la Llave Pública en el Servidor

Para el siguiente paso, el usuario administrador debe transferir de manera segura su "candado" (llave pública) hacia el servidor. Conociendo de antemano la dirección IP estática y el puerto personalizado del servicio SSH de nuestro entorno de VirtualBox, ejecutamos el comando nativo `ssh-copy-id` especificando la ruta exacta de la identidad:

```bash
ssh-copy-id -p 2232 -i ~/.ssh/id_admin-01.pub admin-01@10.0.2.4
```

<br>
<img src="../docs/img/usuario5.png" alt="Copia de llave pública en el servidor" width="700">
<br>

Como se observa en el output de la terminal, el servidor solicita la credencial local del usuario por única vez, procesa la firma criptográfica y confirma la inyección exitosa (`Number of key(s) added: 1`).

###  3. Acceso Directo Seguro

Con la llave pública ya registrada en el archivo `authorized_keys` del servidor, procedemos a realizar la conexión remota desde el cliente. Indicamos de forma explícita qué llave privada utilizar para la autenticación a través del parámetro `-i`:

```bash
ssh -p 2232 -i ~/.ssh/id_admin-01 admin-01@10.0.2.4
```

<br>
<img src="../docs/img/usuario6.png" alt="Login exitoso por llave SSH sin contraseña" width="700">
<br>

La conexión se establece de forma instantánea. El servidor valida el par de claves y nos otorga **acceso directo a la shell de administración sin pedir contraseñas**, logrando un entorno robusto, eficiente y completamente protegido contra vectores de ataque de fuerza bruta por diccionario.



## Creación de Grupo y Usuarios con Comando `useradd`

Primero se creará el grupo de desarrollo y luego se darán de alta 3 usuarios nuevos (**dev-01**, **dev-02** y **dev-03**) utilizando el comando nativo de bajo nivel. Al ejecutar este proceso, los incorporaremos directamente al grupo creado previamente.

### Creación del Grupo

Utilizamos el comando `groupadd` para generar el nuevo contenedor de seguridad en el sistema:

```bash
sudo groupadd desarrollo
```

### Creación de Usuarios e Incorporación al Grupo

Para automatizar el despliegue de forma eficiente, utilizamos parámetros avanzados en una sola línea de comandos para cada usuario:

```bash
sudo useradd -m -s /bin/bash -G desarrollo dev-01
sudo useradd -m -s /bin/bash -G desarrollo dev-02
sudo useradd -m -s /bin/bash -G desarrollo dev-03
```

**Desglose de parámetros utilizados:**
*   **`-m`:** Fuerza al sistema a crear el directorio personal en `/home/usuario` (copiando los archivos base de `/etc/skel`).
*   **`-s /bin/bash`:** Define explícitamente la shell Bash como entorno de comandos (evitando que asigne `/bin/sh` por defecto, la cual es muy limitada y carece de autocompletado).
*   **`-G desarrollo`:** Añade al usuario al grupo secundario `desarrollo` en el mismo momento de su nacimiento.

<br>
<img src="../docs/img/usuario7.png" alt="Creación de usuarios con useradd" width="700">
<br>

Para confirmar que los directorios personales se configuraron de forma correcta bajo el estándar del sistema, listamos el contenido de `/home`:

```bash
ls -la /home
```

<br>
<img src="../docs/img/usuario8.png" alt="Verificación de creación correcta en home" width="700">
<br>

### Asignación de Contraseñas (Inicialización de Cuentas)

Al ser creados con la herramienta de bajo nivel `useradd`, los usuarios nacen en estado "bloqueado" y sin una clave asignada. Para habilitar sus accesos, procedemos a establecer sus credenciales individuales mediante el comando `passwd`:

```bash
sudo passwd dev-01
sudo passwd dev-02
sudo passwd dev-03
```
<br>
<img src="../docs/img/usuario9.png" alt="contraseñas de usuarios" width="500">
<br>

## Configuración del Espacio Compartido en Directorio de Servicios (`/srv`)

Siguiendo el estándar de jerarquía de archivos de Linux (**FHS**), utilizaremos el directorio `/srv` (destinado a los datos de servicios y proyectos que proporciona el sistema) para centralizar el espacio de trabajo del equipo. Evitamos usar `/home` para garantizar una separación estricta entre los datos privados de los usuarios y los activos de la organización.

### Creación del Directorio de Trabajo

Creamos la carpeta donde el equipo de desarrollo colaborará de forma centralizada:

```bash
sudo mkdir -p /srv/desarrollo
```

### Configuración del Grupo Propietario

Por defecto, la carpeta recién creada pertenece a `root:root`. Para permitir que nuestro nuevo grupo tenga acceso, transferimos la propiedad del grupo a `desarrollo` manteniendo a `root` como dueño del archivo:

 *   **`chown`**: Es la herramienta nativa para modificar los propietarios de un elemento en el sistema de archivos.
 *   **El uso de los dos puntos (`:desarrollo`)**: Esta es una buena práctica clave. Al anteponer los dos puntos al nombre del grupo y omitir el usuario, le indicamos al sistema que **mantenga a `root` como usuario dueño original**, pero que transfiera de forma exclusiva el **grupo propietario** a `desarrollo`.
 *   **`/srv/desarrollo`**: Especifica el objetivo o la ruta absoluta sobre la cual se aplicará la directiva.

```bash
sudo chown :desarrollo /srv/desarrollo
```

<br>
<img src="../docs/img/usuario10.png" alt="Creación de carpeta en srv y asignación de grupo" width="700">
<br>

## 5. Hardening de Permisos con Bit Especial (SGID)

Si dejamos el directorio compartiendo sus privilegios por defecto (`drwxr-xr-x`), los desarrolladores se toparán con dos problemas graves de seguridad y usabilidad: no podrán escribir en la carpeta (permiso denegado) y cualquier otro usuario ajeno al proyecto podrá leer el código fuente.

Para solucionar esto aplicando el principio de menor privilegio, implementamos el bit especial **SGID (Set Group ID)** mediante la nomenclatura octal de cuatro dígitos `2770`.

### Aplicación del Hardening

Ejecutamos el cambio de privilegios sobre la carpeta compartida:

```bash
sudo chmod 2770 /srv/desarrollo
```

**Desglose técnico del valor `2770`:**
*   **`2` (Bit Especial SGID):** Garantiza que cualquier archivo creado dentro de este directorio herede automáticamente el grupo propietario (`desarrollo`), permitiendo el trabajo colaborativo fluido sin importar qué usuario cree el archivo.
*   **Primer `7` (Dueño / Root):** Mantiene el control total para el administrador del sistema (`rwx`).
*   **Segundo `7` (Grupo / Desarrollo):** Otorga permisos de lectura, escritura y ejecución (`rwx`) para que el equipo de desarrollo pueda trabajar.
*   **El `0` Final (Otros / Resto del mundo):** Aplica una restricción absoluta (`---`). Ningún usuario ajeno al grupo desarrollo podrá ingresar, leer ni modificar el contenido de este espacio.

<br>
<img src="../docs/img/usuario11.png" alt="Aplicación de chmod 2770 y verificación del SGID" width="700">
<br>