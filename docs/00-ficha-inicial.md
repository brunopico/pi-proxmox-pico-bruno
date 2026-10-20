# Ficha inicial

| Campo | Contenido |
|---|---|
| Identificador del alumno | Bruno Pico |
| URL GitHub | https://github.com/brunopico/pi-proxmox-pico-bruno.git |
| Acceso docente comprobado | PENDIENTE |
| Modalidad fisica o virtualizada | VIRTUALIZADA. Proxmox va en una VM de UTM en mi Mac (al principio era física, el cambio está explicado en la [bitácora](00-bitacora.md)) |
| Recursos conocidos | Mi Mac: MacBook Air con chip M4, 16 GB de RAM y macOS 27.0.1. A la VM de Proxmox le doy 2 vCPU, 6 GB de RAM y 70 GB de disco, con dos tarjetas de red (host-only y red compartida). Proxmox VE 9.2 para ARM64 |
| Nombre del nodo | pve-pico |
| Servicio ASO previsto | PENDIENTE (a elegir entre Samba, NFS, SSH, LDAP, CUPS o Nginx administrado) |
| Red de laboratorio | Dos redes en UTM: host-only 192.168.20.0/24 para gestionar Proxmox desde el Mac (nodo en 192.168.20.10) y red compartida con NAT 10.0.2.0/24 para salir a Internet (nodo en 10.0.2.100, srv-aso en 10.0.2.101 y cli-linux en 10.0.2.102). Gateway y DNS: 10.0.2.1. Todo explicado en el [diagrama de red](02-diagrama-red.md). VLAN: no la uso |
| Riesgos y dudas iniciales | Con 6 GB en la VM no se si iré algo justo de RAM: Proxmox necesita unos 2 GB y los dos invitados 1 GB cada uno. Proxmox para ARM es nuevo y en un Mac solo tiene soporte "best-effort". Las ISOs y plantillas tienen que ser ARM64. Tengo que comprobar si UTM me deja virtualización anidada para las VMs de dentro; si no, los invitados tendrán que ser LXC. Todo depende de mi Mac: si se queda sin RAM o sin disco, se resiente el laboratorio |

[Volver al indice](../README.md)
