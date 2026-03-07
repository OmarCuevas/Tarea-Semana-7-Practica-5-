# VPN Site-to-Site Punto a Punto con Túnel GRE

**Autor:** Omar Cuevas  
**Asignatura:** Seguridad de Redes  
**Instituto:** ITLA  

---

## Descripción General

En esta práctica se implementó una **VPN site-to-site punto a punto con túnel GRE** en un laboratorio de **PNETLab** utilizando routers **Cisco IOU**. La solución fue construida sobre la misma topología física utilizada en las prácticas anteriores, pero en esta ocasión se eliminó la configuración de **VPN basada en enrutamiento con VTI** para montar un **túnel GRE puro**, sin IPSec y sin IKEv2.

El propósito de esta práctica fue encapsular el tráfico entre dos redes LAN remotas por medio de una interfaz lógica **Tunnel0**, de manera que las redes **192.168.63.0/24** y **192.168.64.0/24** pudieran comunicarse a través de una red intermedia no confiable usando **GRE (Generic Routing Encapsulation)**.

A diferencia de las prácticas anteriores:
- **No se utilizó crypto map**
- **No se utilizó ACL de cifrado**
- **No se utilizó IKEv2**
- **No se utilizó IPSec**
- **Sí se utilizó Tunnel0**
- **Sí se utilizó `tunnel mode gre ip`**

La validación de la práctica se realizó correctamente mediante:
- **Ping entre las WAN**
- **Ping entre las IP del túnel GRE**
- **Ping entre PC1 y PC2**
- **show ip interface brief**
- **show ip route**
- **show interfaces Tunnel0**
- **show run interface Tunnel0**

---

## Objetivo

Configurar una **VPN site-to-site punto a punto con túnel GRE** entre **R1** y **R2**, permitiendo la comunicación entre las redes **192.168.63.0/24** y **192.168.64.0/24** a través de una red intermedia, utilizando una interfaz **Tunnel0** y rutas estáticas hacia la red remota.

---

## Topología

PC1 ---- R1 ---- ISP ---- R2 ---- PC2

<img width="1151" height="626" alt="image" src="https://github.com/user-attachments/assets/812298fd-ae2f-494a-8821-19a6238a424f" />


---

## Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Máscara | Descripción |
|------------|----------|--------------|---------|-------------|
| R1 | e0/1 | 10.63.1.1 | 255.255.255.252 | Enlace WAN hacia ISP |
| ISP | e0/1 | 10.63.1.2 | 255.255.255.252 | Enlace hacia R1 |
| ISP | e0/2 | 10.63.2.1 | 255.255.255.252 | Enlace hacia R2 |
| R2 | e0/1 | 10.63.2.2 | 255.255.255.252 | Enlace WAN hacia ISP |
| R1 | e0/2 | 192.168.63.1 | 255.255.255.0 | LAN sede 1 |
| R2 | e0/2 | 192.168.64.1 | 255.255.255.0 | LAN sede 2 |
| R1 | Tunnel0 | 192.168.100.1 | 255.255.255.252 | Extremo del túnel GRE |
| R2 | Tunnel0 | 192.168.100.2 | 255.255.255.252 | Extremo del túnel GRE |
| PC1 | NIC | 192.168.63.10 | 255.255.255.0 | Host sede 1 |
| PC2 | NIC | 192.168.64.10 | 255.255.255.0 | Host sede 2 |

### Gateway de los hosts
- **PC1:** 192.168.63.1
- **PC2:** 192.168.64.1

---

## Parámetros de la Práctica

| Parámetro | Valor |
|----------|-------|
| Tipo de túnel | GRE |
| Protocolo de túnel | GRE (IP protocolo 47) |
| Cifrado | No aplica |
| IKE | No aplica |
| IPSec | No aplica |
| Crypto ACL | No requerida |
| Crypto map | No requerido |
| Subred del túnel | 192.168.100.0/30 |
| Selección de tráfico | Basada en rutas |
| Modo del túnel | `tunnel mode gre ip` |

---

## Explicación de Funcionamiento

En esta práctica la conectividad entre ambas sedes se logró mediante un **túnel GRE** configurado entre **R1** y **R2**. Cada router crea una interfaz lógica **Tunnel0**, la cual usa como origen la interfaz WAN local y como destino la IP WAN del router remoto.

La lógica de funcionamiento es la siguiente:

1. **PC1** intenta comunicarse con **PC2**
2. **R1** revisa su tabla de enrutamiento
3. La red **192.168.64.0/24** apunta hacia **192.168.100.2**, que es la IP del túnel remoto
4. El tráfico entra por **Tunnel0**
5. El router encapsula ese tráfico usando **GRE**
6. El tráfico encapsulado viaja por la WAN entre **10.63.1.1** y **10.63.2.2**
7. **R2** recibe el tráfico GRE, lo desencapsula y lo entrega a la red **192.168.64.0/24**

La principal diferencia con la práctica anterior de **route-based con VTI** es que en esta práctica:
- El túnel **no está protegido por IPSec**
- No existe **IKEv2**
- No existe **IPSec profile**
- No existe **tunnel protection**
- El túnel funciona únicamente con **GRE**

---

## Qué se Eliminó de la Práctica Anterior

Como previamente la topología estaba configurada en modo **route-based con VTI / IKEv2**, antes de montar esta práctica fue necesario eliminar la configuración anterior, incluyendo:

- Interface **Tunnel0** anterior
- Crypto ipsec profile anterior
- Crypto ipsec transform-set anterior
- Crypto ikev2 profile anterior
- Crypto ikev2 keyring anterior
- Crypto ikev2 policy anterior
- Crypto ikev2 proposal anterior
- Rutas estáticas hacia la red remota por la VTI

---

## Configuración de los Dispositivos

### ISP

El router ISP solo se utilizó para dar tránsito entre ambos extremos.

    hostname ISP
    no ip domain-lookup

    interface Ethernet0/1
     ip address 10.63.1.2 255.255.255.252
     no shutdown

    interface Ethernet0/2
     ip address 10.63.2.1 255.255.255.252
     no shutdown

    ip route 192.168.63.0 255.255.255.0 10.63.1.1
    ip route 192.168.64.0 255.255.255.0 10.63.2.2

---

### R1

    hostname R1
    no ip domain-lookup

    interface Ethernet0/1
     ip address 10.63.1.1 255.255.255.252
     no shutdown

    interface Ethernet0/2
     ip address 192.168.63.1 255.255.255.0
     no shutdown

    ip route 0.0.0.0 0.0.0.0 10.63.1.2

    interface Tunnel0
     description GRE-a-R2
     ip address 192.168.100.1 255.255.255.252
     tunnel source Ethernet0/1
     tunnel destination 10.63.2.2
     tunnel mode gre ip
     no shutdown

    ip route 192.168.64.0 255.255.255.0 192.168.100.2

---

### R2

    hostname R2
    no ip domain-lookup

    interface Ethernet0/1
     ip address 10.63.2.2 255.255.255.252
     no shutdown

    interface Ethernet0/2
     ip address 192.168.64.1 255.255.255.0
     no shutdown

    ip route 0.0.0.0 0.0.0.0 10.63.2.1

    interface Tunnel0
     description GRE-a-R1
     ip address 192.168.100.2 255.255.255.252
     tunnel source Ethernet0/1
     tunnel destination 10.63.1.1
     tunnel mode gre ip
     no shutdown

    ip route 192.168.63.0 255.255.255.0 192.168.100.1

---

### Configuración de las PCs

#### PC1

    ip 192.168.63.10 255.255.255.0 192.168.63.1
    save

#### PC2

    ip 192.168.64.10 255.255.255.0 192.168.64.1
    save

---

## Verificación

Primero se hizo **ping desde R1 hacia 10.63.2.2**, porque antes de probar el túnel era necesario confirmar que la WAN de **R1** tenía alcance hacia la WAN de **R2** a través del ISP.

    ping 10.63.2.2

Luego se hizo **ping desde R2 hacia 10.63.1.1**, para confirmar la conectividad WAN también en sentido contrario.

    ping 10.63.1.1

Después se hizo **ping desde R1 hacia 192.168.100.2**, porque esa es la IP de **Tunnel0** en **R2**. Esta prueba sirvió para comprobar que el túnel GRE quedó operativo.

    ping 192.168.100.2

Luego, en **R1**, se verificó el estado de las interfaces, la tabla de enrutamiento y la interfaz del túnel, para confirmar que **Tunnel0** estaba **up/up** y que la red remota se alcanzaba a través del túnel GRE.

    show ip interface brief
    show ip route
    show interfaces Tunnel0

Después se hizo **ping desde R2 hacia 192.168.100.1**, para validar el túnel también desde el otro extremo.

    ping 192.168.100.1

Luego, en **R2**, se revisó igualmente el estado de las interfaces, rutas e interfaz de túnel.

    show ip interface brief
    show ip route
    show interfaces Tunnel0

Posteriormente se hizo **ping desde PC1 hacia 192.168.64.10**, porque esta es la prueba principal de la práctica: demostrar que la LAN de **R1** puede alcanzar la LAN de **R2** a través del túnel GRE.

    PC1> ping 192.168.64.10

Después se revisó en **R1** la tabla de enrutamiento para confirmar que la red remota se estaba enviando por **192.168.100.2**, es decir, por el túnel.

    show ip route

Luego se hizo **ping desde PC2 hacia 192.168.63.10**, para demostrar conectividad en sentido contrario.

    PC2> ping 192.168.63.10

Finalmente, se mostró en ambos routers la configuración de **Tunnel0**, para dejar evidencia clara de que la práctica se realizó con **GRE** y no con VTI ni con policy-based.

    show run interface Tunnel0

---

## Conclusión

La práctica permitió implementar correctamente una **VPN site-to-site punto a punto con túnel GRE** entre dos routers Cisco en PNETLab, manteniendo la misma topología física y reutilizando el direccionamiento WAN y LAN previamente establecido.

La creación de **Tunnel0** en ambos extremos permitió encapsular y transportar el tráfico entre las redes **192.168.63.0/24** y **192.168.64.0/24** a través de la red intermedia, utilizando **GRE** como protocolo de túnel. A diferencia de las prácticas con **policy-based** o **route-based con VTI**, en esta implementación no se aplicó cifrado ni mecanismos de negociación IKE, por lo que el túnel funcionó únicamente como un medio de encapsulación.

La validación mediante **ping**, **show ip interface brief**, **show ip route**, **show interfaces Tunnel0** y **show run interface Tunnel0** confirmó que el túnel GRE quedó operativo y que la comunicación entre ambas sedes se realizó correctamente a través de la interfaz lógica configurada.
