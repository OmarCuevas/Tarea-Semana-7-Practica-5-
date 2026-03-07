# VPN Site-to-Site Punto a Punto con Túnel GRE

Autor: Omar Cuevas  
Asignatura: Seguridad de Redes  
Instituto: ITLA  

---

# Descripción General

En este laboratorio se implementó una VPN Site-to-Site punto a punto utilizando un túnel GRE (Generic Routing Encapsulation) entre dos routers Cisco.

El objetivo de esta práctica es permitir la comunicación entre dos redes LAN remotas a través de una red intermedia simulada (ISP), utilizando un túnel GRE para encapsular el tráfico entre los routers.

GRE permite transportar paquetes entre redes remotas creando una interfaz de túnel lógica entre los dispositivos.

Las redes conectadas en este laboratorio son:

192.168.63.0/24  
192.168.64.0/24  

---

# Objetivo del Laboratorio

El objetivo de este laboratorio es configurar una VPN Site-to-Site utilizando un túnel GRE entre dos routers Cisco.

Esto permite:

- Establecer comunicación entre dos redes LAN remotas
- Crear un túnel lógico entre routers
- Encapsular tráfico entre redes a través de una red pública
- Verificar la conectividad a través del túnel GRE

---

# Topología del Laboratorio

La topología utilizada conecta dos redes LAN mediante un proveedor de servicios simulado.

PC1 ---- R1 ---- ISP ---- R2 ---- PC2

Descripción de dispositivos:

Dispositivo | Función
------------|--------
PC1 | Cliente de la red LAN 1
R1 | Router de la red 1
ISP | Red intermedia simulada
R2 | Router de la red 2
PC2 | Cliente de la red LAN 2

<img width="1230" height="599" alt="image" src="https://github.com/user-attachments/assets/8d10c640-d769-4559-9de0-e0058295b860" />

---

# Direccionamiento IP

## Red LAN 1

Dispositivo | Interfaz | Dirección IP
------------|----------|-------------
PC1 | eth0 | 192.168.63.10
R1 | e0/2 | 192.168.63.1

---

## Red LAN 2

Dispositivo | Interfaz | Dirección IP
------------|----------|-------------
PC2 | eth0 | 192.168.64.10
R2 | e0/2 | 192.168.64.1

---

## Enlaces WAN

Dispositivo | Interfaz | Dirección IP
------------|----------|-------------
R1 | e0/1 | 10.63.1.1
ISP | e0/1 | 10.63.1.2
R2 | e0/1 | 10.63.2.2
ISP | e0/2 | 10.63.2.1

Máscara utilizada:

255.255.255.252

---

# Configuración del Túnel GRE

Para implementar el túnel GRE se configuró una interfaz Tunnel entre los routers R1 y R2.

Red del túnel:

172.16.1.0/30

Router | Interfaz | Dirección IP
------|------|------
R1 | Tunnel0 | 172.16.1.1
R2 | Tunnel0 | 172.16.1.2



---

# Configuración de los Dispositivos

## Router R1

hostname R1

interface e0/1  
 ip address 10.63.1.1 255.255.255.252  
 no shutdown  

interface e0/2  
 ip address 192.168.63.1 255.255.255.0  
 no shutdown  

interface tunnel0  
 ip address 172.16.1.1 255.255.255.252  
 tunnel source e0/1  
 tunnel destination 10.63.2.2  

ip route 192.168.64.0 255.255.255.0 172.16.1.2  

---

## Router R2

hostname R2

interface e0/1  
 ip address 10.63.2.2 255.255.255.252  
 no shutdown  

interface e0/2  
 ip address 192.168.64.1 255.255.255.0  
 no shutdown  

interface tunnel0  
 ip address 172.16.1.2 255.255.255.252  
 tunnel source e0/1  
 tunnel destination 10.63.1.1  

ip route 192.168.63.0 255.255.255.0 172.16.1.1  

---

## Router ISP

hostname ISP

interface e0/1  
 ip address 10.63.1.2 255.255.255.252  
 no shutdown  

interface e0/2  
 ip address 10.63.2.1 255.255.255.252  
 no shutdown  

ip route 192.168.63.0 255.255.255.0 10.63.1.1  
ip route 192.168.64.0 255.255.255.0 10.63.2.2  

---

## PC1 (VPCS)

ip 192.168.63.10 255.255.255.0 192.168.63.1  

---

## PC2 (VPCS)

ip 192.168.64.10 255.255.255.0 192.168.64.1  

---

# Verificación del Túnel

Para comprobar el funcionamiento del túnel GRE se utilizaron los siguientes comandos.

## Verificar interfaz del túnel

show ip interface brief

Este comando permite verificar que la interfaz Tunnel0 se encuentra activa (up/up).



---

## Verificar estado del túnel

show interface tunnel0

<img width="894" height="440" alt="image" src="https://github.com/user-attachments/assets/44968de4-04fa-4927-bcf6-4793b1e15298" />

---

## Prueba de conectividad

Desde PC1 se realiza la prueba:

ping 192.168.64.10

Esto confirma que la comunicación entre ambas redes LAN se realiza correctamente a través del túnel GRE.

<img width="635" height="187" alt="image" src="https://github.com/user-attachments/assets/8fd20fd2-8143-43e6-a493-e183d1311548" />

---


---

# Conclusión

En este laboratorio se configuró un túnel GRE punto a punto entre dos routers Cisco para permitir la comunicación entre redes remotas.

El túnel GRE permite encapsular tráfico entre redes a través de una red intermedia, creando una conexión lógica entre los routers.

Esta tecnología es utilizada en entornos de red para transportar tráfico entre ubicaciones remotas y puede combinarse con IPSec para agregar cifrado y seguridad adicional.
