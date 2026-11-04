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
