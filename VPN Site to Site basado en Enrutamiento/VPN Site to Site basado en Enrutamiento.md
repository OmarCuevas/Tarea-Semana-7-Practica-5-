# VPN Site-to-Site Punto a Punto Basada en Enrutamiento con IKEv2 (Route-Based / VTI)

**Autor:** Omar Cuevas  
**Asignatura:** Seguridad de Redes  
**Instituto:** ITLA  

---

## Descripción General

En esta práctica se implementó una **VPN site-to-site punto a punto basada en enrutamiento** en un laboratorio de **PNETLab** utilizando routers **Cisco IOU**. La solución fue construida sobre la misma topología física que anteriormente se encontraba funcionando en modo **policy-based**, pero en esta ocasión se migró a una arquitectura **route-based** mediante una **VTI (Virtual Tunnel Interface)** protegida con **IKEv2 e IPSec**.

A diferencia de la VPN basada en políticas, en esta implementación el tráfico no se selecciona mediante una **crypto ACL** ni se utiliza **crypto map** aplicado sobre la interfaz WAN. En su lugar, se crea una **interfaz Tunnel0** en cada extremo y el tráfico hacia la red remota se envía por medio de rutas estáticas hacia dicha interfaz. Luego, el túnel es protegido con IPSec.

La validación de la práctica se realizó correctamente mediante:
- **Ping entre PC1 y PC2**
- **Ping entre las IP del túnel**
- **show crypto ikev2 sa**
- **show crypto ipsec sa**
- **show ip route**
- **show ip interface brief**

---

## Objetivo

Configurar una **VPN site-to-site punto a punto basada en enrutamiento con IKEv2** entre **R1** y **R2**, permitiendo la comunicación segura entre las redes **192.168.63.0/24** y **192.168.64.0/24** a través de una red intermedia no confiable, utilizando una **VTI (Virtual Tunnel Interface)** y rutas estáticas hacia la red remota.

---

## Topología

PC1 ---- R1 ---- ISP ---- R2 ---- PC2

<img width="1083" height="627" alt="image" src="https://github.com/user-attachments/assets/e6ae0796-b22b-4376-b787-8eac84bfa79e" />

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
| R1 | Tunnel0 | 172.16.63.1 | 255.255.255.252 | Extremo del túnel VTI |
| R2 | Tunnel0 | 172.16.63.2 | 255.255.255.252 | Extremo del túnel VTI |
| PC1 | NIC | 192.168.63.10 | 255.255.255.0 | Host sede 1 |
| PC2 | NIC | 192.168.64.10 | 255.255.255.0 | Host sede 2 |

### Gateway de los hosts
- **PC1:** 192.168.63.1
- **PC2:** 192.168.64.1

---

## Parámetros de Seguridad

| Parámetro | Valor |
|----------|-------|
| Versión IKE | IKEv2 |
| Autenticación | Pre-Shared Key |
| PSK | Cisco12345 |
| Cifrado IKEv2 | AES-CBC-256 |
| Integridad IKEv2 | SHA-256 |
| DH Group | 14 |
| Cifrado IPSec | ESP-AES-256 |
| Hash IPSec | ESP-SHA256-HMAC |
| Modo IPSec | Tunnel |
| Tipo de túnel | VTI |
| PFS | Group 14 |
| Crypto ACL | No requerida |
| Selección de tráfico | Basada en rutas |

---

## Explicación de Funcionamiento

En esta práctica la VPN funciona con una lógica **route-based**, lo que significa que el tráfico no se cifra porque coincida con una ACL, sino porque el router lo envía a través de una **interfaz de túnel**.

La interfaz **Tunnel0** se crea en ambos routers:
- En **R1**, con la IP **172.16.63.1/30**
- En **R2**, con la IP **172.16.63.2/30**

Cada túnel usa como origen la interfaz WAN local y como destino la IP WAN del peer remoto. Luego, el túnel es protegido con un **IPSec profile** enlazado a **IKEv2**.

El proceso de funcionamiento es el siguiente:

1. **PC1** intenta llegar a **PC2**
2. **R1** revisa su tabla de enrutamiento
3. La red **192.168.64.0/24** apunta hacia **172.16.63.2**, es decir, hacia el túnel
4. El tráfico entra por **Tunnel0**
5. El túnel utiliza **IPSec** para cifrar ese tráfico
6. Se negocia y mantiene la seguridad mediante **IKEv2**
7. **R2** recibe el tráfico, lo descifra y lo reenvía a la LAN **192.168.64.0/24**

La principal diferencia frente a la VPN basada en políticas es que aquí:
- **No existe ACL de tráfico interesante**
- **No existe crypto map aplicado sobre la WAN**
- **El tráfico es protegido porque se enruta por el túnel**

---

## Qué se Eliminó de la Práctica Anterior

Como previamente la topología estaba configurada en modo **policy-based**, antes de montar esta práctica fue necesario eliminar la configuración anterior, incluyendo:

- ACL 110 de tráfico interesante
- Crypto map aplicado en la interfaz WAN
- Transform-set anterior
- IKEv2 proposal anterior
- IKEv2 policy anterior
- IKEv2 keyring anterior
- IKEv2 profile anterior
- Rutas estáticas hacia la red remota por la WAN
- Cualquier Tunnel0 anterior, si existía

---

## Configuración de los Dispositivos

### ISP

El router ISP solo se utilizó para dar tránsito entre ambos extremos de la VPN.

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

    crypto ikev2 proposal PROP-IKEV2
     encryption aes-cbc-256
     integrity sha256
     group 14

    crypto ikev2 policy POL-IKEV2
     proposal PROP-IKEV2

    crypto ikev2 keyring KR-IKEV2
     peer R2
      address 10.63.2.2
      pre-shared-key Cisco12345

    crypto ikev2 profile PROF-IKEV2
     match identity remote address 10.63.2.2 255.255.255.255
     authentication remote pre-share
     authentication local pre-share
     keyring local KR-IKEV2

    crypto ipsec transform-set TS-VTI esp-aes 256 esp-sha256-hmac
     mode tunnel

    crypto ipsec profile PROF-VTI
     set transform-set TS-VTI
     set pfs group14
     set ikev2-profile PROF-IKEV2

    interface Tunnel0
     ip address 172.16.63.1 255.255.255.252
     tunnel source Ethernet0/1
     tunnel destination 10.63.2.2
     tunnel mode ipsec ipv4
     tunnel protection ipsec profile PROF-VTI
     no shutdown

    ip route 192.168.64.0 255.255.255.0 172.16.63.2

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

    crypto ikev2 proposal PROP-IKEV2
     encryption aes-cbc-256
     integrity sha256
     group 14

    crypto ikev2 policy POL-IKEV2
     proposal PROP-IKEV2

    crypto ikev2 keyring KR-IKEV2
     peer R1
      address 10.63.1.1
      pre-shared-key Cisco12345

    crypto ikev2 profile PROF-IKEV2
     match identity remote address 10.63.1.1 255.255.255.255
     authentication remote pre-share
     authentication local pre-share
     keyring local KR-IKEV2

    crypto ipsec transform-set TS-VTI esp-aes 256 esp-sha256-hmac
     mode tunnel

    crypto ipsec profile PROF-VTI
     set transform-set TS-VTI
     set pfs group14
     set ikev2-profile PROF-IKEV2

    interface Tunnel0
     ip address 172.16.63.2 255.255.255.252
     tunnel source Ethernet0/1
     tunnel destination 10.63.1.1
     tunnel mode ipsec ipv4
     tunnel protection ipsec profile PROF-VTI
     no shutdown

    ip route 192.168.63.0 255.255.255.0 172.16.63.1

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

Primero se realizó **ping desde PC1 hacia PC2**, porque esta es la prueba principal de la práctica. Con este ping se demuestra que un host de la red **192.168.63.0/24** puede comunicarse con un host de la red **192.168.64.0/24** atravesando el túnel VPN.

    PC1> ping 192.168.64.10

Después de ese ping, se revisó en **R1** el estado de **IKEv2** e **IPSec**, para confirmar que la negociación estaba activa y que los paquetes se estaban cifrando correctamente.

    show crypto ikev2 sa
    show crypto ipsec sa

Luego se realizó la misma verificación en **R2**, para demostrar que el otro extremo también mantenía la asociación de seguridad activa y recibía tráfico cifrado.

    show crypto ikev2 sa
    show crypto ipsec sa

Después se hizo **ping desde PC2 hacia PC1**, porque era importante demostrar conectividad también en sentido contrario.

    PC2> ping 192.168.63.10

Luego se volvió a revisar **IPSec** en **R1** y **R2**, verificando que los contadores de encapsulación y desencapsulación aumentaran con el tráfico generado desde ambos lados.

    show crypto ipsec sa

Posteriormente se hizo **ping desde R1 a 172.16.63.2**, porque esa es la IP de **Tunnel0** en R2. Esta prueba permite verificar directamente la conectividad entre extremos del túnel.

    ping 172.16.63.2

Después, en **R1**, se verificó el estado de las interfaces y la tabla de enrutamiento para confirmar que **Tunnel0** estaba **up/up** y que la red remota se alcanzaba a través del túnel.

    show ip interface brief
    show ip route

Luego se hizo **ping desde R2 a 172.16.63.1**, para validar el túnel también desde el otro extremo.

    ping 172.16.63.1

Después se revisó en **R2** el estado de sus interfaces y rutas.

    show ip interface brief
    show ip route

Finalmente, se mostró en ambos routers la configuración de **Tunnel0** y el estado de **IKEv2** e **IPSec** para dejar la evidencia completa de la práctica.

    show run interface tunnel0
    show crypto ikev2 sa
    show crypto ipsec sa

---

## Conclusión

La práctica permitió migrar correctamente una **VPN site-to-site basada en políticas** a una **VPN site-to-site basada en enrutamiento con IKEv2**, manteniendo la misma topología física y reutilizando el direccionamiento WAN y LAN previamente establecido.

La creación de la **VTI** en ambos extremos permitió enrutar el tráfico de las redes **192.168.63.0/24** y **192.168.64.0/24** a través del túnel, eliminando la necesidad de usar ACL de tráfico interesante y crypto maps. El túnel quedó protegido mediante **IPSec**, con negociación **IKEv2**, autenticación por **PSK**, cifrado **AES-256**, integridad **SHA-256** y **PFS Group 14**.

La validación mediante **ping**, **show crypto ikev2 sa**, **show crypto ipsec sa**, **show ip interface brief** y **show ip route** confirmó que la VPN quedó operativa y que el tráfico entre ambas sedes viaja cifrado correctamente a través del túnel.
