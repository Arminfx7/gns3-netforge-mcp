# Configuración del ejercicio de red en GNS3

## 1. Topología final

La topología debe mantenerse con estas conexiones:

```text
WindowsXP-1 ── Switch1 ── R1 ── R2 ── Switch2 ── WindowsXP-2
                                  │
                               enlace directo
```

Conexiones utilizadas:

| Equipo A | Interfaz | Equipo B | Interfaz |
|---|---|---|---|
| R1 | FastEthernet0/0 | Switch1 | Ethernet0 |
| Switch1 | Ethernet1 | PC1 | Ethernet0 |
| Switch1 | Ethernet2 | WindowsXP-1 | Ethernet0 |
| R1 | FastEthernet0/1 | R2 | FastEthernet0/1 |
| R2 | FastEthernet0/0 | Switch2 | Ethernet0 |
| Switch2 | Ethernet1 | PC2 | Ethernet0 |
| Switch2 | Ethernet2 | WindowsXP-2 | Ethernet0 |

El enlace entre R1 y R2 es directo por FastEthernet, no por Serial.

## 2. Direccionamiento IP

### LAN izquierda: 192.168.0.0/24

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|---|---|---|---|---|
| R1 | FastEthernet0/0 | 192.168.0.1 | 255.255.255.0 | — |
| PC1 | Ethernet0 | 192.168.0.10 | 255.255.255.0 | 192.168.0.1 |
| WindowsXP-1 | Ethernet0 | 192.168.0.20 | 255.255.255.0 | 192.168.0.1 |

### Enlace entre routers: 10.0.0.0/8

| Dispositivo | Interfaz | Dirección IP | Máscara |
|---|---|---|---|
| R1 | FastEthernet0/1 | 10.0.0.1 | 255.0.0.0 |
| R2 | FastEthernet0/1 | 10.0.0.2 | 255.0.0.0 |

### LAN derecha: 192.168.1.0/24

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|---|---|---|---|---|
| R2 | FastEthernet0/0 | 192.168.1.1 | 255.255.255.0 | — |
| PC2 | Ethernet0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| WindowsXP-2 | Ethernet0 | 192.168.1.20 | 255.255.255.0 | 192.168.1.1 |

## 3. Configuración de R1

Abrir la consola de R1 y ejecutar cada bloque:

```text
enable
configure terminal
hostname R1

interface FastEthernet0/0
 description LAN-IZQUIERDA
 ip address 192.168.0.1 255.255.255.0
 no shutdown
 exit

interface FastEthernet0/1
 description ENLACE-R2
 ip address 10.0.0.1 255.0.0.0
 no shutdown
 exit

ip route 192.168.1.0 255.255.255.0 10.0.0.2
end
write memory
```

Verificación de R1:

```text
show ip interface brief
show ip route
ping 10.0.0.2
ping 192.168.1.1
```

## 4. Configuración de R2

Abrir la consola de R2 y ejecutar:

```text
enable
configure terminal
hostname R2

interface FastEthernet0/0
 description LAN-DERECHA
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit

interface FastEthernet0/1
 description ENLACE-R1
 ip address 10.0.0.2 255.0.0.0
 no shutdown
 exit

ip route 192.168.0.0 255.255.255.0 10.0.0.1
end
write memory
```

Verificación de R2:

```text
show ip interface brief
show ip route
ping 10.0.0.1
ping 192.168.0.1
```

Las interfaces conectadas deben aparecer como `up/up`.

## 5. Configuración de PC1 y PC2

En la consola de PC1:

```text
ip 192.168.0.10 255.255.255.0 192.168.0.1
save
show
ping 192.168.0.1
ping 192.168.1.10
```

En la consola de PC2:

```text
ip 192.168.1.10 255.255.255.0 192.168.1.1
save
show
ping 192.168.1.1
ping 192.168.0.10
```

## 6. Configuración de Windows XP

El adaptador virtual debe ser:

```text
PCnet-FAST III (Am79C973)
```

En Windows XP puede aparecer dentro del sistema como un adaptador AMD o como:

```text
Adaptador Ethernet PCI AMD PCnet
```

Esto es normal y confirma que el controlador corresponde al adaptador PCnet.

### WindowsXP-1

1. Abrir **Panel de control**.
2. Entrar en **Conexiones de red**.
3. Clic derecho en **Conexión de área local** → **Propiedades**.
4. Seleccionar **Protocolo de Internet TCP/IP** → **Propiedades**.
5. Elegir **Usar la siguiente dirección IP**.
6. Introducir:

```text
Dirección IP:       192.168.0.20
Máscara de subred:  255.255.255.0
Puerta de enlace:   192.168.0.1
DNS:                puede dejarse vacío para este ejercicio
```

### WindowsXP-2

Repetir los mismos pasos con:

```text
Dirección IP:       192.168.1.20
Máscara de subred:  255.255.255.0
Puerta de enlace:   192.168.1.1
DNS:                puede dejarse vacío para este ejercicio
```

Comprobar la configuración en cada XP desde CMD:

```cmd
ipconfig /all
```

## 7. Pruebas de conectividad

### Desde WindowsXP-1

```cmd
ping 192.168.0.1
ping 10.0.0.2
ping 192.168.1.1
ping 192.168.1.20
```

### Desde WindowsXP-2

```cmd
ping 192.168.1.1
ping 10.0.0.1
ping 192.168.0.1
ping 192.168.0.20
```

El resultado correcto debe mostrar respuestas con bytes y tiempo, por ejemplo:

```text
Respuesta desde 192.168.1.20: bytes=32 tiempo<1ms TTL=128
```

## 8. Orden recomendado para iniciar el laboratorio

1. Abrir GNS3.
2. Abrir el proyecto.
3. Confirmar que el servidor local está activo en `localhost:3080`.
4. Iniciar R1 y R2.
5. Iniciar Switch1 y Switch2.
6. Iniciar PC1 y PC2.
7. Iniciar WindowsXP-1 y WindowsXP-2.
8. Esperar a que Windows XP termine de arrancar.
9. Configurar las IP dentro de Windows XP.
10. Ejecutar las pruebas de ping.

## 9. Problemas comunes

### No responde el gateway

Comprobar primero en Windows XP:

```cmd
ipconfig
```

La dirección IP, máscara y gateway deben pertenecer a la misma LAN. XP-1 debe usar `192.168.0.0/24` y XP-2 debe usar `192.168.1.0/24`.

### El adaptador aparece como AMD

Es correcto. PCnet-FAST III es el modelo emulado por VirtualBox y Windows XP normalmente lo identifica como adaptador AMD PCnet.

### La interfaz del router aparece administratively down

Ejecutar en la interfaz correspondiente:

```text
enable
configure terminal
interface FastEthernet0/0
no shutdown
end
```

Repetir para FastEthernet0/1 si es necesario.

### R1 y R2 no se alcanzan

Verificar que el enlace sea exactamente:

```text
R1 FastEthernet0/1 <-> R2 FastEthernet0/1
```

Y comprobar:

```text
R1# ping 10.0.0.2
R2# ping 10.0.0.1
```

### El ping entre redes falla

Verificar las rutas estáticas:

En R1:

```text
show ip route 192.168.1.0
```

Debe existir una ruta hacia `192.168.1.0/24` vía `10.0.0.2`.

En R2:

```text
show ip route 192.168.0.0
```

Debe existir una ruta hacia `192.168.0.0/24` vía `10.0.0.1`.

## 10. Resultado esperado

Al finalizar:

- R1 y R2 deben estar conectados por FastEthernet.
- Las interfaces de los routers deben estar `up/up`.
- WindowsXP-1 debe poder hacer ping a WindowsXP-2.
- WindowsXP-2 debe poder hacer ping a WindowsXP-1.
- PC1 y PC2 también deben poder comunicarse entre sí.
- Ambos Windows XP deben usar el adaptador PCnet-FAST III (Am79C973).
