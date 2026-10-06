# Ejercicio GNS3: dos LAN con enlace serial `Serial1/0/0`

Este documento describe una configuración completa y guardable para GNS3 con dos routers c3600, dos switches, dos VPCS y dos máquinas Windows XP.

El enlace entre los routers debe ser serial:

```text
R1 Serial1/0/0 <------------> R2 Serial1/0/0
```

El error mostrado en la imagen no es un error de direccionamiento IP. Es un error de VirtualBox causado por una cadena de snapshots o discos diferenciales que ya no existe:

```text
A differencing image of snapshot could not be found
Could not find an open hard disk with UUID
```

Por eso primero se debe crear un proyecto GNS3 nuevo y utilizar dos VMs de VirtualBox independientes, con UUID diferentes y discos completos o cadenas de snapshots válidas.

## 1. Topología

```text
WindowsXP-1 ── Switch1 ── R1 ═══ serial ═══ R2 ── Switch2 ── WindowsXP-2
                         │                  │
                        PC1                PC2
```

Conexiones:

| Equipo A | Interfaz | Equipo B | Interfaz |
|---|---|---|---|
| R1 | FastEthernet0/0 | Switch1 | Ethernet0 |
| Switch1 | Ethernet1 | PC1 | Ethernet0 |
| Switch1 | Ethernet2 | WindowsXP-1 | Ethernet0 |
| R1 | Serial1/0/0 | R2 | Serial1/0/0 |
| R2 | FastEthernet0/0 | Switch2 | Ethernet0 |
| Switch2 | Ethernet1 | PC2 | Ethernet0 |
| Switch2 | Ethernet2 | WindowsXP-2 | Ethernet0 |

No se debe conectar R1 con R2 mediante UDP, Cloud o enlace Ethernet si el ejercicio solicita `Serial1/0/0`.

## 2. Direccionamiento IP

### LAN izquierda: `192.168.0.0/24`

| Dispositivo | Interfaz | IP | Máscara | Gateway |
|---|---|---|---|---|
| R1 | FastEthernet0/0 | 192.168.0.1 | 255.255.255.0 | — |
| PC1 | Ethernet0 | 192.168.0.10 | 255.255.255.0 | 192.168.0.1 |
| WindowsXP-1 | Ethernet0 | 192.168.0.20 | 255.255.255.0 | 192.168.0.1 |

### Enlace serial: `10.0.0.0/30`

Para un enlace punto a punto se recomienda `/30`, aunque el ejercicio original puede indicar `10.0.0.0/8`.

| Dispositivo | Interfaz | IP | Máscara |
|---|---|---|---|
| R1 | Serial1/0/0 | 10.0.0.1 | 255.255.255.252 |
| R2 | Serial1/0/0 | 10.0.0.2 | 255.255.255.252 |

Si el profesor exige literalmente la red `10.0.0.0/8`, sustituir la máscara serial por `255.0.0.0` en ambos routers. Las IP `10.0.0.1` y `10.0.0.2` seguirán funcionando.

### LAN derecha: `192.168.1.0/24`

| Dispositivo | Interfaz | IP | Máscara | Gateway |
|---|---|---|---|---|
| R2 | FastEthernet0/0 | 192.168.1.1 | 255.255.255.0 | — |
| PC2 | Ethernet0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| WindowsXP-2 | Ethernet0 | 192.168.1.20 | 255.255.255.0 | 192.168.1.1 |

## 3. Creación correcta de los routers seriales

1. Crear un proyecto nuevo en GNS3.
2. Arrastrar dos routers `c3600`.
3. Abrir la configuración de cada router.
4. Agregar un módulo serial compatible, por ejemplo `WIC-1T` o el módulo serial disponible para el c3600.
5. Confirmar que aparezca la interfaz:

```text
Serial1/0/0
```

6. Conectar R1 y R2 usando el enlace serial desde esa interfaz exacta.

Si GNS3 solo muestra `Serial1/0` o `Serial0/0`, utilizar el nombre que realmente muestra la interfaz en la consola. No inventar el nombre: debe coincidir con `show ip interface brief`.

## 4. Configuración de R1

```text
enable
configure terminal
hostname R1

interface FastEthernet0/0
 description LAN-IZQUIERDA
 ip address 192.168.0.1 255.255.255.0
 no shutdown
 exit

interface Serial1/0/0
 description ENLACE-SERIAL-A-R2
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit

ip route 192.168.1.0 255.255.255.0 10.0.0.2
end
write memory
```

`clock rate` solamente debe configurarse en el extremo DCE. Para identificar el extremo DCE:

```text
show controllers serial 1/0/0
```

Si R1 no es DCE, eliminar esa línea de R1 y configurarla en R2:

```text
configure terminal
interface Serial1/0/0
 clock rate 64000
end
write memory
```

## 5. Configuración de R2

```text
enable
configure terminal
hostname R2

interface FastEthernet0/0
 description LAN-DERECHA
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit

interface Serial1/0/0
 description ENLACE-SERIAL-A-R1
 ip address 10.0.0.2 255.255.255.252
 no shutdown
 exit

ip route 192.168.0.0 255.255.255.0 10.0.0.1
end
write memory
```

## 6. Verificación del enlace serial

En ambos routers:

```text
show ip interface brief
```

El resultado esperado es:

```text
FastEthernet0/0  192.168.0.1  up  up
Serial1/0/0      10.0.0.1     up  up
```

En R2 las direcciones deben ser `192.168.1.1` y `10.0.0.2`.

Pruebas:

```text
R1# ping 10.0.0.2
R1# ping 192.168.1.1

R2# ping 10.0.0.1
R2# ping 192.168.0.1
```

Si aparece `up/down`, revisar el `clock rate` en el extremo DCE. Si aparece `administratively down`, ejecutar `no shutdown`.

## 7. Configuración de PC1 y PC2

En PC1:

```text
ip 192.168.0.10 255.255.255.0 192.168.0.1
save
show
ping 192.168.0.1
ping 192.168.1.10
```

En PC2:

```text
ip 192.168.1.10 255.255.255.0 192.168.1.1
save
show
ping 192.168.1.1
ping 192.168.0.10
```

## 8. Configuración de Windows XP

En las dos máquinas virtuales utilizar:

```text
PCnet-FAST III (Am79C973)
```

Windows XP puede mostrarlo como:

```text
Adaptador Ethernet PCI AMD PCnet
```

Eso es correcto.

### WindowsXP-1

Panel de control → Conexiones de red → Conexión de área local → Propiedades → Protocolo TCP/IP.

```text
IP:       192.168.0.20
Máscara:  255.255.255.0
Gateway:  192.168.0.1
```

### WindowsXP-2

```text
IP:       192.168.1.20
Máscara:  255.255.255.0
Gateway:  192.168.1.1
```

Comprobar desde CMD:

```cmd
ipconfig /all
ping 192.168.0.1
ping 192.168.1.1
```

## 9. Pruebas finales

Desde WindowsXP-1:

```cmd
ping 192.168.0.1
ping 10.0.0.2
ping 192.168.1.1
ping 192.168.1.20
```

Desde WindowsXP-2:

```cmd
ping 192.168.1.1
ping 10.0.0.1
ping 192.168.0.1
ping 192.168.0.20
```

## 10. Cómo evitar el error de VirtualBox y UUID

### No copiar manualmente una VM

No copiar ni duplicar a mano archivos `.vbox`, `.vdi` o carpetas de snapshots. Eso puede producir:

```text
has the same UUID as an existing virtual machine
```

### Crear dos VMs independientes

1. Apagar completamente la VM original.
2. En VirtualBox, seleccionar **Clonar**.
3. Elegir **Clon completo**.
4. Seleccionar **Generar nuevas direcciones MAC**.
5. Crear dos máquinas diferentes:

```text
WindowsXP-1
WindowsXP-2
```

6. Confirmar que ambas tengan UUID diferentes:

```cmd
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" list vms
```

Debe aparecer algo similar a:

```text
"WindowsXP-1" {UUID-DIFERENTE-1}
"WindowsXP-2" {UUID-DIFERENTE-2}
```

### Evitar snapshots rotos

El error:

```text
A differencing image of snapshot could not be found
```

significa que falta un archivo de snapshot o disco diferencial. Para evitarlo:

- No mover las carpetas de las VMs después de registrarlas.
- No borrar archivos `.vdi` dentro de `Snapshots`.
- No abrir un proyecto que referencia una VM eliminada.
- Preferir clones completos para este ejercicio.
- Registrar las VMs en VirtualBox antes de agregarlas a GNS3.
- En GNS3, seleccionar las VMs registradas desde **Preferences → VirtualBox VMs**.
- Usar una VM distinta para WindowsXP-1 y WindowsXP-2.

Si la cadena de snapshot ya está dañada, no se corrige cambiando el UUID. Hay que restaurar el archivo faltante o crear una VM nueva desde una imagen XP válida.

## 11. Guardado correcto

1. Crear el proyecto con **File → New blank project**.
2. Guardarlo con un nombre nuevo, por ejemplo:

```text
Ejercicio_Serial_GNS3
```

3. Guardar el proyecto con `Ctrl+S` después de crear la topología.
4. En cada router ejecutar:

```text
write memory
```

5. En cada VPCS ejecutar:

```text
save
```

6. Cerrar GNS3 desde **File → Quit**, no eliminando manualmente la carpeta del proyecto.
7. Volver a abrir el archivo `.gns3` desde GNS3 para verificar que la topología se conserva.

## 12. Error UDP de Dynamips

El mensaje `unable to create UDP NIO` indica que un puerto UDP ya está ocupado o que existe un enlace UDP antiguo.

Para este ejercicio no se necesita un enlace UDP entre R1 y R2. El enlace debe ser serial. Si el mensaje continúa:

1. Detener todos los routers.
2. Eliminar cualquier enlace UDP o Cloud que no pertenezca a la topología.
3. Cerrar el proyecto.
4. Cerrar GNS3.
5. Volver a abrir GNS3 y el proyecto guardado.
6. Crear nuevamente solo el enlace serial `Serial1/0/0`.

No cambiar UUIDs de discos como solución al error UDP; son problemas independientes.

## Resultado esperado

- R1 y R2 están conectados por `Serial1/0/0`.
- El enlace serial aparece `up/up`.
- Las rutas estáticas aparecen en `show ip route`.
- PC1 y PC2 se alcanzan entre sí.
- WindowsXP-1 y WindowsXP-2 usan PCnet-FAST III.
- Las dos VMs tienen UUID diferentes.
- El proyecto se guarda como un archivo `.gns3` sin depender de snapshots faltantes.
