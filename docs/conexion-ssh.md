# Conexión remota y auditoria de red

Esta documentacion esta orientada no unicamente a resolver el paso detallado, sino tambien a resolver preguntas que pueden ir surgiendo en el aprendizaje. Por ejemplo 
previo a abrir un puerto para establecer una conexion externa. ¿Qué se encuentra abierto en mi servidor?

# Auditoria inicial de Puertos 

 Se realiza una auditoría de la superficie de ataque base del sistema operativo mediante el comando `sudo ss -tulpn`.

Este comando nos permite verificar qué sockets están escuchando tráfico (`LISTEN` en TCP) o activos en modo no conectado (`UNCONN` en UDP).

Notaremos mediante el comando ejecutado que el puerto 53 de DNS está abierto en IPv4 e IPv6 en escucha (TCP), y además está activo en IPv4 e IPv6 en UDP. Cabe destacar que el protocolo DNS mayoritariamente resuelve dominios de IPs por UDP; sin embargo, cuando la consulta supera el límite de los 512 bytes, es cuando utiliza el protocolo TCP de respaldo. 

Veremos también en UDP el puerto abierto de DHCP para la resolución de la IP privada del servidor y, por último, el puerto 22 abierto tanto en IPv4 como en IPv6 únicamente en TCP (OpenSSH).