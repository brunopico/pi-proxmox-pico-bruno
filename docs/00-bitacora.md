# Bitacora de sesiones

La bitacora es el registro cronologico del trabajo. Anade una entrada al final
de cada sesion y conserva las anteriores. Escribe datos comprobables y enlaza
las evidencias utiles; evita frases generales como "he trabajado en Proxmox".

## Sesion 01 - 22-09-2026

- Objetivo: iniciar GitHub y la documentacion Markdown.
- Acciones realizadas: He seguido la guía de la primera sesión de GitHub y he ejecutado el script, que me ha creado el repositorio con las carpetas que pide el enunciado (README.md, .gitignore, docs/, scripts/, ansible/ y evidencias/).
- Decisiones y motivos: He dejado los nombres y las carpetas tal cual vienen en el enunciado, porque son obligatorios y así el profesor encuentra todo donde lo espera.
- Pruebas y resultados: Quería tener el repositorio con la estructura del enunciado. He ejecutado el script de la guía y la estructura se ha creado bien.
- Documentos o evidencias: [Ficha inicial](00-ficha-inicial.md).
- Incidencias y solucion o estado pendiente: Ninguna.
- Siguiente paso: Rellenar la ficha inicial y empezar la Fase 1 (inventario del equipo).

## Sesion 02 - 07-10-2026

- Objetivo: empezar la Fase 1, el inventario del equipo y ver si se puede instalar Proxmox (docs/01-inventario-hardware.md).
- Acciones realizadas: He arrancado el equipo con Debian Live desde un USB y he mirado sus características: Pentium G4560 (2 núcleos y 4 hilos), 8 GB DDR4 a 2400 MHz, tarjeta de red Realtek, BIOS ASUS 0810 y VT-x. Con eso he hecho un primer borrador de las tres preguntas y del reparto de recursos.
- Decisiones y motivos: Elijo la modalidad física para trabajar con el hardware real. Pienso instalar con ext4 sobre LVM y no con ZFS, porque ZFS gasta más RAM y solo tengo un disco y 8 GB. A cada invitado le doy 1 vCPU y el servidor lo haré con LXC porque consume menos que una VM.
- Pruebas y resultados: Quería comprobar que el equipo puede virtualizar. Lo he mirado en las especificaciones y en la BIOS y tiene VT-x, pero todavía no había guardado la salida de ningún comando.
- Documentos o evidencias: [Inventario de hardware](01-inventario-hardware.md).
- Incidencias y solucion o estado pendiente: En el borrador puse datos sin comprobar con comandos (el tipo de disco y cuántos módulos de RAM hay). Lo corregí en la sesión 03.
- Siguiente paso: Guardar las salidas de los comandos (lscpu, free, lsblk, df, ip, lspci, dmidecode y smartctl) y terminar el inventario.

## Sesion 03 - 13-10-2026

- Objetivo: terminar la Fase 1 con datos reales y dejar el inventario listo para entregar.
- Acciones realizadas: Desde Debian Live he guardado 11 salidas de comandos en evidencias/capturas-o-salidas/ (lscpu, free, lsblk, df, ip, lspci, systemd-detect-virt, dmidecode de memoria y de BIOS, smartctl y la velocidad de la red sacada de dmesg). Antes de subirlas he quitado los números de serie y el UUID del disco. Con esos datos he rehecho el inventario, el reparto de recursos y las fuentes, y he rellenado la ficha inicial y el README.
- Decisiones y motivos: El disco resultó ser un HDD y no un SSD, así que lo he puesto como componente más crítico en la pregunta 3: es lo que más va a ralentizar el laboratorio y, si falla, lo pierdo todo. La primera mejora sería cambiarlo por un SSD y la segunda la CPU. El dictamen es apto con condiciones, porque cumple los mínimos de Proxmox pero no lo recomendado en disco ni en red.
- Pruebas y resultados: `lscpu` muestra VT-x. `dmidecode` muestra 2 módulos de 4 GB en dual channel y `free -h` 7,6 GiB. `lsblk` da ROTA=1 y `smartctl` confirma que es un Toshiba de 7200 rpm con SMART correcto. La red va a 1 Gbps. `systemd-detect-virt` da none, o sea, es un equipo físico.
- Documentos o evidencias: [Inventario de hardware](01-inventario-hardware.md); [Ficha inicial](00-ficha-inicial.md); salidas en [evidencias](../evidencias/README.md).
- Incidencias y solucion o estado pendiente: En el borrador había puesto SSD y un solo módulo de RAM, y era un HDD con dos módulos; lo he corregido. El ethtool.txt salió vacío y saqué la velocidad de dmesg. El disco tiene 317 GiB de datos que se borrarán al instalar Proxmox y tengo que decidir qué hago con ellos. Las ranuras de RAM las saqué de la web de ASUS y de una tienda, me falta mirarlas en el manual. En la ficha me quedan el acceso del profesor, el servicio ASO y la red.
- Siguiente paso: Enseñar el inventario al profesor, decidir qué hago con los datos del disco, descargar la ISO de Proxmox 9.2 y empezar la instalación.

## Sesion 04 - 20-10-2026

- Objetivo: empezar la Fase 2, el diseño de la red (docs/02-diagrama-red.md).
- Acciones realizadas: He cambiado a modalidad virtualizada y voy a montar Proxmox en una VM de UTM en mi Mac. En UTM le he puesto dos redes a la VM: una host-only (192.168.20.0/24) para entrar a la web de Proxmox y una red compartida (NAT, 10.0.2.0/24) para tener salida a Internet. He hecho la tabla de direcciones, la de puertos, el diagrama y las medidas de aislamiento.
- Decisiones y motivos: Cambio a UTM porque no hay equipos suficientes en clase y además es muy lioso y harían falta varios switches. Uso UTM y no VirtualBox porque mi portatil es arm. Las máquinas invitadas van en la red compartida para que tengan Internet sin montar NAT en Proxmox. Las IPs son fijas: 192.168.20.10 y 10.0.2.100 para el nodo, 10.0.2.101 para srv-aso y 10.0.2.102 para cli-linux.
- Pruebas y resultados: Todavía no hay pruebas, porque esta fase es de diseño. He comprobado en UTM que las dos redes están configuradas (capturas).
- Documentos o evidencias: [Diagrama de red](02-diagrama-red.md); capturas red_hostonly_utm.png y red_compartida_utm.png.
- Incidencias y solucion o estado pendiente: El inventario del equipo físico ya no vale para la nueva modalidad, tengo que preguntar al profesor si lo rehago con los datos del Mac. Me falta elegir el servicio ASO y revisar la RAM. Me queda comprobar en el paso siguiente si UTM permite virtualización anidada de máquinas, si no me es posible usaré un Windows Server 2025 que tengo en la nube..
- Siguiente paso: Elegir el servicio ASO, ajustar la RAM de la VM e instalar Proxmox.

## Sesion 05 - 27-10-2026

- Objetivo: hacer la Fase 3, instalar Proxmox en la VM de UTM (docs/03-instalacion-proxmox.md).
- Acciones realizadas: He creado la VM en UTM con motor QEMU, 2 vCPU, 6 GB de RAM, 70 GB de disco y dos tarjetas (host-only y red compartida). He instalado Proxmox VE 9.2 para ARM64 con ext4 (LVM), la gestión en la tarjeta host-only con la IP 192.168.20.3 y he entrado al panel desde el navegador del Mac. He guardado las salidas de hostnamectl, ip -br addr, pveversion, df -h y la comprobación de KVM, y las capturas de UTM, del instalador, del panel, de la red y del almacenamiento.
- Decisiones y motivos: Probé el motor Apple Virtualization de UTM porque con un M4 permite virtualización anidada, pero no tiene red host-only y el diseño necesita dos tarjetas, así que me quedé con QEMU. Uso ext4 y no ZFS porque ZFS gasta más RAM y la VM solo tiene 6 GB. Como no hay virtualización anidada, srv-aso y cli-linux serán contenedores LXC.
- Pruebas y resultados: `pveversion` da pve-manager 9.2.9 y el panel abre en https://192.168.20.3:8006. `ls -l /dev/kvm` no existe y dmesg dice `HYP mode not available`, y una VM de prueba en Proxmox no arranca por falta de KVM.
- Documentos o evidencias: [Instalación de Proxmox](03-instalacion-proxmox.md); salidas y capturas en evidencias/capturas-o-salidas/.
- Incidencias y solucion o estado pendiente: El instalador obliga a poner gateway en la red de gestión y la host-only no tiene, puse 192.168.20.1 para seguir. Puse el hostname brunop.local en vez de pve-pico, lo cambiaré más adelante. La IP del nodo es la .3 y en el diseño puse la .10, tengo que dejarlo igual en los dos sitios.
- Siguiente paso: Configurar la red del diseño (vmbr1, gateway real y NAT con iptables), elegir el servicio ASO y preparar la plantilla Linux.

## Sesion 06 - 10-11-2026

- Objetivo: hacer la Fase 4, dejar el nodo preparado (almacenamiento, red, accesos y recursos) y aplicar las mejoras de la VM que pidió el profesor (docs/03-instalacion-proxmox.md).
- Acciones realizadas: En UTM he dejado el disco de tamaño fijo con `qemu-img` y he probado la interfaz SCSI. En Proxmox he creado `backup-lab` para las copias, he configurado la red del diseño (`vmbr0` con la .10 y `vmbr1` con 10.0.2.100 y salida a Internet), he puesto el NAT para los invitados y he cambiado los repositorios enterprise por el No-Subscription.
- Decisiones y motivos: Sigo con UTM porque en un Mac ARM VirtualBox tampoco tiene VT-x. De las cinco mejoras hago tres (paravirtualización, modo promiscuo y disco fijo). Cambio la IP del nodo a la .10 para que cuadre con el diagrama. Uso el repositorio No-Subscription porque es gratis.
- Pruebas y resultados: El panel abre en https://192.168.20.10:8006. La ruta por defecto va por 10.0.2.1, el ping al gateway funciona y `apt update` va sin errores. `nic1` está en modo promiscuo, la VM tiene 5,8 GiB de RAM y el disco ocupa 70 GiB.
- Documentos o evidencias: [Configuración base del nodo](03-instalacion-proxmox.md#fase-4-macos-utm-configuración-base-del-nodo); salidas y capturas en evidencias/capturas-o-salidas/.
- Incidencias y solucion o estado pendiente: Con el disco en SCSI la VM no arrancaba porque no veía el disco. Volví a VirtIO y el profesor me dijo que vale. Los repositorios enterprise daban error 401 y los cambié por el No-Subscription. El ping a Internet no responde porque el centro lo bloquea por sus políticas, pero `apt update` descarga bien.
- Siguiente paso: Actualizar los paquetes, descargar una plantilla LXC ARM64 y preparar la plantilla Linux.

## Sesion 07 - 17-11-2026

- Objetivo: pasar el laboratorio a mi portátil con Debian y VirtualBox, que es lo que quería el profesor, y volver a hacer las fases 3 y 4 (docs/03-instalacion-proxmox.md).
- Acciones realizadas: Primero guardé cómo estaba todo en el Mac con la etiqueta `utm-macos-fase4` y me hice la rama `virtualbox-debian` para no machacar nada. Instalé VirtualBox, creé la red host-only `vboxnet0` y la VM `pve-pico` (3 vCPU, 4 GB, 70 GB, EFI, virtio-scsi, virtio-net y VT-x anidado) e instalé Proxmox VE 9.2 amd64. Después configuré `vmbr0` y `vmbr1` con NAT, puse el repositorio No-Subscription, creé `backup-lab` e hice las cinco mejoras de la VM.
- Decisiones y motivos: Cambio de equipo porque en VirtualBox sí tengo VT-x anidado y KVM, y así `srv-aso` puede ser una VM. Le bajo la RAM a la VM a 4 GB porque el portátil solo tiene 7,6 GiB. Uso la Red NAT y no el NAT normal porque el normal no contestaba detrás del bridge.
- Pruebas y resultados: El SHA256 de la ISO coincide. `/dev/kvm` existe y `pveversion` da 9.2.2. El panel abre en https://192.168.20.10:8006, el ping a 10.0.2.1 va, `deb.debian.org` se resuelve y `apt update` va sin errores. El disco sale como `fixed default`.
- Documentos o evidencias: [Fases 3 y 4 en VirtualBox](03-instalacion-proxmox.md#fases-3-y-4-debian-virtualbox); salidas y capturas en evidencias/virtualbox/.
- Incidencias y solucion o estado pendiente: Bastantes: Secure Boot (lo arreglé firmando con MOK), VT-x anidado en gris (`VBoxManage`), me equivoqué con el hostname en el instalador, puse los adaptadores al revés, el NAT normal no iba (cambié a Red NAT), la regla MASQUERADE salía repetida y el disco se creó dinámico (`clonemedium`). Lo explico todo en el documento.
- Siguiente paso: Actualizar los 213 paquetes, bajar las plantillas y crear `srv-aso` y `cli-linux`.

Anadir una entrada con esta estructura en cada sesion. El historial de
GitHub registra los commits; no hace falta incluir el hash del propio
commit dentro de su entrada de bitacora.

[Volver al indice](../README.md)
