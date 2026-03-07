# VPN Client-to-Site L2TP/IPSec con IKEv1

**Autor:** Omar Cuevas  
**Asignatura:** Seguridad de Redes  
**Instituto:** ITLA  

---

## Descripción General

En esta práctica se implementó una **VPN client-to-site L2TP/IPSec con IKEv1** en un laboratorio de **PNETLab**, utilizando un cliente **Linux/Ubuntu QEMU**, un router intermedio actuando como **ISP**, un router Cisco actuando como **LNS (L2TP Network Server)** y un host interno dentro de la LAN corporativa.

El objetivo de esta práctica fue permitir que un cliente remoto se conectara de forma segura a la red corporativa usando **L2TP protegido por IPSec**, autenticándose con una **clave precompartida (PSK)** a nivel IPSec y con **usuario/contraseña PPP** a nivel L2TP/PPP. Una vez establecida la conexión, el cliente recibe una IP del pool VPN y obtiene acceso a la red interna.

La validación de la práctica se realizó correctamente mediante:
- **Establecimiento exitoso de IKEv1/IPSec**
- **Levantamiento de la interfaz `ppp0`**
- **Asignación de IP desde el pool VPN**
- **Ping desde Linux al gateway interno**
- **Ping desde Linux al host interno de la LAN**
- **show crypto isakmp sa**
- **show crypto ipsec sa**
- **show ip local pool**
- **show interfaces Virtual-Access1**

---

## Objetivo

Configurar una **VPN client-to-site L2TP/IPSec con IKEv1**, permitiendo que un cliente Linux remoto se conecte al router LNS, reciba una dirección IP del pool VPN y pueda acceder a la red interna **10.10.1.0/24**.

---

## Topología

Linux ---- ISP ---- R1-LNS ---- PC1

<img width="1274" height="715" alt="image" src="https://github.com/user-attachments/assets/e6154928-c8f6-4b05-9188-07fe92575440" />


---

## Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Máscara | Descripción |
|------------|----------|--------------|---------|-------------|
| Linux | e0 | Internet / Net | — | Solo para descargas |
| Linux | e1 | 200.0.0.6 | 255.255.255.252 | Enlace hacia ISP |
| ISP | e0/1 | 200.0.0.5 | 255.255.255.252 | Hacia Linux |
| ISP | e0/0 | 200.0.0.1 | 255.255.255.252 | Hacia LNS |
| R1-LNS | e0/0 | 200.0.0.2 | 255.255.255.252 | WAN |
| R1-LNS | e0/1 | 10.10.1.1 | 255.255.255.0 | LAN corporativa |
| PC1 | e0 | 10.10.1.100 | 255.255.255.0 | Host interno |
| Pool VPN | Virtual | 192.168.200.10 - 192.168.200.50 | — | Direcciones para clientes |
| Linux | ppp0 | 192.168.200.10 | Dinámica | IP asignada por la VPN |

### Gateway del host interno
- **PC1:** 10.10.1.1

---

## Parámetros de Seguridad

| Parámetro | Valor |
|----------|-------|
| Versión IKE | IKEv1 |
| Autenticación IPSec | Pre-Shared Key |
| PSK | IPSecL2TP |
| Cifrado IKEv1 | AES-256 |
| Hash IKEv1 | SHA |
| DH Group | 2 |
| Modo IPSec | Transport |
| Protocolo de túnel | L2TP |
| Puerto L2TP | UDP 1701 |
| Autenticación PPP | MS-CHAPv2 |
| Usuario VPN | vpnuser |
| Contraseña VPN | VpnPass123 |
| Pool de clientes | 192.168.200.10 – 192.168.200.50 |
| Interfaz virtual | Virtual-Template1 |

---

## Explicación de Funcionamiento

La conexión se establece en dos fases principales:

1. El cliente Linux primero negocia una **asociación IKEv1/IPSec** con el router LNS usando la PSK **IPSecL2TP**
2. Una vez que el canal IPSec queda activo, el cliente establece una sesión **L2TP**
3. Dentro del túnel L2TP se levanta una sesión **PPP**
4. El router autentica al usuario con **MS-CHAPv2**
5. El cliente recibe una IP del pool **192.168.200.10 – 192.168.200.50**
6. La interfaz `ppp0` aparece en Linux
7. El cliente ya puede acceder a la red interna **10.10.1.0/24**

En esta práctica se utilizó **IPSec en modo transport**, ya que **L2TP** es quien encapsula la sesión PPP y no se necesita un túnel IPSec completo en modo tunnel.

---

## Configuración de los Dispositivos

### PC1

    ip 10.10.1.100 255.255.255.0 10.10.1.1
    save

### ISP

    hostname ISP
    no ip domain-lookup

    interface Ethernet0/1
     description HACIA-LINUX
     ip address 200.0.0.5 255.255.255.252
     no shutdown

    interface Ethernet0/0
     description HACIA-LNS
     ip address 200.0.0.1 255.255.255.252
     no shutdown

    ip route 10.10.1.0 255.255.255.0 200.0.0.2
    ip route 192.168.200.0 255.255.255.0 200.0.0.2

### R1-LNS

    hostname R1-LNS
    no ip domain-lookup

    interface Ethernet0/0
     description WAN-ISP
     ip address 200.0.0.2 255.255.255.252
     no shutdown

    interface Ethernet0/1
     description LAN-CORPORATIVA
     ip address 10.10.1.1 255.255.255.0
     no shutdown

    ip route 0.0.0.0 0.0.0.0 200.0.0.1

    aaa new-model
    aaa authentication ppp VPN-AUTH local
    aaa authorization network VPN-AUTH local

    username vpnuser password VpnPass123

    ip local pool POOL-VPN 192.168.200.10 192.168.200.50

    crypto isakmp policy 10
     encr aes 256
     hash sha
     authentication pre-share
     group 2
     lifetime 86400

    crypto isakmp key IPSecL2TP address 0.0.0.0 0.0.0.0
    crypto isakmp nat keepalive 20

    crypto ipsec transform-set TS-L2TP esp-aes 256 esp-sha-hmac
     mode transport

    crypto dynamic-map DMAP-L2TP 10
     set transform-set TS-L2TP

    crypto map CMAP-L2TP 65535 ipsec-isakmp dynamic DMAP-L2TP

    interface Ethernet0/0
     crypto map CMAP-L2TP

    vpdn enable

    vpdn-group L2TP-SERVER
     accept-dialin
      protocol l2tp
      virtual-template 1
     no l2tp tunnel authentication

    interface Virtual-Template1
     ip unnumbered Ethernet0/1
     peer default ip address pool POOL-VPN
     ppp authentication ms-chap-v2 VPN-AUTH
     ppp authorization VPN-AUTH
     no shutdown

### Linux — IP del laboratorio

Se utilizó una segunda interfaz para el laboratorio, separada de la interfaz conectada a internet.

    ip addr add 200.0.0.6/30 dev e1
    ip link set e1 up

    ip route add 200.0.0.0/24 via 200.0.0.5 dev e1
    ip route add 10.10.1.0/24 via 200.0.0.5 dev e1
    ip route add 192.168.200.0/24 via 200.0.0.5 dev e1

### Linux — Instalación de paquetes

    apt update
    apt install strongswan xl2tpd ppp -y

### Linux — `/etc/ipsec.conf`

    config setup
        charondebug="ike 1, knl 1, cfg 0"

    conn L2TP-PSK
        authby=secret
        auto=add
        keyexchange=ikev1
        type=transport
        left=200.0.0.6
        leftprotoport=17/1701
        right=200.0.0.2
        rightprotoport=17/1701
        ike=aes256-sha1-modp1024!
        esp=aes256-sha1!

### Linux — `/etc/ipsec.secrets`

    : PSK "IPSecL2TP"

### Linux — `/etc/xl2tpd/xl2tpd.conf`

    [lac vpn-corporativa]
    lns = 200.0.0.2
    ppp debug = yes
    pppoptfile = /etc/ppp/options.l2tpd.client
    length bit = yes

### Linux — `/etc/ppp/options.l2tpd.client`

    ipcp-accept-local
    ipcp-accept-remote
    refuse-eap
    require-mschap-v2
    noccp
    noauth
    mtu 1280
    mru 1280
    noipdefault
    defaultroute
    usepeerdns
    connect-delay 5000
    name vpnuser
    password VpnPass123

### Linux — Levantar la conexión

    ipsec restart
    systemctl restart xl2tpd
    ipsec up L2TP-PSK
    echo "c vpn-corporativa" > /run/xl2tpd/l2tp-control
    sleep 3
    ip addr show ppp0

---

## Verificación

Primero se verificó que Linux utilizara la interfaz correcta del laboratorio para llegar al servidor LNS.

    ip route get 200.0.0.2

Después se reinició strongSwan y se levantó manualmente la conexión IPSec.

    ipsec restart
    ipsec up L2TP-PSK

La salida esperada debía mostrar:

    connection 'L2TP-PSK' established successfully

Luego se inició la sesión L2TP y se verificó la aparición de la interfaz `ppp0`.

    echo "c vpn-corporativa" > /run/xl2tpd/l2tp-control
    sleep 3
    ip addr show ppp0

En la práctica, la interfaz apareció correctamente con una IP del pool VPN, por ejemplo:

    inet 192.168.200.10 peer 10.10.1.1/32

Después se hizo ping al gateway interno del router LNS, para comprobar que el cliente VPN ya tenía conectividad hacia la red corporativa.

    ping -c 5 10.10.1.1

Luego se hizo ping al host interno **PC1**, verificando acceso completo a la LAN.

    ping -c 5 10.10.1.100

En el router LNS se verificó el estado de IKEv1:

    show crypto isakmp sa

Después se verificaron los contadores IPSec:

    show crypto ipsec sa | include encaps|decaps

También se verificó el uso del pool:

    show ip local pool POOL-VPN

Y por último, la creación de la interfaz virtual PPP:

    show interfaces Virtual-Access1
---

## Conclusión

La práctica permitió implementar correctamente una **VPN client-to-site L2TP/IPSec con IKEv1**, utilizando un cliente Linux como usuario remoto y un router Cisco como servidor **LNS**.

La combinación de **IPSec en modo transport**, **L2TP**, **PPP con MS-CHAPv2**, un **pool dinámico de direcciones** y una **interfaz virtual PPP** hizo posible que el cliente remoto se autenticara, recibiera una IP de la VPN y accediera con éxito a la red corporativa interna.

La validación mediante **ipsec up**, **ppp0**, **ping al gateway**, **ping al host interno**, **show crypto isakmp sa**, **show crypto ipsec sa**, **show ip local pool** y **show interfaces Virtual-Access1** confirmó que la conexión VPN quedó operativa y funcional.
