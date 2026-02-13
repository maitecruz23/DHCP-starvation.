📚 README.md - ATAQUE 1: DHCP STARVATION
📋 PRÁCTICA DE ATAQUES DE RED CON SCAPY
Matrícula: 20241165
Fecha: Febrero 2025
Entorno: PNELAB / EVE-NG
Red: 20.24.116.0/24



ATAQUE 1: DHCP STARVATION
📌 OBJETIVO DEL SCRIPT
Objetivo General:
Agotar el pool de direcciones IP del servidor DHCP legítimo mediante la generación masiva de solicitudes DHCP Discover con direcciones MAC falsificadas, provocando una denegación de servicio (DoS) en la asignación dinámica de direcciones IP.

Objetivos Específicos:

Saturar el servidor DHCP legítimo (20.24.116.4) con 254 peticiones simultáneas

Ocupar todas las direcciones IP disponibles en el pool 20.24.116.0/24

Impedir que la víctima (Windows) obtenga una dirección IP válida

Demostrar la vulnerabilidad de servidores DHCP sin mecanismos de protección

Resultado Esperado:

Pool DHCP: 0/254 direcciones disponibles

Víctima: IP 169.254.x.x (APIPA - Sin conectividad)

Servidor DHCP: 100% de utilización




🏗️ TOPOLOGÍA DE RED


<img width="2048" height="1231" alt="image" src="https://github.com/user-attachments/assets/c835e483-f6ba-4639-93b3-cdff1f56968e" /> 






Tabla de Direccionamiento IP:

<img width="1642" height="445" alt="image" src="https://github.com/user-attachments/assets/4c569c4b-4c7d-444a-9957-f2ca19b1300a" />



Configuración de VLANs:

<img width="1082" height="146" alt="image" src="https://github.com/user-attachments/assets/3d0d4438-8155-4631-a75f-d0e5636be361" />





⚙️ PARÁMETROS USADOS EN EL ATAQUE
Configuración del Script:

<img width="1067" height="680" alt="image" src="https://github.com/user-attachments/assets/48d6826e-21a2-4aec-b1a5-8b6bebb792ea" />




Pool DHCP Legítimo (Router-2):

<img width="944" height="701" alt="image" src="https://github.com/user-attachments/assets/4fe52908-45f9-4c21-bbb0-2f7ae1a1617b" />










Captura 1: Estado Inicial - Antes del Ataque
<img width="874" height="442" alt="image" src="https://github.com/user-attachments/assets/99f9311e-6071-46c2-a3fb-c9913e71ea42" />

1.2 Windows con IP legítima:
<img width="949" height="471" alt="image" src="https://github.com/user-attachments/assets/ff631cb8-c8cd-4631-bfa4-cbcdf60fb197" />

Captura 2: Durante el Ataque
<img width="898" height="575" alt="image" src="https://github.com/user-attachments/assets/0387e62a-418a-4324-9bc1-9c7c2883e25f" />

En el router dhcp
<img width="898" height="503" alt="image" src="https://github.com/user-attachments/assets/eb4aa1e1-f3ac-46ae-9ae3-02d4d7f3e08e" />


En la victima
<img width="938" height="460" alt="image" src="https://github.com/user-attachments/assets/6710e8bc-95b9-4dc9-832f-387305e34ddc" />







💻 REQUISITOS PARA UTILIZAR LA HERRAMIENTA
# Sistema Operativo
- Kali Linux 2024.x o superior
- Python 3.8 o superior

# Dependencias Python
pip3 install scapy
pip3 install netifaces
pip3 install threading

# Instalación en Kali
sudo apt update
sudo apt install python3-scapy python3-pip -y
sudo apt install tcpdump wireshark -y  # Opcional para debugging










Requisitos de Hardware/Red:
<img width="1062" height="546" alt="image" src="https://github.com/user-attachments/assets/ebaff53a-39da-4e78-9d20-b39f7c30cfd4" />





Configuración Previa Necesaria:


EN KALI LINUK:
# 1. Configurar IP estática
sudo ip addr add 20.24.116.2/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 20.24.116.1

# 2. Activar modo promiscuo
sudo ip link set eth0 promisc on

# 3. Verificar conectividad
ping -c 2 20.24.116.1
ping -c 2 20.24.116.4

En Router-2 (Servidor DHCP):
ip dhcp pool POOL_LEGITIMO_20241165
 network 20.24.116.0 255.255.255.0
 default-router 20.24.116.1
 dns-server 8.8.8.8
 lease 1

 En Windows (Víctima):
netsh interface ip set address "eth0" dhcp
netsh interface ip set dns "eth0" dhcp






🛡️ MEDIDAS DE MITIGACIÓN


1. PORT SECURITY (Capa 2 - Switch)

Configuración en Switch Cisco:
! Limitar número de MACs por puerto
interface GigabitEthernet0/2  ! Puerto de Kali
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

! Verificar configuración
show port-security
show port-security interface GigabitEthernet0/2




Efectividad: ❌ NO mitiga DHCP Starvation directamente
Propósito: Previene suplantación de MAC, pero no el ataque masivo





2. DHCP SNOOPING (Capa 2 - Switch)
   
Configuración en Switch Cisco:
! Activar DHCP Snooping global
ip dhcp snooping
ip dhcp snooping vlan 1

! Configurar puertos confiables (Trusted)
interface GigabitEthernet0/3  ! Puerto del servidor DHCP legítimo
 ip dhcp snooping trust

! Configurar límite de peticiones en puertos no confiables
interface GigabitEthernet0/2  ! Puerto de Kali
 ip dhcp snooping limit rate 5  ! Máximo 5 peticiones por segundo

! Verificar configuración
show ip dhcp snooping
show ip dhcp snooping binding




Efectividad: ✅✅ ALTA (50-70%)
Ventajas: Limita velocidad de peticiones DHCP
Desventajas: Requiere switch administrable





3. RATE LIMITING (Capa 2 - Switch)
   
Configuración en Switch Cisco:
! Limitar broadcast/multicast
interface GigabitEthernet0/2
 storm-control broadcast level 0.5  ! 0.5% del ancho de banda
 storm-control multicast level 0.5
 storm-control action shutdown

! Verificar
show storm-control

! Limitar broadcast/multicast
interface GigabitEthernet0/2
 storm-control broadcast level 0.5  ! 0.5% del ancho de banda
 storm-control multicast level 0.5
 storm-control action shutdown

! Verificar
show storm-control





Efectividad: ✅✅ ALTA (60-80%)
Propósito: Limita tráfico DHCP que es broadcast


4. CONFIGURACIÓN DE POOL DHCP (Capa 3 - Router)
   
En Router-2 (Servidor DHCP):
! Limitar número máximo de leases por MAC
ip dhcp pool POOL_LEGITIMO_20241165
 lease 0 0 30  ! Lease time reducido a 30 minutos

! Configurar manualmente binding máximo
ip dhcp pool POOL_LEGITIMO_20241165
 utilization mark high 80  ! Alerta al 80% de uso
 utilization mark low 20

! Verificar
show ip dhcp server statistics




Efectividad: ⚠️ BAJA (10-20%)
Ventajas: Reduce ventana de ataque
Desventajas: No previene, solo mitiga






5. VLAN SEGMENTATION (Capa 2 - Switch)
Configuración en Switch Cisco:

! Crear VLANs separadas
vlan 10
 name CLIENTES_LEGITIMOS

vlan 20
 name ZONA_DMZ

vlan 99
 name NATIVA

! Asignar puertos a VLANs específicas
interface GigabitEthernet0/1  ! Windows Víctima
 switchport mode access
 switchport access vlan 10

interface GigabitEthernet0/2  ! Kali Atacante
 switchport mode access
 switchport access vlan 20  ! Aislado

interface GigabitEthernet0/3  ! Router DHCP
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99

 
 
 
 
Efectividad: ✅✅✅ MUY ALTA (80-90%)
Ventajas: Aísla completamente al atacante
Desventajas: Requiere re-diseño de red





6. AAA + 802.1X (Capa 2 - Switch)
Configuración en Switch Cisco:

! Activar autenticación 802.1X
aaa new-model
aaa authentication dot1x default group radius
dot1x system-auth-control

interface GigabitEthernet0/2
 authentication port-control auto
 dot1x pae authenticator

! Configurar servidor RADIUS
radius-server host 20.24.116.100 key Practica2024





Efectividad: ✅✅✅✅ MUY ALTA (95%+)
Ventajas: Autenticación de dispositivos antes de asignar IP
Desventajas: Complejo de implementar






7. MONITOREO Y DETECCIÓN
   
Script de detección en Python:
#!/usr/bin/env python3
from scapy.all import *
import time

def detectar_starvation():
    """Detecta posibles ataques DHCP Starvation"""
    umbral = 50  # 50 peticiones por minuto
    contador = 0
    inicio = time.time()
    
    def contar_pkt(pkt):
        nonlocal contador
        if DHCP in pkt and pkt[DHCP].options[0][1] == 1:
            contador += 1
    
    print("[*] Monitoreando DHCP Discover...")
    sniff(prn=contar_pkt, filter="udp port 67", timeout=60)
    
    if contador > umbral:
        print(f"[!] ALERTA: {contador} DHCP Discover en 60 segundos")
        print("[!] Posible ataque DHCP Starvation detectado")
    else:
        print(f"[*] Tráfico normal: {contador} peticiones")

if __name__ == "__main__":
    detectar_starvation()



   
    
Efectividad: ✅✅✅ ALTA (70-80%)
Ventajas: Detecta ataque en curso
Desventajas: No previene, solo alerta








8. MEJORES PRÁCTICAS - CHECKLIST DE SEGURIDAD
   
<img width="1393" height="808" alt="image" src="https://github.com/user-attachments/assets/69f287e6-6269-4e7f-a602-ae6eb59588fc" />









9. CONFIGURACIÓN RECOMENDADA - SEGURIDAD MÁXIMA

! ============================================
! CONFIGURACIÓN SEGURA CONTRA DHCP STARVATION
! ============================================

! GLOBAL
ip dhcp snooping
ip dhcp snooping vlan 1,10,20
no ip dhcp snooping information option

! INTERFAZ SERVIDOR DHCP (TRUSTED)
interface GigabitEthernet0/3
 description R2-DHCP-SERVER
 switchport mode access
 switchport access vlan 10
 ip dhcp snooping trust
 spanning-tree portfast

! INTERFAZ CLIENTES (UNTRUSTED)
interface GigabitEthernet0/1
 description WIN-VICTIMA
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 ip dhcp snooping limit rate 5
 spanning-tree bpduguard enable
 spanning-tree portfast

! INTERFAZ AISLADA (ZONA DE RIESGO)
interface GigabitEthernet0/2
 description KALI-ATACANTE
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 ip dhcp snooping limit rate 2
 spanning-tree bpduguard enable
 spanning-tree portfast
 shutdown  ! Activar solo cuando necesario














 📊 CONCLUSIÓN DEL ATAQUE

<img width="1380" height="737" alt="image" src="https://github.com/user-attachments/assets/a943134f-5ab3-4640-998b-8a38b6717456" />


Lecciones Aprendidas:
✅ DHCP es vulnerable a ataques de saturación sin DHCP Snooping

✅ 254 peticiones en 5 segundos son suficientes para agotar un pool /24

✅ La víctima queda completamente aislada sin servicio DHCP

✅ Las medidas de mitigación DEBEN implementarse en el switch, no en el router










⚠️ ADVERTENCIA LEGAL
Este documento es únicamente con fines educativos.
Realizado en entorno controlado de laboratorio (PNELAB/EVE-NG).
No debe ser utilizado en redes de producción sin autorización explícita.

Realizado por: Estudiante de Seguridad Informática
Matrícula: 20241165
Fecha: Febrero 2025
Entorno: PNELAB








📁 ARCHIVOS ADJUNTOS

Archivo	                                 Descripción
dhcp_starvation_20241165.py	        Script principal del ataque
topologia.pnet	                    Script de topología PNELAB
config_router1.txt	                Configuración R1-GATEWAY
config_router2.txt	                Configuración R2-DHCP
config_switch.txt	                  Configuración SW-PRACTICA
capturas/                        	  capturas de pantalla












