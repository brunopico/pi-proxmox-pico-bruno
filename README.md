# Proyecto Intermodular: nodo Proxmox individual

Alumno/a: pico_bruno

URL GitHub: https://github.com/brunopico/pi-proxmox-pico-bruno.git

## Objetivo

Disponer de un laboratorio de virtualización propio, reproducible y documentado, donde practicar la administración de servicios de ASO sin depender del Proxmox del centro. El laboratorio será un único nodo Proxmox VE con dos invitados: un servidor de servicio (`srv-aso`) y un cliente Linux (`cli-linux`) desde el que se comprueba el servicio. El trabajo incluye automatización con Bash y Ansible, y copias de seguridad con su restauración probada. Empecé montando el nodo en mi Mac con UTM, pero ahora lo tengo en mi portátil con Debian y VirtualBox. Aquí sí tengo virtualización anidada (KVM), así que `srv-aso` va a ser una VM y `cli-linux` un contenedor LXC.

## Modalidad

Virtualizada (VirtualBox). Proxmox VE 9.2 amd64 va en una VM de VirtualBox en mi portátil con Debian 13 (Intel i5-10210U). Hasta la fase 4 lo tenía en el Mac con UTM y Proxmox ARM64. Eso lo he dejado guardado en la etiqueta `utm-macos-fase4` y el cambio lo explico en la [bitácora](docs/00-bitacora.md).

## Indice de documentacion

- [Ficha inicial](docs/00-ficha-inicial.md)
- [Bitacora de sesiones](docs/00-bitacora.md)
- [Inventario](docs/01-inventario-hardware.md)
- [Diagrama de red](docs/02-diagrama-red.md)
- [Instalacion de Proxmox](docs/03-instalacion-proxmox.md)
- [Plantilla Linux](docs/04-plantilla-linux.md)
- [VMs y LXC](docs/05-vms-lxc.md)
- [Servicio ASO](docs/06-servicio-aso.md)
- [Automatizacion Bash y Ansible](docs/07-automatizacion.md)
- [Copias y restauracion](docs/08-copias-restauracion.md)
- [Rendimiento y seguridad](docs/09-rendimiento-seguridad.md)
- [Memoria tecnica](docs/10-memoria-tecnica.md)
- [Evidencias](evidencias/README.md)

## Estado

- [x] GitHub creado y primer commit publicado
- [ ] URL entregada y acceso docente comprobado
- [x] Modalidad indicada
- [x] Inventario
- [x] Diagrama de red
- [x] Instalacion Proxmox (repetida en VirtualBox)
- [ ] Plantilla Linux
- [ ] VMs/LXC
- [ ] Servicio ASO
- [ ] Scripts Bash
- [ ] Ansible
- [ ] Copias y restauracion
- [ ] Rendimiento y seguridad
- [ ] Memoria final
