# VPN Hub and Spoke Punto a Multipunto DMVPN Fase 3 con IKEv2 y Enrutamiento Dinámico

**Autor:** Omar Cuevas  
**Asignatura:** Seguridad de Redes  
**Instituto:** ITLA  

---

## Descripción General

En esta práctica se implementó una **VPN hub and spoke punto a multipunto DMVPN Fase 3 con IKEv2**, utilizando **mGRE**, **NHRP**, **IPSec en modo transport** y **EIGRP** como protocolo de enrutamiento dinámico. La topología fue montada en **PNETLab** con un router central actuando como **Hub**, dos routers remotos actuando como **Spokes**, un router intermedio representando al **ISP**, y dos equipos finales conectados a las LAN de cada spoke.

El objetivo principal fue permitir conectividad segura y escalable entre múltiples sedes usando una nube DMVPN, pero añadiendo las características propias de **Fase 3**, donde el Hub puede enviar **NHRP Redirect** y los Spokes pueden instalar rutas directas usando **NHRP Shortcut**. Esto permite que el tráfico entre spokes deje de depender lógicamente del Hub y se optimice el camino.

En esta implementación:
- El **Hub** usa un túnel **mGRE**
- Los **Spokes** se registran dinámicamente mediante **NHRP**
- IPSec protege el tráfico GRE en **modo transport**
- El enrutamiento dinámico se realiza con **EIGRP AS 100**
- El **Hub** usa `ip nhrp redirect`
- Los **Spokes** usan `ip nhrp shortcut`
- Se utilizó **`ip mtu 1400`** en `Tunnel0`

La validación de la práctica se realizó correctamente mediante:
- **show dmvpn**
- **show ip nhrp**
- **show ip eigrp neighbor**
- **show ip route eigrp**
- **show crypto ikev2 sa**
- **show crypto ipsec sa**
- **Ping entre PC1 y PC2**
- **Cambio de ruta desde el Hub hacia el Spoke remoto después del primer ping**

---

## Objetivo

Configurar una **VPN DMVPN Fase 3 hub and spoke con IKEv2 y enrutamiento dinámico**, permitiendo que las redes **192.168.63.0/24** y **192.168.64.0/24** se comuniquen a través de una infraestructura multipunto protegida con IPSec, usando **EIGRP** para el intercambio automático de rutas y aprovechando **NHRP Redirect/Shortcut** para optimizar el tráfico spoke-to-spoke.

---

## Topología

HUB ---- ISP ---- SPOKE1  
HUB ---- ISP ---- SPOKE2  
PC1 ---- SPOKE1  
PC2 ---- SPOKE2  

<img width="713" height="304" alt="image" src="https://github.com/user-attachments/assets/217d7db4-3249-4d00-b0d2-61e99093bcd5" />

---

## Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Máscara | Descripción |
|------------|----------|--------------|---------|-------------|
| HUB | e0/1 | 10.63.1.1 | 255.255.255.252 | WAN hacia ISP |
| ISP | e0/1 | 10.63.1.2 | 255.255.255.252 | Enlace hacia HUB |
| ISP | e0/2 | 10.63.2.1 | 255.255.255.252 | Enlace hacia SPOKE1 |
| SPOKE1 | e0/1 | 10.63.2.2 | 255.255.255.252 | WAN hacia ISP |
| ISP | e0/3 | 10.63.3.1 | 255.255.255.252 | Enlace hacia SPOKE2 |
| SPOKE2 | e0/1 | 10.63.3.2 | 255.255.255.252 | WAN hacia ISP |
| SPOKE1 | e0/0 | 192.168.63.1 | 255.255.255.0 | LAN sitio 1 |
| SPOKE2 | e0/0 | 192.168.64.1 | 255.255.255.0 | LAN sitio 2 |
| HUB | Tunnel0 | 172.16.63.1 | 255.255.255.0 | Red DMVPN |
| SPOKE1 | Tunnel0 | 172.16.63.2 | 255.255.255.0 | Red DMVPN |
| SPOKE2 | Tunnel0 | 172.16.63.3 | 255.255.255.0 | Red DMVPN |
| PC1 | NIC | 192.168.63.10 | 255.255.255.0 | Host sitio 1 |
| PC2 | NIC | 192.168.64.10 | 255.255.255.0 | Host sitio 2 |

### Gateway de los hosts
- **PC1:** 192.168.63.1
- **PC2:** 192.168.64.1

---

## Parámetros de Seguridad y Operación

| Parámetro | Valor |
|----------|-------|
| Tecnología | DMVPN Fase 3 |
| Rol central | Hub |
| Rol remoto | Spokes |
| Versión IKE | IKEv2 |
| Autenticación | Pre-Shared Key |
| PSK | Cisco12345 |
| Cifrado IPSec | AES-256 |
| Integridad IPSec | SHA-256 |
| Modo IPSec | Transport |
| Tipo de túnel | mGRE |
| NHRP Network-ID | 100 |
| NHRP Authentication | nhrpkey |
| Red del túnel | 172.16.63.0/24 |
| Enrutamiento dinámico | EIGRP AS 100 |
| Comando clave en Hub | ip nhrp redirect |
| Comando clave en Spokes | ip nhrp shortcut |
| MTU del túnel | 1400 |

---

## Explicación de Funcionamiento

DMVPN Fase 3 combina **mGRE**, **NHRP**, **IPSec con IKEv2** y **EIGRP** para permitir conectividad segura y escalable entre múltiples sedes.

El funcionamiento en esta práctica fue el siguiente:

1. El **Hub** crea una interfaz `Tunnel0` de tipo **multipoint GRE**
2. Cada **Spoke** también crea su `Tunnel0` y se registra en el Hub usando **NHRP**
3. IPSec protege el tráfico GRE usando **modo transport**
4. **EIGRP** corre sobre la red del túnel y distribuye automáticamente las rutas entre el Hub y los Spokes
5. Inicialmente, un spoke aprende la red remota a través del **Hub**
6. Cuando un spoke intenta llegar a otro spoke, el **Hub** envía un **NHRP Redirect**
7. El spoke origen usa **NHRP Shortcut** para instalar una resolución más directa
8. A partir de ahí, el tráfico spoke-to-spoke se optimiza y evita depender lógicamente del Hub como siguiente salto

La diferencia principal frente a **DMVPN Fase 2** es que en Fase 3 la optimización del tráfico se basa en el mecanismo **Redirect + Shortcut**, lo que permite soportar mejor escenarios con sumarización y escalabilidad.

---

## Qué se Eliminó de la Práctica Anterior

Como previamente la topología estaba configurada en modo **DMVPN Fase 2 con IKEv1**, antes de montar esta práctica fue necesario eliminar la configuración anterior, incluyendo:

- `crypto isakmp policy`
- `crypto isakmp key`
- `router ospf 1`
- interfaz `Tunnel0` anterior
- `crypto ipsec profile` anterior
- `crypto ipsec transform-set` anterior
- parámetros NHRP de Fase 2
- cualquier dependencia de OSPF en el túnel

---

## Configuración de los Dispositivos

### PC1

    ip 192.168.63.10 255.255.255.0 192.168.63.1
    save

### PC2

    ip 192.168.64.10 255.255.255.0 192.168.64.1
    save

### ISP

    hostname ISP
    no ip domain-lookup

    interface Ethernet0/1
     description HACIA-HUB
     ip address 10.63.1.2 255.255.255.252
     no shutdown

    interface Ethernet0/2
     description HACIA-SPOKE1
     ip address 10.63.2.1 255.255.255.252
     no shutdown

    interface Ethernet0/3
     description HACIA-SPOKE2
     ip address 10.63.3.1 255.255.255.252
     no shutdown

    ip route 192.168.63.0 255.255.255.0 10.63.2.2
    ip route 192.168.64.0 255.255.255.0 10.63.3.2

### HUB

    hostname HUB
    no ip domain-lookup

    interface Ethernet0/1
     description WAN-ISP
     ip address 10.63.1.1 255.255.255.252
     no shutdown

    ip route 0.0.0.0 0.0.0.0 10.63.1.2

    crypto ikev2 proposal PROP-DMVPN
     encryption aes-cbc-256
     integrity sha256
     group 14

    crypto ikev2 policy POL-DMVPN
     proposal PROP-DMVPN

    crypto ikev2 keyring KR-DMVPN
     peer SPOKES
      address 0.0.0.0 0.0.0.0
      pre-shared-key Cisco12345

    crypto ikev2 profile PROF-DMVPN
     match identity remote address 0.0.0.0
     authentication remote pre-share
     authentication local pre-share
     keyring local KR-DMVPN

    crypto ipsec transform-set TS-DMVPN esp-aes 256 esp-sha256-hmac
     mode transport

    crypto ipsec profile PROF-IPSEC-DMVPN
     set transform-set TS-DMVPN
     set pfs group14
     set ikev2-profile PROF-DMVPN

    interface Tunnel0
     description DMVPN-HUB-F3
     ip address 172.16.63.1 255.255.255.0
     no ip redirects
     ip mtu 1400
     ip nhrp authentication nhrpkey
     ip nhrp network-id 100
     ip nhrp map multicast dynamic
     ip nhrp redirect
     tunnel source Ethernet0/1
     tunnel mode gre multipoint
     tunnel protection ipsec profile PROF-IPSEC-DMVPN
     no shutdown

    router eigrp 100
     network 172.16.63.0 0.0.0.255
     no auto-summary

### SPOKE1

    hostname SPOKE1
    no ip domain-lookup

    interface Ethernet0/1
     description WAN-ISP
     ip address 10.63.2.2 255.255.255.252
     no shutdown

    interface Ethernet0/0
     description LAN-PC1
     ip address 192.168.63.1 255.255.255.0
     no shutdown

    ip route 0.0.0.0 0.0.0.0 10.63.2.1

    crypto ikev2 proposal PROP-DMVPN
     encryption aes-cbc-256
     integrity sha256
     group 14

    crypto ikev2 policy POL-DMVPN
     proposal PROP-DMVPN

    crypto ikev2 keyring KR-DMVPN
     peer HUB
      address 0.0.0.0 0.0.0.0
      pre-shared-key Cisco12345

    crypto ikev2 profile PROF-DMVPN
     match identity remote address 0.0.0.0
     authentication remote pre-share
     authentication local pre-share
     keyring local KR-DMVPN

    crypto ipsec transform-set TS-DMVPN esp-aes 256 esp-sha256-hmac
     mode transport

    crypto ipsec profile PROF-IPSEC-DMVPN
     set transform-set TS-DMVPN
     set pfs group14
     set ikev2-profile PROF-DMVPN

    interface Tunnel0
     description DMVPN-SPOKE1-F3
     ip address 172.16.63.2 255.255.255.0
     no ip redirects
     ip mtu 1400
     ip nhrp authentication nhrpkey
     ip nhrp network-id 100
     ip nhrp nhs 172.16.63.1
     ip nhrp map 172.16.63.1 10.63.1.1
     ip nhrp map multicast 10.63.1.1
     ip nhrp shortcut
     tunnel source Ethernet0/1
     tunnel mode gre multipoint
     tunnel protection ipsec profile PROF-IPSEC-DMVPN
     no shutdown

    router eigrp 100
     network 172.16.63.0 0.0.0.255
     network 192.168.63.0 0.0.0.255
     no auto-summary

### SPOKE2

    hostname SPOKE2
    no ip domain-lookup

    interface Ethernet0/1
     description WAN-ISP
     ip address 10.63.3.2 255.255.255.252
     no shutdown

    interface Ethernet0/0
     description LAN-PC2
     ip address 192.168.64.1 255.255.255.0
     no shutdown

    ip route 0.0.0.0 0.0.0.0 10.63.3.1

    crypto ikev2 proposal PROP-DMVPN
     encryption aes-cbc-256
     integrity sha256
     group 14

    crypto ikev2 policy POL-DMVPN
     proposal PROP-DMVPN

    crypto ikev2 keyring KR-DMVPN
     peer HUB
      address 0.0.0.0 0.0.0.0
      pre-shared-key Cisco12345

    crypto ikev2 profile PROF-DMVPN
     match identity remote address 0.0.0.0
     authentication remote pre-share
     authentication local pre-share
     keyring local KR-DMVPN

    crypto ipsec transform-set TS-DMVPN esp-aes 256 esp-sha256-hmac
     mode transport

    crypto ipsec profile PROF-IPSEC-DMVPN
     set transform-set TS-DMVPN
     set pfs group14
     set ikev2-profile PROF-DMVPN

    interface Tunnel0
     description DMVPN-SPOKE2-F3
     ip address 172.16.63.3 255.255.255.0
     no ip redirects
     ip mtu 1400
     ip nhrp authentication nhrpkey
     ip nhrp network-id 100
     ip nhrp nhs 172.16.63.1
     ip nhrp map 172.16.63.1 10.63.1.1
     ip nhrp map multicast 10.63.1.1
     ip nhrp shortcut
     tunnel source Ethernet0/1
     tunnel mode gre multipoint
     tunnel protection ipsec profile PROF-IPSEC-DMVPN
     no shutdown

    router eigrp 100
     network 172.16.63.0 0.0.0.255
     network 192.168.64.0 0.0.0.255
     no auto-summary

---

## Verificación

Primero se verificó la conectividad WAN básica, ya que sin alcance IP entre el Hub, el ISP y los Spokes la nube DMVPN no puede formarse correctamente.

### Desde HUB

    ping 10.63.1.2

### Desde SPOKE1

    ping 10.63.2.1
    ping 10.63.1.1

### Desde SPOKE2

    ping 10.63.3.1
    ping 10.63.1.1

Después se revisó el estado general de DMVPN, para confirmar que los peers se registraron correctamente.

    show dmvpn

Luego se revisó NHRP, con el objetivo de verificar las resoluciones dinámicas y posteriormente observar los shortcuts.

    show ip nhrp

Después se revisaron los vecinos EIGRP, para confirmar que el enrutamiento dinámico quedó activo sobre la nube DMVPN.

    show ip eigrp neighbor

Luego se revisaron las rutas EIGRP en los Spokes, para comprobar que las redes remotas estaban siendo aprendidas.

    show ip route eigrp

Después se verificó la seguridad IKEv2/IPSec:

    show crypto ikev2 sa
    show crypto ipsec sa | include encaps|decaps

Finalmente se hizo la prueba principal desde **PC1** hacia **PC2**, usando múltiples repeticiones para evidenciar el comportamiento de Fase 3.

    PC1> ping 192.168.64.10 repeat 10

También se puede probar el retorno:

    PC2> ping 192.168.63.10 repeat 10

Para demostrar la diferencia con Fase 2, en **SPOKE1** primero se limpió NHRP:

    clear ip nhrp

Luego se revisó la ruta antes del ping:

    show ip route eigrp

En ese momento la red remota normalmente aparece vía el Hub.

Después del ping con múltiples repeticiones, se volvió a revisar:

    show ip route eigrp
    show ip nhrp

La evidencia esperada es que la resolución cambie y el spoke utilice una ruta más directa apoyada por **NHRP Redirect + Shortcut**.


---


## Conclusión

La práctica permitió implementar correctamente una **VPN hub and spoke DMVPN Fase 3 con IKEv2 y enrutamiento dinámico**, manteniendo una arquitectura centralizada con un **Hub** y dos sedes remotas conectadas como **Spokes**.

El uso de **mGRE**, **NHRP**, **EIGRP** e **IPSec en modo transport** hizo posible construir una red escalable y segura, donde los Spokes se registran dinámicamente en el Hub, aprenden rutas automáticamente y pueden optimizar el tráfico entre sí.

La incorporación de **`ip nhrp redirect`** en el Hub y **`ip nhrp shortcut`** en los Spokes permitió evidenciar el comportamiento característico de **DMVPN Fase 3**, donde el camino de tráfico spoke-to-spoke se vuelve más eficiente después del primer intercambio.

La validación mediante **show dmvpn**, **show ip nhrp**, **show ip eigrp neighbor**, **show ip route eigrp**, **show crypto ikev2 sa**, **show crypto ipsec sa** y los pings entre **PC1** y **PC2** confirmó que la red DMVPN quedó operativa y funcional.
