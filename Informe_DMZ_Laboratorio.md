# Informe de configuración de DMZ con Cisco Packet Tracer

### 1. Objetivo del laboratorio

Configurar una DMZ segura usando un router Cisco ISR, aplicando NAT y ACLs para controlar el tráfico entre LAN, DMZ y red externa.  
El objetivo principal fue permitir únicamente tráfico web hacia el servidor de la DMZ y proteger la LAN interna de accesos no autorizados.

---

### 2. Topología implementada

La red se diseñó con tres zonas:

- **LAN (192.168.1.0/24):** red interna con estaciones de trabajo.
- **DMZ (192.168.2.0/24):** red intermedia que aloja el servidor web.
- **Externa (200.1.1.0/24):** simula Internet, desde donde se accede al servidor.

Dispositivos usados:
- 1 Router Cisco ISR (Router_FW).
- 1 Servidor en la DMZ.
- 2 PCs (uno en LAN y otro en la red externa).

---

### 3. Plan de direccionamiento IP

| Dispositivo             | IP            | Máscara         | Gateway       |
|-------------------------|---------------|-----------------|---------------|
| PC_Internal             | 192.168.1.10  | 255.255.255.0   | 192.168.1.1   |
| Server_DMZ              | 192.168.2.10  | 255.255.255.0   | 192.168.2.1   |
| PC_External             | 200.1.1.10    | 255.255.255.0   | 200.1.1.1     |
| Router_FW Gi0/0 (LAN)   | 192.168.1.1   | 255.255.255.0   | —             |
| Router_FW Gi0/1 (DMZ)   | 192.168.2.1   | 255.255.255.0   | —             |
| Router_FW Gi0/2 (Ext)   | 200.1.1.1     | 255.255.255.0   | —             |

---

### 4. Configuración aplicada (resumen)

#### Interfaces

interface Gig0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface Gig0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown

interface Gig0/2
 ip address 200.1.1.1 255.255.255.0
 no shutdown

NAT
ip nat inside source static 192.168.2.10 192.168.3.1
ACLs

ACL 100 (Internet → DMZ):
access-list 100 permit tcp any host 192.168.3.1 eq 80
access-list 100 deny ip any any

interface Gig0/2
 ip access-group 100 in

ACL 110 (DMZ → LAN):
access-list 110 permit tcp 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255 established
access-list 110 permit icmp 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255 echo-reply
access-list 110 deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
access-list 110 permit ip any any

interface Gig0/1
 ip access-group 110 in

### 5. Verificaciones realizadas

✅ Ping desde PC_Internal hacia Router_FW (192.168.1.1) exitoso.

✅ Acceso web desde PC_External hacia servidor DMZ (http://192.168.3.1) exitoso.

✅ Bloqueo de acceso desde DMZ hacia PC_Internal (192.168.1.10) confirmado.

✅ Ping desde PC_Internal hacia servidor DMZ (192.168.2.10) exitoso.

❌ Ping desde servidor DMZ hacia PC_Internal denegado (esperado).

❌ Ping desde PC_External hacia servidor DMZ (192.168.3.1) denegado.

### 6. Conclusiones y recomendaciones
Este laboratorio permitió comprender cómo usar ACLs extendidas para controlar tráfico entre distintas zonas de seguridad.
La clave fue aplicar las ACLs en las interfaces correctas y en la dirección adecuada (inbound/outbound).
Además, se observó que las ACLs en Cisco son stateless, por lo que se necesitó usar la opción established y reglas de ICMP específicas para permitir respuestas a conexiones legítimas.

Recomendaciones:

Siempre probar conectividad básica (ping, gateway) antes de aplicar ACLs.

Documentar las ACLs con comentarios para mantener claridad en configuraciones grandes.

Usar NAT y ACLs en conjunto para simular entornos de seguridad reales.