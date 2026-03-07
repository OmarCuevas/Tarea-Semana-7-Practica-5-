# VPN Site-to-Site Punto a Punto Basada en Políticas con IKEv2

**Autor:** Omar Cuevas  
**Asignatura:** Seguridad de Redes  
**Instituto:** ITLA  

---

## Descripción General

En esta práctica se implementó una **VPN site-to-site punto a punto basada en políticas con IKEv2** en un laboratorio de **PNETLab** utilizando routers **Cisco IOU**. La comunicación segura se estableció entre dos redes LAN remotas, conectadas a través de un router ISP que representa una red no confiable.

La solución fue construida con:

- **IKEv2** como protocolo de negociación
- **IPSec en modo túnel**
- **Clave precompartida (PSK)** para autenticación
- **ACL de tráfico interesante**
- **Crypto map** aplicado en la interfaz WAN de cada router

La validación de la práctica se realizó correctamente mediante:

- **Ping entre PC1 y PC2**
- **show crypto ikev2 sa**
- **show crypto ipsec sa**

---

## Objetivo

Configurar una **VPN site-to-site policy-based con IKEv2** entre **R1** y **R2**, permitiendo la comunicación segura entre las redes **192.168.63.0/24** y **192.168.64.0/24** a través de una red intermedia no confiable.

---

## Topología

    PC1 ---- R1 ---- ISP ---- R2 ---- PC2

<img width="1106" height="640" alt="image" src="https://github.com/user-attachments/assets/a580f93e-f93f-42c5-81fb-1b676f49ef47" />

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
| PFS | Group 14 |
| Selección de tráfico | Crypto ACL |

---

## Explicación de Funcionamiento

La VPN funciona identificando como **tráfico interesante** todo paquete que viaje entre la red **192.168.63.0/24** y la red **192.168.64.0/24**. Ese tráfico es definido mediante una ACL extendida y luego asociado a un **crypto map**.

Cuando un host de una sede intenta comunicarse con la otra sede, los routers:

1. Detectan que el tráfico coincide con la ACL de cifrado.
2. Inician la negociación **IKEv2**.
3. Autentican al peer remoto mediante **PSK**.
4. Construyen las asociaciones de seguridad IKEv2 e IPSec.
5. Encapsulan y cifran el tráfico usando **IPSec en modo túnel**.
6. Entregan el tráfico de forma segura al router remoto.
7. El router remoto descifra el tráfico y lo reenvía a su LAN.

En esta práctica no se utilizó GRE. La VPN es completamente **policy-based**, por lo que solo el tráfico definido en la ACL será protegido por IPSec.

---

## ACL de Tráfico Interesante

### En R1

    access-list 110 permit ip 192.168.63.0 0.0.0.255 192.168.64.0 0.0.0.255

### En R2

    access-list 110 permit ip 192.168.64.0 0.0.0.255 192.168.63.0 0.0.0.255


---

## Configuración de los Dispositivos

### ISP

Se configuró el direccionamiento de las interfaces y las rutas estáticas necesarias para dar conectividad entre ambos extremos.

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
    ip route 192.168.64.0 255.255.255.0 10.63.1.2

    access-list 110 permit ip 192.168.63.0 0.0.0.255 192.168.64.0 0.0.0.255

    crypto ikev2 proposal IKEV2-PROP
     encryption aes-cbc-256
     integrity sha256
     group 14

    crypto ikev2 policy IKEV2-POLICY
     proposal IKEV2-PROP

    crypto ikev2 keyring IKEV2-KEYRING
     peer R2
      address 10.63.2.2
      pre-shared-key Cisco12345

    crypto ikev2 profile IKEV2-PROFILE
     match identity remote address 10.63.2.2 255.255.255.255
     authentication remote pre-share
     authentication local pre-share
     keyring local IKEV2-KEYRING

    crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
     mode tunnel

    crypto map VPN-MAP 10 ipsec-isakmp
     set peer 10.63.2.2
     set transform-set TS
     set pfs group14
     set ikev2-profile IKEV2-PROFILE
     match address 110

    interface Ethernet0/1
     crypto map VPN-MAP

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
    ip route 192.168.63.0 255.255.255.0 10.63.2.1

    access-list 110 permit ip 192.168.64.0 0.0.0.255 192.168.63.0 0.0.0.255

    crypto ikev2 proposal IKEV2-PROP
     encryption aes-cbc-256
     integrity sha256
     group 14

    crypto ikev2 policy IKEV2-POLICY
     proposal IKEV2-PROP

    crypto ikev2 keyring IKEV2-KEYRING
     peer R1
      address 10.63.1.1
      pre-shared-key Cisco12345

    crypto ikev2 profile IKEV2-PROFILE
     match identity remote address 10.63.1.1 255.255.255.255
     authentication remote pre-share
     authentication local pre-share
     keyring local IKEV2-KEYRING

    crypto ipsec transform-set TS esp-aes 256 esp-sha256-hmac
     mode tunnel

    crypto map VPN-MAP 10 ipsec-isakmp
     set peer 10.63.1.1
     set transform-set TS
     set pfs group14
     set ikev2-profile IKEV2-PROFILE
     match address 110

    interface Ethernet0/1
     crypto map VPN-MAP

---

### Configuración de las PCs

#### PC1

    ip 192.168.63.10 255.255.255.0 192.168.63.1

#### PC2

    ip 192.168.64.10 255.255.255.0 192.168.64.1

---

## Verificación

### 1. Verificar conectividad base

Antes de probar la VPN, se debe confirmar que:

- R1 alcanza a 10.63.1.2
- R2 alcanza a 10.63.2.1
- R1 puede alcanzar a 10.63.2.2 a través del ISP
- R2 puede alcanzar a 10.63.1.1 a través del ISP
- Existe ruta hacia la red remota en ambos routers

### 2. Generar tráfico interesante

Desde **PC1**, hacer ping hacia **PC2**:

    ping 192.168.64.10

O desde **R1**:

    ping 192.168.64.10 source 192.168.63.1

### 3. Verificar asociación IKEv2

En R1 y R2:

    show crypto ikev2 sa
    
<img width="1162" height="262" alt="image" src="https://github.com/user-attachments/assets/e3fe33ec-65f7-482a-9364-b7dc6076cf11" />


### 4. Verificar asociación IPSec

En R1 y R2:

    show crypto ipsec sa

<img width="922" height="496" alt="image" src="https://github.com/user-attachments/assets/2696ce5a-7adc-4fbd-b390-47293bc97c1a" />

### 5. Verificar conectividad final

Se debe comprobar que **PC1** puede hacer ping a **PC2** y viceversa.

<img width="670" height="248" alt="image" src="https://github.com/user-attachments/assets/91711b83-f6f0-402b-93ec-b0856081043d" />

---

## Conclusión

La práctica permitió implementar correctamente una **VPN site-to-site policy-based con IKEv2** entre dos routers Cisco en PNETLab, utilizando una red intermedia como medio no confiable. La comunicación entre las redes **192.168.63.0/24** y **192.168.64.0/24** quedó protegida mediante **IPSec en modo túnel**, usando autenticación por **PSK**, cifrado **AES-256**, integridad **SHA-256** y **PFS grupo 14**.

La validación exitosa mediante **ping**, **show crypto ikev2 sa** y **show crypto ipsec sa** confirmó que la negociación y el cifrado del tráfico se realizaron correctamente. Esta práctica demuestra el funcionamiento de una VPN basada en políticas sin necesidad de túneles GRE, protegiendo únicamente el tráfico definido en la ACL de cifrado.
