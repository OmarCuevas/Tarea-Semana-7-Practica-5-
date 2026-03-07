🔐 VPN Site-to-Site IPSec IKEv1 (Policy-Based)

Autor: Omar Cuevas
Asignatura: Seguridad de Redes
Instituto: ITLA

📘 Introducción

En esta práctica se implementó una VPN Site-to-Site utilizando IPSec con IKEv1 basado en políticas entre dos routers Cisco.

El objetivo principal es permitir la comunicación segura entre dos redes LAN remotas a través de una red pública simulada (ISP), utilizando cifrado para proteger la confidencialidad e integridad del tráfico.

La VPN se establece entre los routers R1 y R2, permitiendo que los dispositivos de ambas redes puedan comunicarse como si estuvieran en la misma red privada.

🎯 Objetivo de la VPN

Esta VPN tiene como finalidad:

Establecer un túnel seguro entre dos redes privadas

Proteger el tráfico utilizando IPSec

Utilizar IKEv1 para la negociación del túnel

Permitir comunicación segura entre:

192.168.63.0/24
192.168.64.0/24
🖧 Topología de la Red
PC1 ---- R1 ---- ISP ---- R2 ---- PC2

Cada router tiene una red LAN y se conecta al ISP mediante enlaces WAN.

🌐 Direccionamiento IP
LAN 1
Dispositivo	Interfaz	Dirección IP
PC1	-	192.168.63.10
R1	e0/2	192.168.63.1
LAN 2
Dispositivo	Interfaz	Dirección IP
PC2	-	192.168.64.10
R2	e0/2	192.168.64.1
Enlaces WAN
Dispositivo	Interfaz	Dirección IP
R1	e0/1	10.63.1.1
ISP	e0/1	10.63.1.2
R2	e0/1	10.63.2.2
ISP	e0/2	10.63.2.1

Máscara utilizada:

255.255.255.252
🔐 Parámetros de Seguridad
IKE Phase 1
Parámetro	Valor
Encryption	AES
Hash	SHA
Authentication	Pre-Shared Key
Diffie-Hellman	Group 2
Lifetime	86400

Clave compartida:

cisco
IPSec Phase 2
Parámetro	Valor
Transform Set	esp-aes esp-sha-hmac
Modo	Tunnel
ACL	VPN-TRAFFIC
📜 Tráfico Interesante

El tráfico que será cifrado está definido por la siguiente ACL:

permit ip 192.168.63.0 0.0.0.255 192.168.64.0 0.0.0.255

Esto indica que todo el tráfico entre ambas redes LAN será protegido por IPSec.

⚙️ Verificación del Túnel VPN

Para comprobar el funcionamiento del túnel VPN se utilizan los siguientes comandos.

Generar tráfico

Desde PC1:

ping 192.168.64.10
Verificar IKE (Fase 1)
show crypto isakmp sa

Resultado esperado:

QM_IDLE

Esto indica que la fase 1 del túnel está establecida correctamente.

Verificar IPSec (Fase 2)
show crypto ipsec sa

En este comando se pueden observar los contadores de:

packets encaps
packets decaps

Esto confirma que el tráfico está siendo cifrado y descifrado correctamente.

📸 Capturas de Pantalla Requeridas

La documentación debe incluir capturas de:

Topología en PNETLab

Configuración de R1

Configuración de R2

Resultado del ping entre PC1 y PC2

Resultado del comando:

show crypto isakmp sa

Resultado del comando:

show crypto ipsec sa

Estas capturas demuestran que el túnel VPN se estableció correctamente.

📌 Conclusión

La implementación de esta VPN permitió conectar de forma segura dos redes remotas utilizando IPSec con IKEv1.

Se comprobó que el tráfico entre ambas redes es cifrado antes de atravesar la red pública simulada, garantizando la confidencialidad e integridad de la información.

Este tipo de solución es ampliamente utilizado en entornos empresariales para conectar sucursales o sedes remotas de manera segura.
