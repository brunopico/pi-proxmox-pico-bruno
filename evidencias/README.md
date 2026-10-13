# Proyecto Intermodular: nodo Proxmox individual

Alumno/a: pico_bruno

URL GitHub: https://github.com/brunopico/pi-proxmox-pico-bruno.git

## Objetivo

Disponer de un laboratorio de virtualización propio, reproducible y documentado, donde practicar la administración de servicios de ASO sin depender del Proxmox del centro. El laboratorio será un único nodo Proxmox VE con dos invitados: un servidor de servicio (`srv-aso`) y un cliente Linux (`cli-linux`) desde el que se comprueba el servicio. El trabajo incluye automatización con Bash y Ansible, y copias de seguridad con su restauración probada. Como el equipo tiene recursos limitados (2 núcleos, 8 GB de RAM y un disco duro mecánico), el reparto de recursos se calcula y se justifica en el inventario.

## Modalidad

Física. Proxmox VE se instalará directamente en un equipo real (Intel Pentium G4560, 8 GB de RAM, HDD de 500 GB y placa ASUS H110I-PLUS) y no en una VM de VirtualBox. Así se trabaja con el hardware real, se evita la virtualización anidada y se pueden medir las limitaciones reales del equipo. El inventario confirma que el equipo es apto con condiciones (VT-x disponible y requisitos mínimos de Proxmox cumplidos).

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
- [ ] Diagrama de red
- [ ] Instalacion Proxmox
- [ ] Plantilla Linux
- [ ] VMs/LXC
- [ ] Servicio ASO
- [ ] Scripts Bash
- [ ] Ansible
- [ ] Copias y restauracion
- [ ] Rendimiento y seguridad
- [ ] Memoria final
