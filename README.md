# Topologia1_IKEv1_PolicyBased

https://www.youtube.com/watch?v=1I3OG6CFe0c

Instituto Tecnologico
Documentacion Tecnica - Configuracion de VPN
VPN Site-to-Site IPSec IKEv1
Basada en Politicas (Policy-Based / Crypto Map)
Estudiante: Darwing Manuel Pena Reyes
Matricula: 2024-2690
Asignatura: Ethical Hacking 2
Fecha: Julio 2026
 1. Objetivo de la VPN
Establecer una conexion VPN segura punto a punto entre dos sedes (LAN de R2 y LAN de R3) a traves de una red publica no confiable (representada por el router ISP), cifrando unicamente el trafico que coincide con una lista de acceso (ACL) definida explicitamente. Este metodo permite un control granular de que trafico se protege, siendo la forma clasica de implementar IPSec en dispositivos Cisco antes de la adopcion de interfaces de tunel virtual.
2. Topologia de red
La topologia consta de dos routers peer (R2 y R3), cada uno con su propia LAN y un switch de acceso, interconectados a traves de un router ISP intermedio que unicamente conoce las redes publicas /30, sin ninguna ruta hacia las redes privadas. El cifrado se activa bajo demanda cuando se detecta trafico que coincide con la ACL VPN-TRAFFIC.
2.1 Interfaces utilizadas
Dispositivo / Interfaz	Rol
R2 - F0/0	WAN, hacia ISP, con crypto map aplicado
R2 - G2/0	LAN, hacia switch y PC1
R3 - F1/0	WAN, hacia ISP, con crypto map aplicado
R3 - G2/0	LAN, hacia switch y PC2
ISP - F0/0 / F1/0	Enlaces publicos hacia R2 y R3 respectivamente
<img width="833" height="591" alt="image" src="https://github.com/user-attachments/assets/64265e24-b0ca-48c6-a12b-80a3f6d015f7" />

2.2 Direccionamiento IP
Segmento	Red / IP
LAN R2	20.24.26.0/24 - Gateway .90 - PC1 .91
LAN R3	20.24.27.0/24 - Gateway .90 - PC2 .91
Enlace ISP-R2	172.16.26.0/30 (R2 .1 / ISP .2)
Enlace ISP-R3	172.16.27.0/30 (ISP .1 / R3 .2)


Explicacion: Captura general de la topologia armada en GNS3, mostrando todos los dispositivos, sus interfaces conectadas, y el recuadro con nombre y matricula del estudiante.
3. Parametros utilizados
Parametro	Valor configurado
Version IKE	IKEv1 (ISAKMP)
Metodo de autenticacion	Pre-shared key (CiscoVPN123)
Cifrado Fase 1	AES 256
Hash Fase 1	SHA256
Grupo Diffie-Hellman	Grupo 14
Transform-set (Fase 2)	esp-aes 256 + esp-sha256-hmac
Modo IPSec	Tunnel
Mecanismo de seleccion de trafico	ACL extendida (policy-based)

4. Configuracion y explicacion (capturas de pantalla)
Configuracion de ISAKMP (Fase 1) en R2
crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14
crypto isakmp key CiscoVPN123 address 172.16.27.2

Explicacion: Se define la politica de negociacion de Fase 1: cifrado AES-256, hash SHA256, autenticacion por llave pre-compartida y grupo Diffie-Hellman 14. La llave se asocia unicamente a la IP publica del peer remoto (172.16.27.2), ya que en este escenario el peer es fijo.
ACL y Crypto Map (mecanismo policy-based)
ip access-list extended VPN-TRAFFIC
 permit ip 20.24.26.0 0.0.0.255 20.24.27.0 0.0.0.255
crypto map MAP-IKEV1 10 ipsec-isakmp
 set peer 172.16.27.2
 set transform-set TS-V1
 match address VPN-TRAFFIC

Explicacion: La ACL VPN-TRAFFIC define exactamente que trafico debe cifrarse (de la LAN de R2 hacia la LAN de R3). El crypto map referencia esa ACL con 'match address', y se aplica sobre la interfaz fisica de salida (F0/0). Este es el elemento que distingue una VPN policy-based de una route-based.
5. Evidencia de funcionamiento
Estado de la SA de Fase 1 (ISAKMP)
dst             src             state          conn-id status
172.16.27.2     172.16.26.1     QM_IDLE           1001 ACTIVE
El estado QM_IDLE con status ACTIVE confirma que la negociacion de Fase 1 se completo correctamente entre ambos peers.
Prueba de conectividad desde PC1
trace 20.24.27.91
 1   20.24.26.90   29.222 ms
 2     *  *  *
 3   *20.24.27.91   179.694 ms (Destination port unreachable)
El segundo salto no responde (* * *) porque el trafico va cifrado dentro del tunel IPSec y no existe una interfaz logica visible para el traceroute; el tercer salto confirma que el paquete llego completo al destino final.
6. Conclusion
La VPN IPSec IKEv1 policy-based quedo funcional y verificada: la Fase 1 (ISAKMP) establecio el canal seguro, la Fase 2 (IPSec) cifro el trafico definido por la ACL, y las pruebas de ping/traceroute desde las PCs confirmaron conectividad extremo a extremo entre ambas LANs a traves de la red publica simulada por el router ISP.
