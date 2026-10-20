# 02-diagrama-red

## Objetivo
En esta fase del proyecto voy a decidir qué redes va a usar la VM de Proxmox, en mi caso uso UTM para virtualizarla, debo de decidir qué redes configuro y qué IP tiene la máquina. La idea es poder gestionar Proxmox desde mi Mac y que Proxmox y las máquinas invitadas tengan salida a la red.

## Procedimiento
Para este laboratorio necesito que Proxmox esté conectado a dos redes, la primera es la red 'host only'. Host only la he configurado con la red 192.168.20.0/24, esta red la usaré para conectarme a Proxmox vía HTTPS por el puerto TCP:8006. La segunda es red NAT o en mi caso (uso UTM en vez de VirtualBox) 'red compartida'. Que en UTM se llame red compartida no implica que sea adaptador puente, hace una red NAT la cual he configurado para que sea 10.0.2.0/24.

### Configuración de la VM
En estas capturas muestro el estado de la configuración de las interfaces de la VM.

![Red Host Only](../evidencias/capturas-o-salidas/red_hostonly_utm.png)
![Red NAT](../evidencias/capturas-o-salidas/red_compartida_utm.png)

### Interfaces y bridges de Proxmox

| Interfaz / bridge | Red | IP del nodo | Para qué la uso |
|---|---|---|---|
| Interfaz 1 (host-only) → `vmbr0` | Host-only 192.168.20.0/24 | 192.168.20.10 | Esta interfaz me permite conectarme a la interfaz web de Proxmox desde mi Mac vía HTTPS por el puerto TCP:8006. En este bridge también van `srv-aso` y `cli-linux` |
| Interfaz 2 (red compartida) → `vmbr1` | Red compartida (NAT) 10.0.2.0/24 | 10.0.2.100 | Esta interfaz da salida a Internet a Proxmox. Las VMs salen a Internet a través de Proxmox, que hace NAT hacia esta red |

Las máquinas invitadas van en la red host-only (`vmbr0`) porque así desde mi Mac puedo llegar a ellas directamente para probar y administrar.


### Tabla de direccionamiento

| Elemento | Nombre | IP | Función | Recursos |
|---|---|---|---|---|
| Nodo Proxmox | `pve-pico` | 192.168.20.10 (gestión) / 10.0.2.100 | Hipervisor | 2 vCPU, 6 GB RAM, 70 GB disco (VM de UTM) |
| Servidor ASO | `srv-aso` | 192.168.20.101 | Servicio principal (___) | 1 vCPU, 1 GB, 10 GB |
| Cliente | `cli-linux` | 192.168.20.102 | Pruebas | 1 vCPU, 1 GB, 10 GB |

### Gateway y DNS

| Red / equipo | Gateway | DNS | Comentario |
|---|---|---|---|
| `pve-pico` en host-only 192.168.20.0/24 | Sin gateway | No aplica | Solo se usa para gestionar Proxmox desde el Mac |
| `pve-pico` en red compartida 10.0.2.0/24 | 10.0.2.1 | 10.0.2.1, 1.1.1.1 | 10.0.2.1 es el Mac, que reenvía las consultas al DNS de la red en la que esté |
| `srv-aso` y `cli-linux` (192.168.20.0/24) | 192.168.20.10 | 10.0.2.1, 1.1.1.1 | Salen a Internet a través de Proxmox, que hace NAT hacia la red compartida |

### Puertos necesarios

| Origen | Destino | Puerto | Para qué |
|---|---|---|---|
| Mac | `pve-pico` (192.168.20.10) | 8006/TCP | Web de Proxmox |
| Mac | `pve-pico` (192.168.20.10) | 22/TCP | SSH al nodo |
| `cli-linux` | `srv-aso` | ___ | Servicio ASO |
| Invitados | Internet (a través de Proxmox) | 80, 443/TCP y 53/UDP | Actualizaciones y DNS |

## Diagrama de Red

```mermaid
flowchart TD
    Mac["Mac (UTM)"] -->|"Host-only 192.168.20.0/24<br>8006 y 22"| VMBR0["vmbr0<br>192.168.20.0/24"]
    VMBR0 --- PVE["pve-pico<br>192.168.20.10 / 10.0.2.100<br>Nodo Proxmox"]
    PVE -->|"NAT (salida a Internet)"| VMBR1["vmbr1<br>Red compartida UTM 10.0.2.0/24"]
    VMBR1 --> NET["Internet"]
    VMBR0 --> SRV["srv-aso<br>192.168.20.101<br>Servicio ASO"]
    VMBR0 --> CLI["cli-linux<br>192.168.20.102<br>Cliente pruebas"]
    CLI -->|"puerto ___"| SRV
```
He hecho también un esquema como el que está hecho en el apartado de redes virtuales y recorridos:

![Esquema Red Desde Cliente](../evidencias/capturas-o-salidas/esquema_red.png)




## Comandos usados

En este apartado no he usado ningún comando, realizaré las configuraciones en el siguiente paso.

## Evidencias

- [Red host-only en UTM](../evidencias/capturas-o-salidas/red_hostonly_utm.png): adaptador host-only de la VM, con la red 192.168.20.0/24.
- [Red compartida en UTM](../evidencias/capturas-o-salidas/red_compartida_utm.png): adaptador de red compartida de la VM, con la red NAT 10.0.2.0/24.

## Incidencias

## Conclusion
Este apartado me permite seguir con los siguientes pasos, este paso es esencial para tener claro más adelante cómo voy a realizar las configuraciones de los adaptadores durante la instalación y después de la instalación. De momento he configurado dos adaptadores, uno con host only que nos sirve para gestionar y otro que hace NAT para la salida de las VMs a Internet.

[Volver al indice](../README.md)