# VPN Site-to-Site IPSec IKEv1 (Policy Based)

**Autor:** Omar Cuevas  
**Asignatura:** Seguridad de Redes  
**Instituto:** ITLA  

---

## Tabla de Contenido
- [Descripción General](#descripción-general)
- [Objetivo del Laboratorio](#objetivo-del-laboratorio)
- [Topología del Laboratorio](#topología-del-laboratorio)
- [Direccionamiento IP](#direccionamiento-ip)
- [Parámetros de Seguridad](#parámetros-de-seguridad)
- [Funcionamiento de la VPN](#funcionamiento-de-la-vpn)
- [Verificación](#verificación)
- [Conclusión](#conclusión)

---

## Descripción General

Este laboratorio implementa una **VPN Site-to-Site utilizando IPSec con IKEv1 basado en políticas** entre dos routers Cisco.

El objetivo es permitir la comunicación segura entre dos redes LAN remotas a través de una red pública simulada (ISP).  
IPSec se utiliza para proporcionar **confidencialidad, integridad y autenticación del tráfico**, garantizando que la información transmitida entre ambas redes no pueda ser interceptada o modificada.

En esta implementación:

- **R1** protege la red `192.168.63.0/24`
- **R2** protege la red `192.168.64.0/24`
- El **ISP** actúa como red de tránsito

---

## Objetivo del Laboratorio

Los objetivos principales de este laboratorio son:

- Implementar una **VPN Site-to-Site entre dos routers Cisco**
- Comprender el funcionamiento de **IPSec y IKEv1**
- Configurar **políticas de seguridad**
- Verificar el establecimiento del túnel VPN
- Analizar el tráfico cifrado entre redes

Redes protegidas:


192.168.63.0/24
192.168.64.0/24


---

## Topología del Laboratorio


PC1 ---- R1 ---- ISP ---- R2 ---- PC2


Descripción de los dispositivos:

| Dispositivo | Función |
|-------------|--------|
| PC1 | Cliente en la red LAN 1 |
| R1 | Gateway VPN de la red 1 |
| ISP | Red pública simulada |
| R2 | Gateway VPN de la red 2 |
| PC2 | Cliente en la red LAN 2 |

### Captura de la Topología


(Aquí va screenshot de la topología en PNETLab)


---

## Direccionamiento IP

### LAN 1

| Dispositivo | Interfaz | Dirección IP |
|-------------|----------|-------------|
| PC1 | eth0 | 192.168.63.10 |
| R1 | e0/2 | 192.168.63.1 |

---

### LAN 2

| Dispositivo | Interfaz | Dirección IP |
|-------------|----------|-------------|
| PC2 | eth0 | 192.168.64.10 |
| R2 | e0/2 | 192.168.64.1 |

---

### Enlaces WAN

| Dispositivo | Interfaz | Dirección IP |
|-------------|----------|-------------|
| R1 | e0/1 | 10.63.1.1 |
| ISP | e0/1 | 10.63.1.2 |
| R2 | e0/1 | 10.63.2.2 |
| ISP | e0/2 | 10.63.2.1 |

Máscara utilizada:


255.255.255.252


---

## Parámetros de Seguridad

### IKE Phase 1

| Parámetro | Valor |
|----------|------|
| Encryption | AES |
| Hash | SHA |
| Authentication | Pre-Shared Key |
| Diffie-Hellman | Group 2 |
| Lifetime | 86400 |

Clave compartida:


cisco


---

### IPSec Phase 2

| Parámetro | Valor |
|----------|------|
| Transform Set | esp-aes esp-sha-hmac |
| Mode | Tunnel |
| ACL de tráfico interesante | VPN-TRAFFIC |

ACL utilizada:


permit ip 192.168.63.0 0.0.0.255 192.168.64.0 0.0.0.255


---

## Funcionamiento de la VPN

El establecimiento del túnel ocurre en las siguientes fases:

1. R1 inicia la negociación **IKE Phase 1** con R2.
2. Se establece una **Security Association (SA)** entre ambos routers.
3. Se negocia la **Phase 2 de IPSec**.
4. Se crea el túnel IPSec.
5. El tráfico entre las redes LAN es identificado por la ACL.
6. IPSec cifra el tráfico antes de enviarlo al ISP.
7. R2 descifra el tráfico y lo envía a la red interna.

---

## Verificación

### Generar tráfico

Desde PC1:


ping 192.168.64.10


### Captura del Ping


(Aquí va screenshot del ping funcionando)


---

### Verificar IKE Phase 1


show crypto isakmp sa


Resultado esperado:


QM_IDLE


### Captura del comando


(Aquí va screenshot de show crypto isakmp sa)


---

### Verificar IPSec


show crypto ipsec sa


Contadores importantes:


packets encaps
packets decaps


### Captura del comando


(Aquí va screenshot de show crypto ipsec sa)


---

## Conclusión

La implementación de esta VPN permitió establecer un túnel seguro entre dos redes remotas utilizando **IPSec con IKEv1**.

Durante el laboratorio se pudo observar cómo los routers negocian los parámetros de seguridad y cómo el tráfico entre ambas redes es cifrado antes de atravesar la red pública simulada.

Este tipo de solución es ampliamente utilizado en entornos empresariales para conectar **sucursales o sedes remotas de manera segura a través de Internet**.

---
