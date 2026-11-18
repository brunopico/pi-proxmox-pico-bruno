# Inventario investigado y viabilidad

Bruno Picó 2X ASIR

## Contexto de la observación

- Fecha y sistema utilizado: 13/10/2026. Arranqué el equipo con un Debian Live (Debian 13, kernel 6.12.94) desde un USB con Ventoy. `systemd-detect-virt` da `none`, así que estoy mirando el equipo físico y no una VM (`systemd.txt`). La primera vez que miré las especificaciones fue el 07/10/2026.
- Modalidad, equipo anfitrión y VM VirtualBox si procede: Física, el equipo real. VirtualBox no procede.
- Versión de Proxmox prevista: la 9.2 (lo confirmaré cuando descargue la ISO).

---

## Inventario inicial

| Área | Dato observado y evidencia | Especificación y fuente | Límite o diferencia | Justificación |
|---|---|---|---|---|
| CPU | Intel Pentium G4560 a 3,50 GHz, de 64 bits, con 2 núcleos y 2 hilos por núcleo (4 CPU lógicas). En `lscpu` sale `Virtualization: VT-x` y el flag `vmx` (`lscpu.txt`) | Proxmox pide una CPU de 64 bits con Intel VT o AMD-V (web de requisitos). En Intel ARK el G4560 tiene 2 núcleos, 4 hilos, VT-x y VT-d, socket FCLGA1151 y DDR4-2133/2400 en 2 canales | Cumple. Si sale el flag `vmx` es que VT-x está activado en la BIOS (si estuviera desactivado normalmente no saldría). Lo confirmaré cuando arranque KVM. Ojo, que 4 hilos no son 4 núcleos: físicos solo hay 2 | Proxmox no pone un mínimo de núcleos, pero con solo 2 núcleos voy a ir limitado, porque el host y los invitados se reparten la CPU. |
| RAM | 8 GB DDR4 en 2 módulos CRUCIAL CT4G4DFS824A de 4 GB a 2400 MT/s, uno en `ChannelA-DIMM1` y otro en `ChannelB-DIMM1` (dual channel, sin ECC). El sistema ve 7,6 GiB (`free -h`), el resto se lo quedan el firmware y seguramente la gráfica integrada. Swap: 0 B en el Live (`memory.txt`, `free.txt`) | Proxmox recomienda mínimo 2 GB para el sistema más lo que gasten los invitados. Para evaluación vale con 1 GB. Según Intel ARK la CPU admite DDR4-2133/2400, 2 canales y hasta 64 GB | Cumple el mínimo y los 2 GB recomendados. Quitando esos 2 GB me quedan unos 5,6 GiB para los invitados. Al tener dos módulos uso los dos canales. La placa es mini-ITX con 2 ranuras DDR4 (hasta 2400 MHz y 32 GB, según la ficha de una tienda) y las dos están ocupadas. El SMBIOS dice 4 posiciones, pero ranuras de verdad solo hay 2. Para ampliar tendría que cambiar los módulos, por ejemplo por 2 × 8 GB | La RAM no es lo que más me va a frenar, pero sí limita cuánta memoria les puedo dar a los invitados. Los 2,4 GiB "usados" que salen en el Live no son de Proxmox, es el propio Live que va cargado en la RAM. |
| Almacenamiento | Disco duro mecánico TOSHIBA DT01ACA050 de 500 GB (465,8 GiB), 3,5 pulgadas, 7200 rpm, SATA a 6 Gb/s, sectores de 512 B lógicos y 4096 B físicos, `ROTA=1`. **No es un SSD.** SMART: PASSED, 0 sectores reasignados, 0 pendientes, 5444 horas encendido (unos 227 días) y 35 ºC. Tiene una sola partición (`sda1`, 458 GiB) con 317 GiB usados (73 %), o sea, tiene datos (`lsblk.txt`, `smartctl.txt`, `df.txt`) | Proxmox recomienda almacenamiento rápido y redundante, mejor SSD. Para evaluación vale con un disco duro. Según Toshiba (serie DT01ACA): 500 GB, 7200 rpm, SATA 6 Gbit/s, 3,5 pulgadas | Cumple el mínimo y de espacio va sobrado. Lo recomendado no lo cumple: no es SSD y no hay redundancia | Espacio hay de sobra y ahora mismo el disco está bien, pero un disco mecánico es mucho más lento que un SSD (arrancar, clonar y hacer copias). Además es el único disco, así que si falla pierdo Proxmox, las VMs, los datos y las copias. Al instalar Proxmox se borra entero, con los 317 GiB que tiene ahora. |
| Red | Tarjeta Realtek RTL8111/8168/8211/8411 (la RTL8111H integrada en la placa, ID `10ec:8168`), interfaz `enp2s0` en UP con el driver `r8169`. El driver dice `Link is Up - 1Gbps/Full`, o sea 1 Gbit/s en dúplex completo (`ipbr.txt`, `lspci.txt`, `velocidad.txt`) | Para evaluación vale con una tarjeta. Lo recomendado son tarjetas Gigabit redundantes (requisitos de Proxmox) | Cumple el mínimo y es Gigabit, pero no hay redundancia. El driver `r8169` viene en el kernel y en el Debian Live funciona. Comprobaré en la instalación que Proxmox también la reconoce | Como solo hay una tarjeta, la gestión de Proxmox y el tráfico del servicio van por la misma. |
| Plataforma | Placa ASUS H110I-PLUS (`lspci.txt`). BIOS AMI versión 0810 del 09/08/2017, con "UEFI is supported" (`bios.txt`). El equipo arrancó en UEFI, porque `efivarfs` está montado (`df.txt`). VT-x se ve en el sistema (`lscpu.txt`) | Proxmox necesita CPU y placa con VT-x para KVM (web de requisitos). Documentación de ASUS en Fuentes técnicas | Que Intel diga que lo "soporta" no es lo mismo que que esté "activado" en la BIOS. Aquí está soportado y, por lo que veo, activado. La BIOS es antigua (2017) y en `lscpu.txt` salen varias vulnerabilidades como "Vulnerable: ... no microcode" (MDS, SRBDS, MMIO stale data). Esto lo miraré en la fase de seguridad (microcódigo y BIOS) | Sin VT-x activado no podría usar KVM, solo contenedores LXC. |

---

## 1. ¿Puedo instalar Proxmox en este equipo?

Contrastar el inventario con requisitos oficiales de la versión prevista.
Responder por separado y justificar con datos y fuentes:

- Puedo instalar y arrancar Proxmox: Sí, con condiciones. Cumple los mínimos de Proxmox para evaluación (CPU de 64 bits con VT-x, más de 1 GB de RAM, un disco y una tarjeta de red), así que la instalación debería ir bien. De lo recomendado cumplo la CPU y los 2 GB de RAM, pero no el almacenamiento (es un disco mecánico, uno solo y sin redundancia) ni la red (solo hay una tarjeta).
- Puedo ejecutar el servidor ASO y el cliente del proyecto: Sí, con condiciones. Con el reparto de la pregunta 2 me quedan unos 1,6 GiB de RAM y 322 GiB de disco libres teniendo Proxmox, `srv-aso` y `cli-linux` encendidos a la vez, siempre que no les meta mucha carga. Las condiciones son usar LXC cuando se pueda, no clonar ni hacer copias mientras uso el cliente (con el disco mecánico se nota mucho) y comprobar cómo va de verdad en la sesión 5.
- Condiciones o comprobaciones pendientes: Ya he comprobado con las salidas que se ve VT-x (`lscpu.txt`), que hay 2 módulos de 4 GB en dual channel (`memory.txt`), que el driver `r8169` funciona y la red va a 1 Gbit/s (`velocidad.txt`), que la BIOS es la 0810 con UEFI (`bios.txt`) y que el disco tiene el SMART bien (`smartctl.txt`). Me queda comprobar que el instalador de Proxmox reconoce la tarjeta de red, decidir qué hago con los 317 GiB de datos del disco antes de formatearlo y ver el rendimiento real con los dos invitados encendidos (sesión 5).
- Fuentes oficiales Proxmox: https://www.proxmox.com/en/products/proxmox-virtual-environment/requirements

---

## 2. ¿Qué limitaciones tendré con este hardware?

Analizar RAM, CPU, disco y red para las cargas previstas. Identificar lo
medido y lo estimado. En VirtualBox, investigar host real y VM: versión,
RAM reservada al anfitrión, vCPU, VDI, espacio físico y red local.
No se utiliza el Proxmox del centro como recurso.

Modalidad Física:
Después de mirar el equipo, lo que veo más flojo es el procesador y el disco duro. Dos núcleos para virtualizar es muy poco: el host y los invitados se van a repartir los 4 hilos, así que no podré tener varias máquinas trabajando a tope a la vez. Para mi proyecto, que solo son dos invitados (`srv-aso` y `cli-linux`) sin mucha carga, creo que me llega.

La RAM son 8 GB DDR4 en dos módulos de 4 GB en dual channel (se ven 7,6 GiB). No es mucho para virtualizar, pero para esta práctica creo que va a ser suficiente y no me va a hacer de cuello de botella.

El disco es un HDD TOSHIBA DT01ACA050 de 500 GB (465,8 GiB) a 7200 rpm por SATA, con el SMART bien. De espacio va sobrado para el sistema y varias máquinas, pero un disco mecánico es mucho más lento que un SSD, así que arrancar, clonar o copiar máquinas va a ser lo más lento del laboratorio. Y como solo hay un disco, no hay redundancia.

La red es una Realtek RTL8111H integrada que va a 1 Gbit/s en dúplex completo, así que para lo que voy a hacer debería ir bien.

VirtualBox: No procede

### Reparto de recursos

| Elemento | CPU/vCPU | RAM | Disco | Justificación |
|---|---|---|---|---|
| Proxmox | Compartida (2 núcleos / 4 hilos) | 2 GB | 16 GB | Es lo que recomienda Proxmox en su web (2 GB para el sistema) y así le dejo algo de margen para que vaya bien. |
| srv-aso | 1 vCPU | 2 GB | 10 GB | Lo haré con LXC, porque un contenedor usa el kernel del host y gasta menos RAM y CPU que una VM. Lo confirmaré cuando elija el servicio ASO. |
| cli-linux | 1 vCPU | 1 GB | 8 GB | Una VM pequeña (o un LXC) solo para probar el servicio desde un cliente. |
| Plantilla, ISOs y copias | No aplica | No aplica | 40 GB | Para las ISOs, la plantilla Linux y las copias del servicio. Lo ajustaré con el tamaño real de las ISOs en la sesión 5. |
| Margen de operación | Sin reserva fija | 1 GB | 70 GB (≈15 %) | Para no quedarme sin memoria en el host y tener sitio para clonar y hacer copias. |

Mostrar cálculos de RAM y disco, reparto de vCPU, unidades y supuestos.
Explicar las cargas que pueden funcionar simultáneamente.

**Cálculo de RAM:** 7,6 GiB que se ven (`free -h`) − (2 GB de Proxmox + 2 GB de srv-aso + 1 GB de cli-linux + 1 GB de margen) = **1,6 GiB libres**. Aquí cuento GB y GiB como si fueran lo mismo, para simplificar.

**Cálculo de disco:** 465,8 GiB − (16 + 10 + 8 + 40 + 70 GiB) = 465,8 − 144 = **321,8 GiB libres (unos 322 GiB)**.

**Reparto de vCPU:** el host tiene 2 núcleos / 4 hilos. A cada invitado le doy 1 vCPU, así que el host y los dos invitados se reparten los hilos. Las vCPU no son núcleos solo para esa máquina: si varias trabajan a tope a la vez, se pelean por la CPU.

**Supuestos:** instalo con ext4 sobre LVM (lo que viene por defecto en el instalador) y no con ZFS, porque ZFS gasta más RAM para la caché y solo tengo un disco y poca RAM. Lo confirmaré en la instalación. Los invitados no van a tener mucha carga. Como el disco es mecánico, mejor contenedores LXC y clones enlazados de la plantilla (si se puede) para escribir menos en el disco.

**Cargas simultáneas:** Proxmox, `srv-aso` y `cli-linux` a la vez, sin mucha carga. Lo que no conviene es clonar, hacer copias o instalar cosas pesadas mientras uso el cliente, y menos con el disco mecánico. Lo probaré en la sesión 5.

| Limitación | Evidencia o estimación | Consecuencia | Medida y límite que permanece |
|---|---|---|---|
| Procesador: solo 2 núcleos / 4 hilos | Medido: `lscpu.txt`. Estimado: cómo va con carga | Si varias máquinas trabajan a la vez, todo irá más lento | Doy 1 vCPU por máquina y uso LXC siempre que pueda en vez de VMs. Aun así, si hay mucha carga a la vez, todo irá lento. |
| RAM: 7,6 GiB visibles (2 × 4 GB) | Medido: `memory.txt` y `free.txt`. El reparto es estimado | Me quedan unos 5,6 GiB para los invitados, así que no puedo encender muchas VMs, y si clono o hago una copia a la vez que uso el cliente puede ir justo de memoria | Dejo 1 GB de margen y hago `srv-aso` con LXC. Esto no me da más memoria: si usara VMs completas seguiría igual de limitado. La swap no sustituye a la RAM. |
| Disco mecánico (HDD 7200 rpm) | Medido: `lsblk.txt` (`ROTA=1`) y `smartctl.txt` (7200 rpm). Estimado: un HDD de 7200 rpm hace unas cien operaciones aleatorias por segundo y un SSD SATA decenas de miles (es un dato general, lo mediré en la sesión 5) | Arrancar, clonar y copiar va lento. Si varias máquinas usan el disco a la vez, tienen que esperar y todo parece lento aunque sobre CPU | Uso LXC, clones enlazados y hago copias y clones cuando no estoy usando el cliente. Lo lento del disco no se arregla así, solo con un SSD. |
| Disco único y con datos | Medido: `lsblk.txt` y `df.txt` (un solo disco con 317 GiB usados). El SMART hoy está bien (`smartctl.txt`) | No hay redundancia: si falla, pierdo el nodo, las VMs y las copias. Al instalar Proxmox se borran los datos que tiene | Copiar antes los datos que hay ahora y guardar las copias del laboratorio en otro sitio. Que el SMART esté bien no quiere decir que no vaya a fallar, y las copias en el mismo disco no sirven si se rompe el disco. |
| Red: una sola tarjeta a 1 Gbit/s | Medido: `ipbr.txt`, `lspci.txt` y `velocidad.txt` | La gestión de Proxmox y el servicio van por la misma red | Como el laboratorio tiene poco tráfico, no es problema. Pero no hay redundancia: si falla la tarjeta, me quedo sin acceso al nodo. |

---

## 3. ¿Qué componentes son más críticos en mi servidor y por qué?

Ordenar al menos tres componentes para este equipo y su servicio.
Distinguir cuello de botella, disponibilidad e integridad de los datos.

| Prioridad y componente | Función | Efecto si es insuficiente | Efecto si falla | Evidencia y justificación |
|---|---|---|---|---|
| 1. Disco duro mecánico (TOSHIBA DT01ACA050) | Guarda Proxmox, las VMs, los datos y las copias | Arrancar, clonar y copiar va lento, y las máquinas tienen que esperar al disco cuando trabajan a la vez | Pierdo el nodo, las VMs, los datos y las copias, porque está todo en el mismo disco | `lsblk.txt` y `smartctl.txt`: HDD de 7200 rpm, único y sin redundancia. Ahora está bien (SMART PASSED, 0 sectores reasignados, 5444 h), pero un disco mecánico puede fallar sin avisar. Es lo que más frena el laboratorio y lo que más daño hace si falla. |
| 2. CPU | Procesar todo | Las VMs van más lentas cuando varias trabajan a la vez | El equipo no arranca y el servicio se cae, pero no se pierden datos | `lscpu.txt`: solo 2 núcleos / 4 hilos, limita cuántas máquinas pueden trabajar a la vez. |
| 3. RAM | Memoria para el host y los invitados | Los procesos van lentos y se pueden cerrar por falta de memoria | El equipo se cuelga o no arranca y se pierde lo que no estuviera guardado | `memory.txt` y `free.txt`: 2 × 4 GB en dual channel, unos 5,6 GiB para los invitados. |

**Cuello de botella y lo que más daño hace si falla:** en mi equipo el disco es las dos cosas, porque es lo que más frena el laboratorio y también lo peor que puede fallar. La CPU es lo que marca cuántas máquinas pueden trabajar a la vez. Si falla la CPU o la RAM, el servicio se para hasta que cambie la pieza, pero si falla el disco puedo perder los datos, las VMs y las copias.

Componente que mejoraría primero y motivo para priorizarlo frente a otro: el disco, cambiando el HDD por un SSD SATA. Es lo que más se nota en un laboratorio de virtualización (arrancar, clonar y copiar), no tiene piezas que se muevan y es más barato que cambiar la CPU o la placa.

Lo segundo sería la CPU. Con dos núcleos voy muy limitado, y con un procesador de al menos 4 núcleos podría tener casi el doble de máquinas funcionando sin que se queden sin CPU. En la lista de CPU compatibles de ASUS para esta placa hay procesadores de 4 núcleos como el Core i5-7400, i5-7500, i5-7600 o i7-7700 (antes habría que mirar qué versión de BIOS pide ASUS para cada uno). Pongo antes el disco que la CPU porque con dos invitados que no hacen mucho la CPU me llega, pero el disco ya va lento solo con clonar una máquina.

Lo tercero sería la RAM. Como las 2 ranuras están ocupadas, tendría que cambiar los módulos por otros más grandes (la placa admite hasta 32 GB).

---

## Fuentes técnicas

Consultar al menos tres fuentes primarias, incluyendo Proxmox y
fabricante o plataforma virtual. Relacionarlas con las tres respuestas.

| Fuente y URL | Versión/modelo y fecha de consulta | Afirmación respaldada | Aplicación al nodo |
|---|---|---|---|
| Proxmox VE, requisitos: https://www.proxmox.com/en/products/proxmox-virtual-environment/requirements | Versión: 9.2 (confirmar al descargar la ISO). Consultado: 07/10/2026 | Mínimo para evaluación: CPU de 64 bits con VT, 1 GB de RAM, disco y una tarjeta de red. Recomendado: 2 GB de RAM para el sistema, almacenamiento rápido y redundante (mejor SSD) y tarjetas Gigabit redundantes | La uso en la pregunta 1 y para comparar la RAM, el disco y la red del inventario |
| Intel ARK del G4560: https://www.intel.la/content/www/xl/es/products/sku/97143/intel-pentium-processor-g4560-3m-cache-3-50-ghz/specifications.html | Modelo: Pentium G4560. Consultado: 13/10/2026 | 2 núcleos / 4 hilos, VT-x y VT-d, socket FCLGA1151, DDR4-2133/2400, 2 canales y hasta 64 GB | La uso para la limitación de CPU de la pregunta 2, la RAM y la prioridad 2 de la pregunta 3 |
| Toshiba, serie DT01ACA (cliente HDD): https://toshiba.semicon-storage.com/it/product/storage-products/client-hdd/dt01acaxxx.html | Modelo: DT01ACA050. Consultado: 13/10/2026 | Disco duro de 500 GB, 7200 rpm, SATA 6 Gbit/s, 3,5 pulgadas | La uso para demostrar que el disco es mecánico, para la limitación de la pregunta 2 y para la prioridad 1 de la pregunta 3 |
| ASUS, soporte de la placa H110I-PLUS: documentación https://www.asus.com/supportonly/h110i-plus/helpdesk_manual/ y CPU compatibles https://www.asus.com/supportonly/h110i-plus/helpdesk_cpu/ | Modelo: H110I-PLUS, manual de usuario E11036 (10/11/2015) y manual de actualización de BIOS E13571 (24/10/2017). Consultado: 13/10/2026 | La placa tiene documentación oficial y la BIOS se puede actualizar. En la lista de CPU compatibles está el Pentium G4560 y procesadores de 6.ª y 7.ª generación (Celeron, Pentium, Core i3, i5 e i7) del socket LGA1151 | La uso para el modelo de placa y la BIOS 0810 del inventario, y para las opciones de mejorar la CPU de la pregunta 3 |
| Ficha de una tienda (fuente secundaria): https://www.voelkner.de/products/1059543/Asus-H110I-PLUS-Mainboard-Sockel-PC-Intel-LGA-1151-Formfaktor-Details-Mini-ITX-Mainboard-Chipsatz-Intel-H110.html | Modelo: H110I-PLUS. Consultado: 13/10/2026 | Placa mini-ITX, chipset Intel H110, 2 ranuras DDR4 hasta 2400 MHz y 32 GB como máximo | La uso para la RAM del inventario y para la idea de ampliar cambiando los módulos. Como es de una tienda, habría que contrastarlo con el manual E11036 de ASUS |

---

## Dictamen de viabilidad

Apto, apto con condiciones o no apto para la propuesta actual: Apto con condiciones.

Sintetizar las tres respuestas y las condiciones pendientes de comprobar.

El equipo cumple los requisitos mínimos de Proxmox y, de lo recomendado, la CPU y los 2 GB de RAM para el sistema, así que según la web de Proxmox debería poder ejecutarlo sin problemas. Lo que no cumple es lo recomendado en almacenamiento (es un disco mecánico, uno solo y sin redundancia) ni la redundancia de red. Por el procesador y el disco no espero que vaya muy fluido: tendré que "dosificar" la CPU para no sobrecargarla y no clonar ni hacer copias mientras uso el cliente. Me queda comprobar que el instalador reconoce la tarjeta de red, decidir qué hago con los 317 GiB de datos del disco antes de formatearlo y ver cómo va de verdad con `srv-aso` y `cli-linux` encendidos a la vez (sesión 5).

---

## Evidencias y revisión docente

Enlazar salidas revisadas desde ../evidencias/. Un listado automático no
sustituye la investigación. Registrar la pregunta de defensa y respuesta.

- [lscpu](../evidencias/capturas-o-salidas/lscpu.txt)
- [free](../evidencias/capturas-o-salidas/free.txt)
- [memory (dmidecode)](../evidencias/capturas-o-salidas/memory.txt)
- [lsblk](../evidencias/capturas-o-salidas/lsblk.txt)
- [df](../evidencias/capturas-o-salidas/df.txt)
- [smartctl](../evidencias/capturas-o-salidas/smartctl.txt)
- [ip -br addr](../evidencias/capturas-o-salidas/ipbr.txt)
- [lspci](../evidencias/capturas-o-salidas/lspci.txt)
- [velocidad de red (dmesg)](../evidencias/capturas-o-salidas/velocidad.txt)
- [bios (dmidecode)](../evidencias/capturas-o-salidas/bios.txt)
- [systemd-detect-virt](../evidencias/capturas-o-salidas/systemd.txt)

**Pregunta de defensa:** ¿Qué cambiaría si tuviera 4 GB menos de RAM?

**Mi respuesta:** Con 4 GB menos me quedaría un solo módulo de 4 GB, unos 3,6 GiB visibles (7,6 − 4), y encima en single channel. Mi reparto necesita 6 GB (2 de Proxmox, 2 de `srv-aso`, 1 de `cli-linux` y 1 de margen), así que me faltarían unos 2,4 GiB y no podría tener los dos invitados encendidos a la vez. La RAM pasaría a ser el cuello de botella principal, por delante de la CPU, porque me limitaría directamente cuántas máquinas puedo tener funcionando.

Tendría que apretar más: dejar 2 GB para Proxmox, hacer `srv-aso` como LXC con 1 GB (que es el servicio principal) y encender `cli-linux` solo cuando haga pruebas, o hacerlo un LXC de unos 512 MB. Aun así iría muy justo de margen, así que no podría clonar ni hacer copias mientras uso el cliente. Con esto gasto menos, pero no tengo más memoria: seguiría sin caber todo a la vez.

La swap no la usaría como solución, porque no es RAM de verdad. Además mi disco es mecánico, y si el sistema empieza a tirar de swap ahí, todo el laboratorio iría lentísimo. Como mucho serviría de último recurso para que no se cierre nada por falta de memoria. La solución de verdad sería volver a poner más RAM.

[Volver al índice](../README.md)
