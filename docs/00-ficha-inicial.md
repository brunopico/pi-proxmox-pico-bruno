# Ficha inicial

| Campo | Contenido |
|---|---|
| Identificador del alumno | Bruno Pico |
| URL GitHub | https://github.com/brunopico/pi-proxmox-pico-bruno.git |
| Acceso docente comprobado | SÍ (el profesor ya ha revisado el repositorio en la nota de seguimiento de la fase 4) |
| Modalidad fisica o virtualizada | VIRTUALIZADA. Proxmox va en una VM de VirtualBox en mi portátil con Debian (al principio era física y luego UTM en el Mac, los cambios los explico en la [bitácora](00-bitacora.md)) |
| Recursos conocidos | Portátil con Debian 13.7: Intel i5-10210U (4 núcleos, 8 hilos, VT-x), 7,6 GiB de RAM y NVMe de 930 GB. A la VM de Proxmox le doy 3 vCPU, 4 GB de RAM y 70 GB de disco fijo, con dos tarjetas de red (host-only y Red NAT). Proxmox VE 9.2 amd64 |
| Nombre del nodo | pve-pico |
| Servicio ASO previsto | PENDIENTE (a elegir entre Samba, NFS, SSH, LDAP, CUPS o Nginx administrado) |
| Red de laboratorio | Dos redes en VirtualBox: host-only `vboxnet0` 192.168.20.0/24 para gestionar Proxmox desde el portátil y para los invitados (nodo en 192.168.20.10, srv-aso en .101 y cli-linux en .102) y Red NAT `NatNetwork` 10.0.2.0/24 para salir a Internet (nodo en 10.0.2.100). Gateway y DNS: 10.0.2.1. Todo explicado en el [diagrama de red](02-diagrama-red.md). VLAN: no la uso |
| Riesgos y dudas iniciales | El portátil solo tiene 7,6 GiB de RAM y la VM se lleva 4 GB (dentro, el nodo ve 3,8 GiB), así que voy un poco justo: Proxmox gasta unos 1,5 GB, `srv-aso` (VM) tendrá 1 GB y `cli-linux` (LXC) 512 MB. Si abro muchas cosas a la vez en Debian seguramente vaya lento. Todo depende del portátil: si se queda sin RAM o falla el disco, se resiente el laboratorio |

[Volver al indice](../README.md)
