## 1. Instalación Segura desde el Repositorio Oficial (GPG)

Para garantizar la integridad y la actualización constante del motor de contenedores ante posibles vulnerabilidades de día cero (*Zero-Days*), evitamos el uso de los paquetes genéricos de los repositorios de Ubuntu. En su lugar, configuramos de forma segura la fuente oficial de Docker mediante la verificación de firmas criptográficas GPG.

### Comandos ejecutados en el Laboratorio:

#### Paso 1: Purga y Remoción de Paquetes Conflictivos Anteriores
Como primera medida de hardening, es indispensable asegurar que el sistema operativo no cuente con versiones de Docker obsoletas, paquetes genéricos de la distribución (`docker.io`) o empaquetados en formato Snap/Flatpak. Esto previene conflictos en la gestión de binarios y elimina dependencias desactualizadas que podrían contener fallas de seguridad explotables.

Ejecutamos el comando de remoción masiva en el servidor:

```bash
sudo apt remove \$(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc docker-buildx podman-docker containerd runc | cut -f1)
```

<br>
<img src="../docs/img/docker1.png" alt="Purga inicial de paquetes no oficiales" width="1200">
<br>


#### Paso 2: Actualizar los índices del sistema y validar dependencias previas
Preparamos el gestor de paquetes `apt` para asegurar que pueda comunicarse a través de protocolos seguros HTTPS y manejar certificados de confianza:

```bash
sudo apt update
sudo apt install ca-certificates curl
```

<br>
<img src="../docs/img/docker2.png" alt="Validación de dependencias ca-certificates y curl" width="1100">
<br>

*Nota de laboratorio: En mi caso, como se observa en la consola, el gestor de paquetes verificó el sistema y determinó que ambos componentes ya se encuentran instalados de fábrica en su versión más reciente, dejando el entorno listo y asegurado para el siguiente paso.*

#### Paso 3: Creación de Directorio Seguro y Descarga de la Llave Criptográfica Oficial
Para que el gestor de paquetes confíe en el repositorio de Docker, debemos descargar su firma digital legítima. Siguiendo las mejores prácticas actuales de Debian/Ubuntu, almacenamos esta llave en un directorio aislado y restringido de confianza (`keyrings`).

Ejecutamos la secuencia para generar la ruta y descargar el certificado oficial:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://docker.com -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

<br>
<img src="../docs/img/docker3.png" alt="Descarga e instalación de la llave oficial docker.asc" width="1100">
<br>

**Análisis de Hardening Aplicado:**
*   **`install -m 0755 -d`**: A diferencia de un `mkdir` común, este comando crea la carpeta `/etc/apt/keyrings` y le clava de forma explícita los permisos seguros `755` (Lectura, escritura y ejecución para root; lectura y entrada para el resto), garantizando que ningún usuario común pueda modificar su contenido.
*   **`docker.asc`**: Es la llave criptográfica oficial en formato de texto claro firmado.
*   **`chmod a+r`**: Le otorga permisos de lectura a todos los usuarios del sistema (`all + read`) única y exclusivamente sobre el archivo de la llave, permitiendo que el gestor `apt` pueda leer la firma para validar los paquetes, pero impidiendo cualquier tipo de alteración o modificación maliciosa.

