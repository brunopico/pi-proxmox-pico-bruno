# Bitacora de sesiones

La bitacora es el registro cronologico del trabajo. Anade una entrada al final
de cada sesion y conserva las anteriores. Escribe datos comprobables y enlaza
las evidencias utiles; evita frases generales como "he trabajado en Proxmox".

## Sesion 01 - 22-09-2026

- Objetivo: iniciar GitHub y la documentacion Markdown.
- Acciones realizadas: Seguida guia de primera sesion github, script ejecutado y formacion de la estructura. Se crea el repositorio individual con la estructura obligatoria del enunciado (README.md, .gitignore, docs/, scripts/, ansible/ y evidencias/).
- Decisiones y motivos: Se respetan los nombres y la estructura del enunciado sin modificarlos, porque son obligatorios y así el profesor encuentra cada documento donde lo espera. No se valoraron alternativas porque la estructura viene fijada.
- Pruebas y resultados: Resultado esperado: repositorio con la estructura del enunciado. Método: script de la guía de la primera sesión. Resultado obtenido: estructura creada.
- Documentos o evidencias: [Ficha inicial](00-ficha-inicial.md).
- Incidencias y solucion o estado pendiente: Ninguna anotada en esta sesión.
- Siguiente paso: Completar la ficha inicial y empezar la Fase 1 (inventario del equipo y viabilidad de Proxmox).

## Sesion 02 - 07-10-2026

- Objetivo: iniciar la Fase 1: inventario del equipo y viabilidad de Proxmox (docs/01-inventario-hardware.md).
- Acciones realizadas: Arranque del equipo con Debian Live desde USB y lectura de sus características: Intel Pentium G4560 (2 núcleos, 4 hilos), 8 GB DDR4 a 2400 MHz, red Realtek RTL8111/8168/8411, BIOS ASUS versión 0810 y VT-x soportado. Redacción de un primer borrador de las tres preguntas (instalación, limitaciones y componentes críticos) y de la tabla de reparto de recursos.
- Decisiones y motivos: Se trabaja con el equipo físico (modalidad física) y no con VirtualBox, para investigar el hardware real. Se prevé instalar con ext4 sobre LVM y no con ZFS, porque ZFS consume más RAM para su caché y solo hay un disco y poca memoria. Se asigna 1 vCPU a cada invitado y se prefiere LXC para el servidor, que consume menos que una VM completa.
- Pruebas y resultados: Resultado esperado: confirmar que la CPU y la placa permiten virtualizar. Método: lectura de las especificaciones del equipo y de la BIOS. Resultado obtenido: VT-x soportado; faltaba guardar la salida de comandos como evidencia.
- Documentos o evidencias: [Inventario de hardware](01-inventario-hardware.md).
- Incidencias y solucion o estado pendiente: El primer borrador incluía datos sin evidencia de comandos (tipo de disco y número de módulos de RAM). Se comprobaron y corrigieron el 13-10-2026 (ver sesión 03).
- Siguiente paso: Guardar las salidas de los comandos (lscpu, free, lsblk, df, ip, lspci, dmidecode y smartctl) y completar el inventario.

## Sesion 03 - 13-10-2026

- Objetivo: completar la Fase 1 con datos medidos y dejar el inventario listo para entregar.
- Acciones realizadas: Con Debian Live (Debian 13, kernel 6.12.94) se guardaron 11 salidas en evidencias/capturas-o-salidas/: lscpu, free, lsblk, df, ip -br addr, lspci, systemd-detect-virt, dmidecode (memoria y BIOS), smartctl y el mensaje del driver de red con la velocidad del enlace (dmesg). Se eliminaron números de serie y el UUID del disco antes de publicarlas. Con esos datos se reescribió el inventario, se recalculó el reparto de recursos y se completaron las fuentes (Proxmox, Intel ARK, Toshiba y ASUS). También se completaron la ficha inicial (recursos, nodo y riesgos) y el README (objetivo, modalidad y estado).
- Decisiones y motivos: (1) El disco es un HDD, no un SSD, así que pasa a prioridad 1 en la pregunta 3 (es lo que más frena el laboratorio y lo que más daño hace si falla) y la mejora que se propone primero es sustituirlo por un SSD SATA; la CPU queda como segunda mejora. (2) Dictamen de viabilidad: apto con condiciones, porque se cumplen los mínimos de Proxmox pero no lo recomendado en almacenamiento ni en red.
- Pruebas y resultados: VT-x: `lscpu` muestra Virtualization VT-x y el flag vmx. RAM: `dmidecode` muestra 2 módulos Crucial de 4 GB DDR4 a 2400 MT/s (ChannelA y ChannelB, dual channel) y `free -h` 7,6 GiB visibles. Disco: `lsblk` da ROTA=1 y `smartctl` confirma TOSHIBA DT01ACA050 de 7200 rpm, SMART PASSED, 0 sectores reasignados y 5444 horas de uso. Red: driver r8169 y enlace a 1 Gbps en dúplex completo. Plataforma: placa ASUS H110I-PLUS, BIOS AMI 0810 con UEFI. Equipo físico confirmado con `systemd-detect-virt` (none).
- Documentos o evidencias: [Inventario de hardware](01-inventario-hardware.md); [Ficha inicial](00-ficha-inicial.md); salidas en [evidencias](../evidencias/README.md).
- Incidencias y solucion o estado pendiente: (1) El borrador anotaba un SSD y un solo módulo de RAM; las salidas mostraron un HDD y dos módulos, y se corrigió el inventario. (2) El archivo ethtool.txt quedó vacío; la velocidad del enlace se obtuvo del mensaje del driver en dmesg (velocidad.txt). (3) El disco contiene 317 GiB de datos y Proxmox los borrará al instalar; pendiente de decidir si se copian o se confirma que se puede formatear. (4) Las ranuras DIMM (2, ambas ocupadas) y las CPU compatibles se obtuvieron de la web de soporte de ASUS y de la ficha de un distribuidor; queda contrastar las ranuras con el manual E11036. (5) En la ficha inicial siguen pendientes el acceso docente, el servicio ASO y la red del laboratorio (gateway, DNS y direcciones fijas).
- Siguiente paso: Revisar el inventario con el profesor, resolver qué se hace con los datos del disco, descargar la ISO de Proxmox 9.2 y comprobar su hash, e iniciar la instalación (docs/03-instalacion-proxmox.md).

Anadir una entrada con esta estructura en cada sesion. El historial de
GitHub registra los commits; no hace falta incluir el hash del propio
commit dentro de su entrada de bitacora.

[Volver al indice](../README.md)
