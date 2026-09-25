# Configuración de Red y Seguridad Perimetral (Firewall)

En este apartado se documenta el establecimiento de un direccionamiento IP estático, la asignación de servidores DNS seguros y el endurecimiento de la seguridad de red mediante reglas de Firewall (UFW).

## 1. Configuración de IP Estática y DNS mediante Netplan
 
 Primero se debe localizar el archivo dentro de la ruta `/etc/netplan` ejecutamos el comando:
 ```bash
 ls /etc/netplan
 ```
 Una vez localizado el archivo procedemos a realizar una copia del mismo, como paso preventivo. En caso de romper el sistema es importante contar con una copia. Mediante `cp` como se muestra en la imagen mas abajo. 

<br>

<img src="../docs/img/netplan1.png" alt = "ruta del archivo" width="500">

   <br>

<br>

<img src="../docs/img/netplan2.png" alt = "copia del archivo" width="500">

   <br>   





