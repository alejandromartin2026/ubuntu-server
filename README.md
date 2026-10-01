# Ubuntu Server - Guia desde Cero y Hardening

¡Bienvenidos a mi laboratorio de administracion de servidores! en este repositorio ducumento todo mi aprendizaje, configuracion y securización de servidores basado en **Ubuntu Server**.

El objetivo es crear una guia práctica que vaya desde la instalacion limpia hasta puesta en producción con buenas practicas de ciberseguridad.

## Estado del proyecto: En Desarrollo 
 
Este repositorio es mi laboratorio activo de estudio. El contenido se irá subiendo y actualizando de forma progresiva a medida que avance en mis practicas. 


## Contenido del repositorio 

*   **Instalación Básica:** Configuración del hipervisor, particionamiento y primer inicio.  
*   **Configuración de red:** Uso de Netplan, IPs estáticas y DNS, Port Forwarding y red nat
*   **Gestión de accesos:** Control de usuarios, privilegios con `sudo`, máscaras de permisos (`Umask`) y auditoría de seguridad.
*   **Servicios Esenciales** Despliegue de servidores web (Nginx/Apache) y virtualizacion con Docker.
*   **Hardening (Seguridad):** Desactivar acceso root por SSH y configurar el servicio, cambio de puerto de conexion SSH, configuracion de Firewall (UFW), llaves SSH, y fail2ban  

## Estructura del proyecto 

*   `/configs`: Archivos de configuración de muestra.
*   `/docs`: Guía detallada paso a paso.
    *   `01-instalacion.md` ✅ Completo
    *   `02-conexion-ssh.md` ✅ Completo 
    *   `03-netplan-firewall-fail2ban.md` ✅ Completo
    *   `04-usuarios-y-permisos.md` 🛠️ En curso
*   `/troubleshooting`: Guía sobre la resolución de problemas.
    *   `01-recuperacion-de-contraseña-grub.md`

## Requisitos del Laboratorio
      
*   **OS**: Ubuntu Server LTS 
*   **Entorno**: VirtualBox