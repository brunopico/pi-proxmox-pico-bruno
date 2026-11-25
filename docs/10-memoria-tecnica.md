# Memoria tecnica: nodo Proxmox individual

La memoria presenta el resultado tecnico por temas. Debe explicar como esta
construido el laboratorio, por que se tomaron las decisiones y con que pruebas
se demostro su funcionamiento. Usa la bitacora como fuente, pero no copies el
diario completo: resume la configuracion final, los resultados y las
limitaciones que sigan abiertas.

## Resumen

Voy a montar un laboratorio de virtualización mío: un nodo Proxmox VE 9.2 dentro de una VM de VirtualBox en mi portátil con Debian. Dentro del nodo habrá dos invitados: `srv-aso`, una VM con el servicio de ASO, y `cli-linux`, un contenedor LXC desde el que pruebo ese servicio. El laboratorio es para mí, para practicar lo que damos en ASO sin depender del Proxmox del centro. Al final quiero tener el servicio funcionando y probado desde el cliente, los despliegues automatizados con Bash y Ansible y una copia de seguridad hecha y restaurada.

Empecé en el Mac con UTM, pero lo pasé a Debian con VirtualBox porque ahí tengo KVM. Ahora mismo (fase 4) el nodo está instalado y actualizado, tiene red, un almacenamiento para copias y los repositorios funcionando. Este resumen lo revisaré cuando termine el proyecto.

## Necesidad y objetivos

Necesidad que atiende el laboratorio: necesito un sitio donde poder instalar, romper y volver a montar servicios de ASO sin miedo a estropear nada y sin depender de los equipos ni del Proxmox del instituto. Tiene que ser un laboratorio que pueda volver a montar desde cero siguiendo mi documentación, y que tenga copias para poder volver atrás si algo sale mal.

- Objetivo comprobable 1: tener el nodo Proxmox accesible desde mi portátil por el panel (`https://192.168.20.10:8006`) y por SSH, con KVM, salida a Internet y actualizado sin errores. **Conseguido** (fases 3 y 4, [pruebas](03-instalacion-proxmox.md#fases-3-y-4-debian-virtualbox)).
- Objetivo comprobable 2: crear `srv-aso` (VM) y `cli-linux` (LXC) a partir de una plantilla, con las IPs del [diseño](02-diagrama-red.md), y comprobar desde el cliente que el servicio ASO funciona. Pendiente (fases 5, 6 y 7).
- Objetivo comprobable 3: hacer una copia de seguridad de `srv-aso` en `backup-lab`, restaurarla y comprobar que un dato del servicio sigue ahí después de restaurar. Pendiente (fase 9).

## Alcance y criterios de aceptacion

**Qué incluye:**
- Un nodo Proxmox VE 9.2 en una VM de VirtualBox (3 vCPU, 4 GB de RAM y 70 GB de disco fijo) en mi portátil con Debian.
- Dos redes: una host-only (192.168.20.0/24) para entrar al nodo y para los invitados, y una Red NAT (10.0.2.0/24) para salir a Internet.
- Dos invitados: `srv-aso` (VM) con el servicio ASO y `cli-linux` (LXC) para probarlo.
- Una plantilla Linux, dos scripts de Bash, un playbook de Ansible, copias con su restauración y pruebas de rendimiento y seguridad.

**Qué queda fuera:**
- Alta disponibilidad, clúster o varios nodos: solo hay un nodo.
- Entrar al laboratorio desde Internet: solo se entra desde mi portátil.
- El Proxmox del centro, que no forma parte del proyecto.
- Comprar hardware: uso el portátil que ya tengo.

**Servicio ASO:** todavía no lo he elegido.

**Criterios de aceptación:**

| Resultado | Prueba para darlo por bueno | Estado |
|---|---|---|
| Nodo Proxmox | Panel en `https://192.168.20.10:8006`, `pveversion` correcto y existe `/dev/kvm` | Hecho ([pve-fase3.txt](../evidencias/virtualbox/pve-fase3.txt)) |
| Red del nodo | Ruta por defecto por 10.0.2.1, el DNS resuelve y `apt update` va sin errores | Hecho ([pve-conectividad.txt](../evidencias/virtualbox/pve-conectividad.txt), [pve-apt-update.txt](../evidencias/virtualbox/pve-apt-update.txt)) |
| Almacenamiento | `local`, `local-lvm` y `backup-lab` activos en `pvesm status` | Hecho ([pve-almacenamiento.txt](../evidencias/virtualbox/pve-almacenamiento.txt)) |
| Cuentas y permisos | Una cuenta limitada con solo `Sys.Audit` puede ver el estado del nodo, pero no leer el registro (403), y después de quitarle el permiso ya no ve nada | Hecho ([pve-cuenta-lectura.txt](../evidencias/virtualbox/pve-cuenta-lectura.txt)) |
| Nodo actualizado | `apt full-upgrade` sin errores y, después de reiniciar, todo sigue funcionando y no quedan paquetes pendientes | Hecho ([pve-upgrade.txt](../evidencias/virtualbox/pve-upgrade.txt), [pve-tras-upgrade.txt](../evidencias/virtualbox/pve-tras-upgrade.txt)) |
| Plantilla Linux | Se puede clonar y el clon arranca bien | Fase 5 |
| Invitados | `srv-aso` y `cli-linux` arrancan con su IP y hostname y salen a Internet por el NAT del nodo | Fase 6 |
| Servicio ASO | Desde `cli-linux` el servicio de `srv-aso` funciona | Fase 7 |
| Automatización | Los dos scripts y el playbook funcionan y se pueden repetir | Fase 8 |
| Copias | La copia se restaura y el dato que guardé sigue ahí | Fase 9 |
| Rendimiento y seguridad | Medidas tomadas y explicadas | Fases 10 y 11 |

## Modalidad e inventario

**Modalidad:** virtualizada, con VirtualBox 7.2.22 en mi portátil con Debian 13.7 (i5-10210U de 4 núcleos y 8 hilos, 7,6 GiB de RAM y NVMe de 930 GB). Al principio iba a ser física y luego lo monté en el Mac con UTM, pero en UTM no tenía KVM y los invitados solo podían ser contenedores. En VirtualBox sí tengo VT-x anidado, así que `srv-aso` puede ser una VM. Lo de UTM está guardado en la etiqueta `utm-macos-fase4`.

**Inventario:** lo hice con el ordenador de clase (Pentium G4560, 8 GB de RAM y un disco mecánico), como pidió el profesor, y salió "apto con condiciones", sobre todo por el disco mecánico y los 2 núcleos. Los datos del portátil que uso ahora están en las [evidencias del anfitrión](../evidencias/virtualbox/host-lscpu.txt).

Detalle: [Inventario](01-inventario-hardware.md).

## Diseno de red

El nodo tiene dos bridges: `vmbr0` sobre la host-only (192.168.20.10/24, sin gateway), para el panel, el SSH y los invitados, y `vmbr1` sobre la Red NAT de VirtualBox (10.0.2.100/24, gateway 10.0.2.1), para salir a Internet. Los invitados (`srv-aso` en .101 y `cli-linux` en .102) salen a Internet a través del nodo, que hace NAT. Así solo se puede entrar al laboratorio desde mi portátil.

Uso otros nombres e IPs que la referencia del profesor, pero funciona igual. Lo comprobé con `ip -br addr`, `ip route` y un ping al gateway. Lo que falta es probar el NAT desde los invitados (fase 6) y poner el puerto del servicio ASO cuando lo elija.

Detalle: [Diagrama y direccionamiento](02-diagrama-red.md).

## Instalacion y configuracion de Proxmox

**Configuración final:** Proxmox VE 9.2 amd64 (ISO comprobada con SHA256), con ext4 sobre LVM, hostname `pve-pico`, repositorio No-Subscription y tres almacenamientos (`local`, `local-lvm` y `backup-lab`). En VirtualBox hice las cinco mejoras de la VM: VT-x anidado, paravirtualización KVM, modo promiscuo, disco fijo y virtio-scsi.

**Por qué:** ext4 y no ZFS porque ZFS gasta más RAM y voy justo. El No-Subscription porque el enterprise es de pago. `backup-lab` porque en `local-lvm` no se pueden guardar copias.

**Cómo lo comprobé:** existe `/dev/kvm`, el disco y el LVM salen bien en `lsblk`, `pvs`, `vgs` y `lvs` (el thin pool está vacío porque aún no hay invitados), el panel abre, `apt update` va sin errores y `pvesm status` muestra los tres almacenamientos activos. Después actualicé el nodo: 225 paquetes, `pve-manager` de 9.2.2 a 9.2.32 y kernel nuevo 7.0.14-30. Reinicié y lo volví a comprobar todo: red, almacenamiento, KVM y 0 paquetes pendientes.

**Accesos:** para administrar uso solo `root@pam`. Para ver cómo funcionan los permisos creé una cuenta `alumno-lectura@pve` con un rol que solo tiene `Sys.Audit` en el nodo. Con ella podía ver el estado del nodo, pero leer el registro daba 403, y cuando le quité el permiso ya no podía ver nada. La cuenta la dejo sin permisos hasta que el profesor diga que se puede borrar.

**Reparto de memoria:** aunque la VM tiene 4 GB, el nodo ve 3,8 GiB. 3,8 − 1,5 del nodo − 1 de `srv-aso` − 0,5 de `cli-linux` = 0,8 GiB de margen. Los 4 GB de swap no los cuento como RAM.

**Previsto y obtenido:**

| Qué | Previsto | Obtenido |
|---|---|---|
| Modalidad | Física, luego UTM en el Mac | VirtualBox en Debian, porque en UTM no había KVM |
| RAM de la VM | 6 GB | 4 GB (3,8 GiB visibles), porque el portátil tiene 7,6 GiB |
| `srv-aso` | LXC | VM, porque ahora hay KVM |
| Red NAT | NAT normal, gateway 10.0.2.2 | Red NAT `NatNetwork`, gateway 10.0.2.1 |
| Mejoras de la VM | 3 de 5 en UTM | Las 5 en VirtualBox |

**Limitaciones:** voy justo de RAM, y las copias de `backup-lab` están en el mismo disco que el laboratorio.

Detalle: [Procedimiento](03-instalacion-proxmox.md).

## Plantilla Linux y despliegue

PENDIENTE (fases 5 y 6): [Plantilla](04-plantilla-linux.md) y [VMs/LXC](05-vms-lxc.md)

## Servicio ASO

PENDIENTE (fase 7): [Instalacion y pruebas](06-servicio-aso.md)

## Automatizacion con Bash y Ansible

PENDIENTE (fase 8): [Automatizacion](07-automatizacion.md)

## Copias y restauracion

PENDIENTE (fase 9): [Procedimiento y validacion](08-copias-restauracion.md)

## Rendimiento y seguridad

PENDIENTE (fases 10 y 11): [Resultados](09-rendimiento-seguridad.md)

## Incidencias y decisiones

Las decisiones más importantes hasta ahora:
- Pasar de físico a virtualizado, y del Mac (UTM) a Debian (VirtualBox), para tener KVM y poder usar VMs.
- Usar la Red NAT de VirtualBox en vez del NAT normal, porque el NAT normal no respondía detrás del bridge de Proxmox.
- Bajar la RAM de la VM a 4 GB para no dejar al portátil sin memoria.
- Usar ext4 y no ZFS, porque ZFS gasta más RAM.
- Usar `backup-lab`, que ya existía, como destino de copias, limitado a mi nodo, en vez de crear otro.

Las incidencias que tuve (Secure Boot, VT-x anidado en gris, NAT, regla duplicada, disco dinámico, la asignación de permisos que no se había creado...) están explicadas en el [03](03-instalacion-proxmox.md#incidencias-2) y día a día en la [bitácora](00-bitacora.md).

Siguen abiertas dos cosas: el portátil va justo de RAM, y las copias de `backup-lab` están en el mismo disco que el laboratorio. En la fase 9 sacaré una copia fuera.

## Presupuesto y sostenibilidad

No he comprado nada: uso mi portátil, y Proxmox, VirtualBox y Debian son gratis. Uso el repositorio No-Subscription en vez de la suscripción de pago, porque para un laboratorio me sobra. Lo único que cuesta es la luz del portátil, que gasta poco (el i5-10210U es un procesador de bajo consumo, de 15 W según Intel), y además solo enciendo el laboratorio cuando lo uso. Virtualizar me permite tener tres máquinas en un solo equipo en vez de tres ordenadores. Lo completaré al final del proyecto.

## Conclusiones

PENDIENTE: lo escribiré al terminar el proyecto. De momento, hasta la fase 4, el nodo está listo y actualizado para empezar con la plantilla y los invitados.

## Fuentes y anexos

[Evidencias](../evidencias/README.md) y [volver al indice](../README.md)
