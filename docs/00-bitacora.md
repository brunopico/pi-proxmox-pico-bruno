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

Anadir una entrada con esta estructura en cada sesion. El historial de
GitHub registra los commits; no hace falta incluir el hash del propio
commit dentro de su entrada de bitacora.

[Volver al indice](../README.md)
