Documentación Técnica
VPN Site-to-Site IPSec IKEv1 (Policy Based)

Autor: Omar Cuevas
Asignatura: Seguridad de Redes
Instituto: ITLA

1. Introducción

En esta práctica se implementó una VPN Site-to-Site utilizando IPSec con IKEv1 basado en políticas entre dos routers Cisco.

El objetivo principal es permitir la comunicación segura entre dos redes LAN remotas a través de una red pública simulada (ISP), utilizando cifrado para proteger la confidencialidad e integridad del tráfico.

La VPN se establece entre los routers R1 y R2, permitiendo que los dispositivos de ambas redes puedan comunicarse como si estuvieran en la misma red privada.

2. Objetivo de la VPN

El objetivo de esta VPN es:

Establecer un túnel seguro entre dos redes privadas

Proteger el tráfico mediante IPSec

Utilizar IKEv1 para la negociación del túnel

Permitir comunicación segura entre:

192.168.63.0/24
y
192.168.64.0/24

Esto permite que los equipos de ambas LAN puedan comunicarse de forma cifrada a través de una red no confiable.

3. Topología de la Red

La topología utilizada en esta práctica es la siguiente:

PC1 ---- R1 ---- ISP ---- R2 ---- PC2

Cada router tiene una red local y se conecta al ISP mediante enlaces WAN.

4. Direccionamiento IP
Red LAN 1
Dispositivo	Interfaz	Dirección IP
PC1	-	192.168.63.10
R1	e0/2	192.168.63.1
Red LAN 2
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
5. Parámetros de Seguridad Utilizados

Los parámetros utilizados para establecer el túnel IPSec son:

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
ACL de tráfico interesante	VPN-TRAFFIC
6. Tráfico Interesante

El tráfico que será cifrado está definido por la siguiente ACL:

permit ip 192.168.63.0 0.0.0.255 192.168.64.0 0.0.0.255

Esto significa que todo el tráfico entre ambas redes LAN será protegido por IPSec.

7. Configuración de los Dispositivos
Router R1

(Aquí puedes poner captura de pantalla del router)

Explicación:

R1 se encarga de iniciar el túnel IPSec hacia R2.
También contiene la configuración del crypto map, la política IKE y la ACL de tráfico interesante.

Captura sugerida:

show run

configuración de crypto isakmp policy

crypto map

Router R2

(Aquí puedes poner captura de pantalla del router)

Explicación:

R2 recibe la negociación IPSec desde R1 y establece el túnel seguro entre ambas redes.

Captura sugerida:

show run

configuración de crypto map

crypto isakmp key

8. Verificación de la VPN

Para verificar el funcionamiento de la VPN se utilizaron los siguientes comandos.

Generar tráfico

Desde PC1:

ping 192.168.64.10

Esto inicia la negociación del túnel IPSec.

Verificar IKE
show crypto isakmp sa

Resultado esperado:

QM_IDLE

Esto indica que la fase 1 del túnel está establecida.

Verificar IPSec
show crypto ipsec sa

En este comando se pueden observar los contadores de:

packets encaps
packets decaps

Esto indica que el tráfico está siendo cifrado y descifrado correctamente.
