# VPN Hub and Spoke Punto a Multipunto DMVPN Fase 2 con IKEv1 y Enrutamiento Dinámico

**Autor:** Omar Cuevas  
**Asignatura:** Seguridad de Redes  
**Instituto:** ITLA  

---

## Descripción General

En esta práctica se implementó una **VPN hub and spoke punto a multipunto con DMVPN Fase 2**, utilizando **IKEv1**, **mGRE**, **NHRP**, **IPSec en modo transport** y **OSPF** como protocolo de enrutamiento dinámico. La topología fue montada en **PNETLab** con un router central actuando como **Hub**, dos routers remotos actuando como **Spokes**, un router intermedio representando al **ISP**, y dos equipos finales conectados a las LAN de cada spoke.

El objetivo principal fue permitir que los spokes se registraran dinámicamente en el hub, establecieran conectividad a través de la nube DMVPN y aprendieran rutas de manera automática mediante OSPF. Al tratarse de **DMVPN Fase 2**, una vez que el hub resuelve la información NHRP, el tráfico **spoke-to-spoke** puede fluir de manera más eficiente sin depender permanentemente del hub para el reenvío de datos.

En esta implementación:
- El **Hub** usa un túnel **mGRE**
- Los **Spokes** se registran dinámicamente mediante **NHRP**
- IPSec protege el tráfico GRE en **modo transport**
- El enrutamiento dinámico se realiza con **OSPF área 0**
- Se utilizaron **`ip mtu 1400`** y **`ip ospf mtu-ignore`** en `Tunnel0` para evitar problemas de MTU en IOU

La validación de la práctica se realizó correctamente mediante:
- **show dmvpn**
- **show ip nhrp**
- **show ip ospf neighbor**
- **show ip route ospf**
- **show crypto isakmp sa**
- **show crypto ipsec sa**
- **Ping entre PC1 y PC2**

---

## Objetivo

Configurar una **VPN DMVPN Fase 2 hub and spoke con IKEv1 y enrutamiento dinámico**, permitiendo que las redes **192.168.63.0/24** y **192.168.64.0/24** se comuniquen a través de una infraestructura multipunto protegida con IPSec, usando **OSPF** para el intercambio automático de rutas.

---

## Topología

HUB ---- ISP ---- SPOKE1  
HUB ---- ISP ---- SPOKE2  
PC1 ---- SPOKE1  
PC2 ---- SPOKE2  


<img width="1087" height="428" alt="image" src="https://github.com/user-attachments/assets/3fb109cc-6809-4d84-b142-fb096357f292" />

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
| Tecnología | DMVPN Fase 2 |
| Rol central | Hub |
| Rol remoto | Spokes |
| Versión IKE | IKEv1 |
| Autenticación | Pre-Shared Key |
| PSK | Cisco12345 |
| Cifrado IPSec | AES-256 |
| Integridad IPSec | SHA-256 |
| Modo IPSec | Transport |
| Tipo de túnel | mGRE |
| NHRP Network-ID | 1 |
| NHRP Authentication | nhrpkey |
| Red del túnel | 172.16.63.0/24 |
| Enrutamiento dinámico | OSPF Área 0 |
| Tipo de red OSPF | point-to-multipoint |
| MTU del túnel | 1400 |

---

## Explicación de Funcionamiento

DMVPN combina **mGRE**, **NHRP**, **IPSec** y **enrutamiento dinámico** para permitir conectividad escalable entre múltiples sedes. En esta práctica:

1. El **Hub** crea una interfaz `Tunnel0` de tipo **multipoint GRE**
2. Cada **Spoke** crea también su `Tunnel0` y se registra en el hub usando **NHRP**
3. El **Hub** actúa como servidor NHRP y conoce la relación entre la IP del túnel y la IP NBMA real de cada spoke
4. **OSPF** corre sobre la red del túnel y permite el intercambio automático de rutas entre el hub y los spokes
5. **IPSec** protege el tráfico GRE usando **modo transport**, ya que GRE ya se encarga de la encapsulación lógica
6. Cuando un spoke necesita llegar a otro, el hub resuelve la información NHRP y permite el establecimiento del flujo correspondiente dentro de la nube DMVPN

La diferencia frente a una VPN punto a punto tradicional es que aquí no se crean manualmente túneles separados entre todas las sedes. En su lugar, la red DMVPN permite escalar mejor y soportar comunicación dinámica entre spokes.

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

    crypto isakmp policy 10
     encr aes 256
     hash sha256
     authentication pre-share
     group 14
     lifetime 86400

    crypto isakmp key Cisco12345 address 0.0.0.0 0.0.0.0

    crypto ipsec transform-set TS-DMVPN esp-aes 256 esp-sha256-hmac
     mode transport

    crypto ipsec profile PROF-DMVPN
     set transform-set TS-DMVPN

    interface Tunnel0
     description DMVPN-HUB-F2
     ip address 172.16.63.1 255.255.255.0
     no ip redirects
     ip mtu 1400
     ip nhrp authentication nhrpkey
     ip nhrp network-id 1
     ip nhrp map multicast dynamic
     ip ospf network point-to-multipoint
     ip ospf mtu-ignore
     tunnel source Ethernet0/1
     tunnel mode gre multipoint
     tunnel protection ipsec profile PROF-DMVPN
     no shutdown

    router ospf 1
     router-id 1.1.1.1
     network 172.16.63.0 0.0.0.255 area 0

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

    crypto isakmp policy 10
     encr aes 256
     hash sha256
     authentication pre-share
     group 14
     lifetime 86400

    crypto isakmp key Cisco12345 address 0.0.0.0 0.0.0.0

    crypto ipsec transform-set TS-DMVPN esp-aes 256 esp-sha256-hmac
     mode transport

    crypto ipsec profile PROF-DMVPN
     set transform-set TS-DMVPN

    interface Tunnel0
     description DMVPN-SPOKE1-F2
     ip address 172.16.63.2 255.255.255.0
     no ip redirects
     ip mtu 1400
     ip nhrp authentication nhrpkey
     ip nhrp network-id 1
     ip nhrp nhs 172.16.63.1
     ip nhrp map 172.16.63.1 10.63.1.1
     ip nhrp map multicast 10.63.1.1
     ip ospf network point-to-multipoint
     ip ospf mtu-ignore
     tunnel source Ethernet0/1
     tunnel mode gre multipoint
     tunnel protection ipsec profile PROF-DMVPN
     no shutdown

    router ospf 1
     router-id 2.2.2.2
     network 172.16.63.0 0.0.0.255 area 0
     network 192.168.63.0 0.0.0.255 area 0

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

    crypto isakmp policy 10
     encr aes 256
     hash sha256
     authentication pre-share
     group 14
     lifetime 86400

    crypto isakmp key Cisco12345 address 0.0.0.0 0.0.0.0

    crypto ipsec transform-set TS-DMVPN esp-aes 256 esp-sha256-hmac
     mode transport

    crypto ipsec profile PROF-DMVPN
     set transform-set TS-DMVPN

    interface Tunnel0
     description DMVPN-SPOKE2-F2
     ip address 172.16.63.3 255.255.255.0
     no ip redirects
     ip mtu 1400
     ip nhrp authentication nhrpkey
     ip nhrp network-id 1
     ip nhrp nhs 172.16.63.1
     ip nhrp map 172.16.63.1 10.63.1.1
     ip nhrp map multicast 10.63.1.1
     ip ospf network point-to-multipoint
     ip ospf mtu-ignore
     tunnel source Ethernet0/1
     tunnel mode gre multipoint
     tunnel protection ipsec profile PROF-DMVPN
     no shutdown

    router ospf 1
     router-id 3.3.3.3
     network 172.16.63.0 0.0.0.255 area 0
     network 192.168.64.0 0.0.0.255 area 0

---

## Verificación

Primero se verificó la conectividad WAN básica, ya que sin alcance IP entre el hub, el ISP y los spokes, la nube DMVPN no puede formarse correctamente.

### Desde HUB

    ping 10.63.1.2

### Desde SPOKE1

    ping 10.63.2.1
    ping 10.63.1.1

### Desde SPOKE2

    ping 10.63.3.1
    ping 10.63.1.1

Después se revisó en el **HUB** el estado de DMVPN, porque ahí se debe confirmar que los spokes se hayan registrado correctamente mediante NHRP.

    show dmvpn

Luego se revisó la base NHRP en el **HUB**, para verificar qué túneles y direcciones NBMA fueron aprendidos dinámicamente.

    show ip nhrp

Después se revisaron los vecinos OSPF en el **HUB**, para confirmar que ambos spokes establecieron adyacencia sobre `Tunnel0`.

    show ip ospf neighbor

Luego se verificó en **SPOKE1** y **SPOKE2** la tabla de rutas OSPF, para comprobar que cada spoke aprendió dinámicamente la red LAN remota.

    show ip route ospf

Después se revisó la seguridad IKEv1/IPSec, para confirmar que las asociaciones se levantaron correctamente.

    show crypto isakmp sa
    show crypto ipsec sa | include encaps|decaps

Finalmente se hizo la prueba principal de la práctica, realizando **ping desde PC1 hacia PC2**, con el fin de demostrar conectividad entre las LAN remotas a través de la nube DMVPN.

    PC1> ping 192.168.64.10

También se puede hacer la prueba en sentido contrario:

    PC2> ping 192.168.63.10

---

## Conclusión

La práctica permitió implementar correctamente una **VPN hub and spoke DMVPN Fase 2 con IKEv1 y enrutamiento dinámico**, manteniendo una arquitectura centralizada con un **Hub** y dos sedes remotas conectadas como **Spokes**.

El uso de **mGRE**, **NHRP**, **OSPF** e **IPSec en modo transport** hizo posible construir una red escalable y segura, donde los spokes pueden registrarse dinámicamente en el hub y aprender rutas automáticamente. La utilización de **OSPF point-to-multipoint** sobre el túnel permitió intercambiar rutas sin necesidad de configuraciones estáticas adicionales dentro de la nube DMVPN.

La validación mediante **show dmvpn**, **show ip nhrp**, **show ip ospf neighbor**, **show ip route ospf**, **show crypto isakmp sa**, **show crypto ipsec sa** y los pings entre **PC1** y **PC2** confirmó que la red DMVPN quedó operativa y funcional.
