# VPN Site-to-Site basada en Enrutamiento con IPSec IKEv1 (GRE over IPSec)

**Autor:** Omar Cuevas  
**Asignatura:** Seguridad de Redes  
**Instituto:** ITLA  

---

# Introducción

En este laboratorio se implementó una **VPN Site-to-Site punto a punto basada en enrutamiento utilizando IKEv1** entre dos routers Cisco.

La implementación utiliza un **túnel GRE protegido mediante IPSec**, lo que permite establecer una comunicación segura entre dos redes LAN remotas a través de una red pública simulada (ISP).

IPSec se encarga de proporcionar **confidencialidad, integridad y autenticación del tráfico**, mientras que GRE permite enrutar el tráfico entre las redes internas a través del túnel VPN.

Las redes conectadas mediante la VPN son:

192.168.63.0/24  
192.168.64.0/24

---

# Objetivo del Laboratorio

El objetivo de este laboratorio es configurar una **VPN Site-to-Site basada en enrutamiento** entre dos routers Cisco utilizando GRE sobre IPSec.

Esta implementación permite:

- Establecer comunicación segura entre dos redes LAN remotas
- Proteger el tráfico mediante IPSec
- Crear un túnel GRE entre los routers
- Verificar el funcionamiento del túnel VPN

---

# Topología del Laboratorio

La topología utilizada en este laboratorio conecta dos redes LAN a través de una red pública simulada.

PC1 ---- R1 ---- ISP ---- R2 ---- PC2

Descripción de los dispositivos:

| Dispositivo | Función |
|-------------|--------|
| PC1 | Cliente de la red LAN 1 |
| R1 | Gateway VPN de la red 1 |
| ISP | Red pública simulada |
| R2 | Gateway VPN de la red 2 |
| PC2 | Cliente de la red LAN 2 |

<img width="1226" height="576" alt="image" src="https://github.com/user-attachments/assets/62d0d2e5-e56d-4f1e-ba0b-9378a2511fc8" />

---

# Direccionamiento IP

## Red LAN 1

| Dispositivo | Interfaz | Dirección IP |
|-------------|----------|-------------|
| PC1 | eth0 | 192.168.63.10 |
| R1 | e0/2 | 192.168.63.1 |

---

## Red LAN 2

| Dispositivo | Interfaz | Dirección IP |
|-------------|----------|-------------|
| PC2 | eth0 | 192.168.64.10 |
| R2 | e0/2 | 192.168.64.1 |

---

## Enlaces WAN

| Dispositivo | Interfaz | Dirección IP |
|-------------|----------|-------------|
| R1 | e0/1 | 10.63.1.1 |
| ISP | e0/1 | 10.63.1.2 |
| R2 | e0/1 | 10.63.2.2 |
| ISP | e0/2 | 10.63.2.1 |

Máscara utilizada:

255.255.255.252

---

# Configuración del Túnel GRE

Para implementar una VPN basada en enrutamiento se configuró un **túnel GRE entre los routers R1 y R2**.

Red del túnel:

172.16.1.0/30

| Router | Interfaz | Dirección IP |
|------|------|------|
| R1 | Tunnel0 | 172.16.1.1 |
| R2 | Tunnel0 | 172.16.1.2 |

Este túnel permite enrutar tráfico entre ambas redes LAN.


---

# Parámetros de Seguridad IPSec

## IKE Phase 1

| Parámetro | Valor |
|----------|------|
| Encryption | AES |
| Hash | SHA |
| Authentication | Pre-Shared Key |
| Diffie-Hellman | Group 2 |
| Lifetime | 86400 |

Clave compartida utilizada:

cisco

---

## IPSec Phase 2

Transform Set utilizado:

esp-aes esp-sha-hmac

ACL utilizada para proteger el tráfico GRE:

permit gre host 10.63.1.1 host 10.63.2.2


---

# Funcionamiento de la VPN

El establecimiento del túnel VPN ocurre en las siguientes etapas:

1. R1 inicia la negociación de **IKE Phase 1** con R2.
2. Ambos routers establecen una **Security Association (SA)**.
3. Se negocia **IPSec Phase 2**.
4. Se establece el túnel GRE entre los routers.
5. IPSec cifra el tráfico del túnel GRE.
6. El tráfico cifrado atraviesa la red pública (ISP).
7. R2 recibe el tráfico, lo descifra y lo entrega a la red LAN.

---

# Verificación del Túnel VPN

Para comprobar el funcionamiento de la VPN se realizaron las siguientes pruebas.

---

## Verificar interfaz de túnel

show ip interface brief

Este comando permite verificar que la interfaz **Tunnel0 se encuentra activa (up/up)**.

<img width="847" height="218" alt="image" src="https://github.com/user-attachments/assets/5fe274f8-d7cf-4915-9018-525736e22ffd" />

---

## Verificar IKE

show crypto isakmp sa

Resultado esperado:

QM_IDLE

Esto indica que la fase 1 de IKE se estableció correctamente.

<img width="892" height="191" alt="image" src="https://github.com/user-attachments/assets/14fa76e9-5cfd-4e05-9c17-5045b7cecef3" />

---

## Verificar IPSec

show crypto ipsec sa

Contadores importantes:

packets encaps  
packets decaps

Estos valores indican que el tráfico está siendo cifrado y descifrado correctamente.

<img width="896" height="488" alt="image" src="https://github.com/user-attachments/assets/8a88328c-2553-472e-8271-56ea797e957d" />

---

## Prueba de conectividad

Desde PC1 se realiza una prueba de conectividad hacia PC2.

ping 192.168.64.10

Esto confirma que la comunicación entre ambas redes LAN se realiza correctamente a través del túnel VPN.

<img width="661" height="206" alt="image" src="https://github.com/user-attachments/assets/8140b6d8-1cf1-489e-952a-351f2a4c2158" />

---

# Conclusión

En este laboratorio se implementó una **VPN Site-to-Site basada en enrutamiento utilizando GRE sobre IPSec con IKEv1**.

Esta arquitectura permite enrutar tráfico entre redes remotas de forma segura y ofrece mayor flexibilidad que una VPN basada en políticas, ya que permite transportar múltiples redes dentro del túnel y utilizar protocolos de enrutamiento dinámico.

La implementación demostró cómo IPSec puede utilizarse para proteger el tráfico GRE y garantizar la seguridad de la comunicación entre redes remotas a través de una red pública.
