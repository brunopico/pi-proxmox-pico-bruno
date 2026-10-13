# Inventario investigado y viabilidad

Bruno Picó 2X ASIR

## Contexto de la observación

- Fecha y sistema utilizado: 13/10/2026, Debian Live (Debian 13, kernel 6.12.94) arrancado desde USB con Ventoy. `systemd-detect-virt` devuelve `none`: se observa el equipo físico y no una VM (evidencia: `systemd.txt`). Primera revisión de especificaciones el 07/10/2026.
- Modalidad, equipo anfitrión y VM VirtualBox si procede: Física, equipo real. VirtualBox: No procede.
- Versión de Proxmox prevista: Instalaremos la versión 9.2 (confirmar al descargar la ISO).

---

## Inventario inicial

| Área | Dato observado y evidencia | Especificación y fuente | Límite o diferencia | Justificación |
|---|---|---|---|---|
| CPU | Intel Pentium G4560 a 3,50 GHz, x86_64 (64 bits), 1 socket, 2 núcleos con 2 hilos cada uno (4 CPU lógicas). `lscpu` indica `Virtualization: VT-x` y el flag `vmx` aparece en la lista. Evidencia: `lscpu.txt` | Proxmox: Intel 64 o AMD64 con el flag Intel VT o AMD-V (web de requisitos). Intel ARK del G4560: 2 núcleos, 4 hilos, VT-x: Sí, VT-d: Sí, socket FCLGA1151, memoria DDR4-2133/2400 y hasta 2 canales | Cumple los requisitos pedidos. Que el flag `vmx` sea visible indica que VT-x está habilitado en la BIOS (lo normal es que el kernel no lo muestre si la BIOS lo desactiva); se confirmará al arrancar KVM en la instalación. Solo hay 2 núcleos físicos: los 4 hilos no equivalen a 4 núcleos | En la web de Proxmox no se indican mínimos de núcleos. En nuestro caso, al tener solo dos núcleos físicos (4 hilos), habrá limitaciones claras para virtualizar, ya que host e invitados comparten esa capacidad de procesamiento. |
| RAM | 8 GB DDR4 formados por 2 módulos CRUCIAL CT4G4DFS824A de 4 GB a 2400 MT/s, uno en `ChannelA-DIMM1` y otro en `ChannelB-DIMM1` (dual channel, sin ECC). El sistema ve 7,6 GiB (`free -h`); la diferencia la reservan el firmware y, probablemente, la gráfica integrada. Swap: 0 B (en el Live). Evidencia: `memory.txt`, `free.txt` | Proxmox recomienda como mínimo 2 GB de RAM para el sistema y sus servicios, más la memoria de los invitados. Mínimo para evaluación: 1 GB. Intel ARK: la CPU admite DDR4-2133/2400, 2 canales y hasta 64 GB | Cumple el mínimo de evaluación y los 2 GB recomendados para el sistema. Descontados esos 2 GB, quedan unos 5,6 GiB para invitados y margen. Con dos módulos se aprovechan los dos canales de la CPU. La placa es mini-ITX con 2 ranuras DIMM DDR4 (hasta 2400 MHz y 32 GB en total, según la ficha de un distribuidor) y ambas están ocupadas (el SMBIOS declara 4 posiciones, pero solo 2 son ranuras físicas), por lo que ampliar la RAM exigiría sustituir los módulos, por ejemplo por 2 × 8 GB | La RAM no es nuestro principal cuello de botella, pero sí limita la memoria disponible para los invitados. Los 2,4 GiB «usados» del Live no representan a Proxmox (el sistema Live y su overlay viven en RAM). |
| Almacenamiento | Disco duro mecánico (HDD) TOSHIBA DT01ACA050: 500 GB (465,8 GiB), 3,5 pulgadas, 7200 rpm, SATA 3.0 negociado a 6,0 Gb/s, sectores de 512 B lógicos y 4096 B físicos; `ROTA=1`. **No es un SSD.** SMART: PASSED, 0 sectores reasignados, 0 pendientes, 5444 horas encendido (≈227 días) y 35 ºC. Una única partición (`sda1`, 458 GiB de sistema de archivos) con 317 GiB usados (73 %): contiene datos. Evidencia: `lsblk.txt`, `smartctl.txt`, `df.txt` | Proxmox recomienda almacenamiento rápido y redundante, con mejores resultados en SSD; mínimo de evaluación: un disco duro. Fabricante (Toshiba, serie DT01ACA): 500 GB, 7200 rpm, SATA 6 Gbit/s, 3,5 pulgadas | Cumple el mínimo (es un disco duro) y sobra capacidad. No cumple lo recomendado: ni es SSD ni hay redundancia | Hay espacio de sobra y el disco está sano ahora mismo, pero un disco mecánico es mucho más lento que un SSD en accesos aleatorios (arranques, clonados y copias) y, al ser único, si falla se pierden Proxmox, las VMs, los datos y las copias locales. Instalar Proxmox borra el disco entero, incluidos los 317 GiB actuales. |
| Red | Controlador Ethernet Realtek RTL8111/8168/8211/8411 (RTL8111H integrado en la placa, ID `10ec:8168`), interfaz `enp2s0` en estado UP, driver en uso `r8169`. El driver informa de `Link is Up - 1Gbps/Full`: 1 Gbit/s en dúplex completo. Evidencia: `ipbr.txt`, `lspci.txt`, `velocidad.txt` | Mínimo para evaluación: una NIC. Recomendado: NIC Gbit redundantes (web de requisitos de Proxmox) | Cumple el mínimo (una NIC) y es Gigabit. No cumple la recomendación de redundancia. El driver `r8169` es del propio kernel de Linux y funciona en Debian Live; se confirmará que Proxmox reconoce la tarjeta durante la instalación | Un solo adaptador: la gestión de Proxmox y el tráfico del servicio comparten la misma interfaz. |
| Plataforma | Placa ASUS H110I-PLUS (`lspci.txt`). Firmware AMI versión 0810, fechado el 09/08/2017, con «UEFI is supported» (`bios.txt`); el sistema arrancó en UEFI (`efivarfs` montado en `df.txt`). VT-x visible en el sistema (`lscpu.txt`) | Proxmox exige CPU y placa con VT-x para KVM (web de requisitos). Documentación del fabricante de la placa: ver Fuentes técnicas | «Soportado» (lo dice Intel) no es lo mismo que «activado» (lo dice la BIOS): aquí está soportado y, por lo observado, activado. La BIOS es antigua (2017) y `lscpu.txt` marca varias entradas como «Vulnerable: ... no microcode» (MDS, SRBDS, MMIO stale data): se tratará en la fase de seguridad (microcódigo y BIOS) | Sin VT-x activado no habría virtualización completa con KVM, solo contenedores LXC. |

---

## 1. ¿Puedo instalar Proxmox en este equipo?

Contrastar el inventario con requisitos oficiales de la versión prevista.
Responder por separado y justificar con datos y fuentes:

- Puedo instalar y arrancar Proxmox: Sí, con condiciones. El equipo cumple los requisitos mínimos para evaluación de Proxmox (CPU de 64 bits con VT-x, más de 1 GB de RAM, un disco duro y una NIC), por lo que la instalación debe ser posible. De los requisitos recomendados cumple la CPU y la RAM base de 2 GB, pero no cumple el almacenamiento rápido y redundante (el disco es un HDD mecánico, único y sin redundancia) ni las NIC redundantes (una sola tarjeta de red).
- Puedo ejecutar el servidor ASO y el cliente del proyecto: Sí, con condiciones. El reparto de la pregunta 2 deja unos 1,6 GiB de RAM y 322 GiB de disco libres con Proxmox, `srv-aso` y `cli-linux` encendidos a la vez, siempre con cargas ligeras. Las condiciones son usar LXC cuando sea posible, no clonar ni hacer copias mientras se usa el cliente (el disco mecánico lo penaliza) y confirmar el rendimiento real en la sesión 5.
- Condiciones o comprobaciones pendientes: COMPROBADO con las salidas: VT-x visible (`lscpu.txt`), 2 módulos de 4 GB en dual channel (`memory.txt`), driver `r8169` reconocido y enlace a 1 Gbit/s (`velocidad.txt`), BIOS 0810 con UEFI (`bios.txt`) y disco con SMART correcto (`smartctl.txt`). PENDIENTE: que el instalador de Proxmox reconozca la tarjeta de red, resolver qué hacer con los 317 GiB de datos actuales del disco antes de formatearlo, y medir el rendimiento real con los dos invitados encendidos (sesión 5).
- Fuentes oficiales Proxmox: https://www.proxmox.com/en/products/proxmox-virtual-environment/requirements

---

## 2. ¿Qué limitaciones tendré con este hardware?

Analizar RAM, CPU, disco y red para las cargas previstas. Identificar lo
medido y lo estimado. En VirtualBox, investigar host real y VM: versión,
RAM reservada al anfitrión, vCPU, VDI, espacio físico y red local.
No se utiliza el Proxmox del centro como recurso.

Modalidad Física:
Después de analizar el equipo físico hemos marcado como puntos más débiles el procesador y el disco duro mecánico. Dos núcleos físicos para virtualizar es algo muy limitado: host e invitados compartirán los 4 hilos, por lo que no será posible mantener varias máquinas virtuales trabajando a la vez al máximo. Para nuestro proyecto, que necesita solo dos invitados (`srv-aso` y `cli-linux`) con cargas ligeras, debería ser suficiente. Los 8 GB de RAM DDR4 están repartidos en dos módulos de 4 GB en dual channel (7,6 GiB visibles): no es una gran cantidad para virtualizar, pero para esta práctica esperamos que sea suficiente y no creemos que haga cuello de botella. El disco es un HDD TOSHIBA DT01ACA050 de 500 GB (465,8 GiB) a 7200 rpm por SATA, con SMART correcto: tiene capacidad de sobra para el sistema y varias máquinas virtuales, pero un disco mecánico es mucho más lento que un SSD en accesos aleatorios, de modo que arrancar, clonar o copiar máquinas será lo más lento del laboratorio, y al ser un único disco no hay redundancia. La red cuenta con un adaptador Realtek RTL8111H integrado que negocia 1 Gbit/s en dúplex completo, por lo que debería ser capaz para esta práctica y las cargas esperadas.

VirtualBox: No procede

### Reparto de recursos

| Elemento | CPU/vCPU | RAM | Disco | Justificación |
|---|---|---|---|---|
| Proxmox | Compartida (2 núcleos / 4 hilos) | 2 GB | 16 GB | Son los requisitos recomendados que tiene Proxmox en su página (2 GB para el sistema y sus servicios), además nos permite tener cierto margen para que el sistema funcione correctamente. |
| srv-aso | 1 vCPU | 2 GB | 10 GB | LXC: un contenedor comparte el kernel del host y consume menos RAM y CPU que una VM completa. Se confirmará al elegir el servicio ASO definitivo. |
| cli-linux | 1 vCPU | 1 GB | 8 GB | VM ligera (o LXC) solo para probar el servicio desde un cliente. |
| Plantilla, ISOs y copias | No aplica | No aplica | 40 GB | ISOs, plantilla Linux y copias locales del servicio. Se ajustará al tamaño real de las ISOs en la sesión 5. |
| Margen de operación | Sin reserva fija | 1 GB | 70 GB (≈15 %) | Evitar quedarnos sin memoria en el host y dejar espacio para clonar y para las copias. |

Mostrar cálculos de RAM y disco, reparto de vCPU, unidades y supuestos.
Explicar las cargas que pueden funcionar simultáneamente.

**Cálculo de RAM:** 7,6 GiB visibles (`free -h`) − (2 GB Proxmox + 2 GB srv-aso + 1 GB cli-linux + 1 GB de margen) = **1,6 GiB libres** (en este reparto se tratan GB y GiB como equivalentes aproximados).

**Cálculo de disco:** 465,8 GiB − (16 + 10 + 8 + 40 + 70 GiB) = 465,8 − 144 = **321,8 GiB libres (≈322 GiB)**.

**Reparto de vCPU:** el host tiene 2 núcleos / 4 hilos. Asigno 1 vCPU a cada invitado, de modo que host y los dos invitados se reparten los hilos lógicos. Las vCPU no son núcleos exclusivos: si varias máquinas trabajan a la vez al máximo, competirán por la CPU.

**Supuestos:** instalación con ext4 sobre LVM (la opción por defecto del instalador) en lugar de ZFS, porque ZFS consume más RAM para su caché y solo disponemos de un disco y poca RAM; se confirmará durante la instalación. Uso ligero de los invitados. Como el disco es mecánico, se prefieren contenedores LXC y clones enlazados desde la plantilla (si el almacenamiento lo permite) para reducir las escrituras.

**Cargas simultáneas:** Proxmox, `srv-aso` y `cli-linux` a la vez, con uso ligero. No conviene clonar, hacer copias o instalar paquetes pesados mientras se usa el cliente, y menos aún con el disco mecánico. Se comprobará con pruebas en la sesión 5.

| Limitación | Evidencia o estimación | Consecuencia | Medida y límite que permanece |
|---|---|---|---|
| Procesador: solo 2 núcleos / 4 hilos | Medido: `lscpu.txt`. Estimado: rendimiento bajo carga | Posible rendimiento limitado y caída de rendimiento en las máquinas virtuales si varias trabajan a la vez | Limitación a 1 vCPU por VM y uso de LXC siempre que sea posible en vez de VMs completas para reducir la carga. No resuelve que, con mucha carga a la vez, todo vaya más lento. |
| RAM: 7,6 GiB visibles (2 × 4 GB) | Medido: `memory.txt` y `free.txt`. Reparto: estimado | Unos 5,6 GiB para invitados y margen: no se pueden encender muchas VMs, y una clonación o una copia pesada a la vez que el cliente puede apretar la memoria | Reservar 1 GB de margen y usar LXC para `srv-aso`. No añade memoria: el límite permanece si usara VMs completas. La swap no sustituye a la RAM. |
| Disco mecánico (HDD 7200 rpm) | Medido: `lsblk.txt` (`ROTA=1`) y `smartctl.txt` (7200 rpm). Estimado: un HDD de 7200 rpm hace del orden de cien operaciones aleatorias por segundo, frente a decenas de miles de un SSD SATA (referencia genérica, por medir en la sesión 5) | Arranques, clonados y copias lentos; si varias máquinas acceden a la vez, esperan al disco y el laboratorio se siente lento aunque sobre CPU | Contenedores LXC, clones enlazados, y copias y clonados fuera del uso del cliente. No resuelve la latencia del disco mecánico: solo un SSD lo hace. |
| Disco único y con datos previos | Medido: `lsblk.txt` y `df.txt` (un solo disco, 317 GiB usados). SMART correcto hoy (`smartctl.txt`) | Sin redundancia: si falla, se pierden el nodo, las VMs y las copias locales. Instalar Proxmox borra los datos actuales | Copia de los datos actuales antes de instalar, y copias de seguridad del laboratorio en otro destino. Un SMART correcto no garantiza que el disco no falle: las copias locales no protegen ante un fallo del propio disco. |
| Red: una sola NIC a 1 Gbit/s | Medido: `ipbr.txt`, `lspci.txt` y `velocidad.txt` | La gestión de Proxmox y el servicio comparten red | Mantener un laboratorio con poco tráfico. No hay redundancia: si falla la NIC, no hay acceso al nodo. |

---

## 3. ¿Qué componentes son más críticos en mi servidor y por qué?

Ordenar al menos tres componentes para este equipo y su servicio.
Distinguir cuello de botella, disponibilidad e integridad de los datos.

| Prioridad y componente | Función | Efecto si es insuficiente | Efecto si falla | Evidencia y justificación |
|---|---|---|---|---|
| 1. Disco duro mecánico (TOSHIBA DT01ACA050) | Almacenamiento no volátil de Proxmox, VMs, datos y copias locales | Arranques, clonados y copias lentos, y máquinas esperando al disco cuando trabajan a la vez | Pérdida del nodo, de las VMs, de los datos y de las copias locales (todo está en el mismo disco) | `lsblk.txt` y `smartctl.txt`: HDD de 7200 rpm, único y sin redundancia; hoy está sano (SMART PASSED, 0 sectores reasignados, 5444 h), pero un HDD puede fallar sin avisar. Es lo que más ralentiza el laboratorio y su fallo es el de mayor impacto en disponibilidad e integridad de los datos. |
| 2. CPU | Procesamiento | Caídas o bajadas de rendimiento en las VMs cuando varias trabajan a la vez | El equipo no arranca; el servicio queda caído, pero no se pierden datos | `lscpu.txt`: solo 2 núcleos / 4 hilos, limita cuántas máquinas pueden trabajar a la vez. |
| 3. RAM | Memoria de los procesos del host y los invitados | Retraso en los procesos, mal rendimiento y posibles cierres por falta de memoria | El equipo se cuelga o no arranca; se pierde lo que no estuviera guardado | `memory.txt` y `free.txt`: 2 × 4 GB en dual channel, unos 5,6 GiB para invitados y margen. |

**Cuello de botella frente a mayor impacto si falla:** con este equipo, el disco concentra las dos cosas en el día a día (es lo que más frena el laboratorio y lo que más daño hace si falla), mientras que la CPU marca el límite de máquinas que pueden trabajar a la vez. Un fallo de CPU o de RAM deja el servicio parado hasta sustituir la pieza, mientras que un fallo del disco puede perder los datos, las VMs y las copias locales.

Componente que mejoraría primero y motivo para priorizarlo frente a otro: el disco, sustituyendo el HDD por un SSD SATA. Es la mejora que más se nota en un laboratorio de virtualización (arranques, clonados y copias), elimina las partes móviles de un disco mecánico y es más barata que cambiar CPU o placa. La CPU sería la segunda mejora: dos núcleos para virtualizar limitan enormemente el sistema, y un procesador con al menos 4 núcleos permitiría tener prácticamente el doble de máquinas virtuales en funcionamiento sin riesgo de que se queden sin CPU (la lista de CPU compatibles de ASUS para esta placa incluye procesadores de 4 núcleos como Core i5-7400, i5-7500, i5-7600 o i7-7700, comprobando antes la versión mínima de BIOS que ASUS indica para cada uno). Se prioriza el disco frente a la CPU porque con solo dos invitados ligeros la CPU alcanza, mientras que el disco ya es lento con una sola máquina clonándose. La RAM sería la tercera mejora: con las 2 ranuras ocupadas habría que sustituir los módulos por otros de mayor capacidad (la placa admite hasta 32 GB).

---

## Fuentes técnicas

Consultar al menos tres fuentes primarias, incluyendo Proxmox y
fabricante o plataforma virtual. Relacionarlas con las tres respuestas.

| Fuente y URL | Versión/modelo y fecha de consulta | Afirmación respaldada | Aplicación al nodo |
|---|---|---|---|
| Proxmox VE, requisitos: https://www.proxmox.com/en/products/proxmox-virtual-environment/requirements | Versión: 9.2 (confirmar al descargar la ISO). Consultado: 07/10/2026 | Mínimo para evaluación: CPU de 64 bits con VT, 1 GB de RAM, disco y una NIC. Recomendado: 2 GB de RAM para el sistema, almacenamiento rápido y redundante (mejor con SSD) y NIC Gbit redundantes | Respalda la respuesta de la pregunta 1 y la comparación de RAM, disco y red del inventario |
| Intel ARK del G4560: https://www.intel.la/content/www/xl/es/products/sku/97143/intel-pentium-processor-g4560-3m-cache-3-50-ghz/specifications.html | Modelo: Pentium G4560. Consultado: 13/10/2026 | 2 núcleos / 4 hilos, VT-x: Sí, VT-d: Sí, socket FCLGA1151, memoria DDR4-2133/2400, 2 canales y hasta 64 GB | Respalda la limitación de CPU de la pregunta 2, la comparación de RAM y la prioridad 2 de la pregunta 3 |
| Toshiba, serie DT01ACA (cliente HDD): https://toshiba.semicon-storage.com/it/product/storage-products/client-hdd/dt01acaxxx.html | Modelo: DT01ACA050. Consultado: 13/10/2026 | Disco duro de 500 GB, 7200 rpm, SATA 6 Gbit/s, 3,5 pulgadas | Respalda que el disco es mecánico, la limitación de rendimiento de la pregunta 2 y la prioridad 1 de la pregunta 3 |
| ASUS, soporte de la placa H110I-PLUS: documentación https://www.asus.com/supportonly/h110i-plus/helpdesk_manual/ y CPU compatibles https://www.asus.com/supportonly/h110i-plus/helpdesk_cpu/ | Modelo: H110I-PLUS, manual de usuario E11036 (10/11/2015) y manual de actualización de BIOS E13571 (24/10/2017). Consultado: 13/10/2026 | La placa tiene documentación oficial del fabricante y su BIOS es actualizable. La lista de CPU compatibles incluye el Pentium G4560 y procesadores de 6.ª y 7.ª generación (Celeron, Pentium, Core i3, i5 e i7) del socket LGA1151 | Respalda el modelo de placa y la BIOS 0810 del inventario, y las opciones de mejora de la CPU de la pregunta 3 |
| Ficha de distribuidor (fuente secundaria): https://www.voelkner.de/products/1059543/Asus-H110I-PLUS-Mainboard-Sockel-PC-Intel-LGA-1151-Formfaktor-Details-Mini-ITX-Mainboard-Chipsatz-Intel-H110.html | Modelo: H110I-PLUS. Consultado: 13/10/2026 | Placa mini-ITX, chipset Intel H110, 2 ranuras DIMM DDR4 hasta 2400 MHz y 32 GB como máximo | Respalda la RAM del inventario y la propuesta de ampliar sustituyendo módulos; por ser una fuente secundaria, conviene contrastarla con el manual E11036 de ASUS |

---

## Dictamen de viabilidad

Apto, apto con condiciones o no apto para la propuesta actual: Apto con condiciones.

Sintetizar las tres respuestas y las condiciones pendientes de comprobar.

El nodo físico que tenemos cumple los requisitos mínimos de Proxmox y la parte recomendada relativa a la CPU y a los 2 GB de RAM del sistema: según la propia página de Proxmox, nuestro nodo debe ser capaz de ejecutar el sistema con solvencia, ya que estamos por encima de los requisitos mínimos. No cumple lo recomendado en almacenamiento (el disco es un HDD mecánico, único y sin redundancia) ni la redundancia de red. Debido al procesador y al disco mecánico, no esperamos una fluidez muy alta: tenemos que «dosificar» la CPU para no sobrecargarla y evitar clonar o copiar mientras se usa el cliente. Condiciones pendientes: que el instalador de Proxmox reconozca la tarjeta de red, decidir qué se hace con los 317 GiB de datos actuales del disco antes de formatearlo, y comprobar el rendimiento real con `srv-aso` y `cli-linux` encendidos a la vez (sesión 5).

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

**Mi respuesta:** Con 4 GB menos me quedaría un solo módulo de 4 GB, es decir, unos 3,6 GiB visibles (7,6 − 4) y, además, en single channel. Mi reparto actual necesita 6 GB (2 de Proxmox, 2 de `srv-aso`, 1 de `cli-linux` y 1 de margen), así que faltarían unos 2,4 GiB y no podría mantener los dos invitados encendidos a la vez. La RAM pasaría a ser el cuello de botella principal, por delante de la CPU, porque limitaría directamente cuántas máquinas pueden funcionar al mismo tiempo.

Tendría que tomar medidas más estrictas: reservar 2 GB para Proxmox, dejar `srv-aso` como LXC con 1 GB (el servicio principal) y encender `cli-linux` solo durante las pruebas, o bajarlo a un LXC de unos 512 MB. Aun así el margen sería muy pequeño, por lo que no podría clonar ni hacer copias mientras se usa el cliente. Estas medidas reducen el consumo, pero no añaden memoria: el límite de que no caben todas las máquinas a la vez seguiría ahí.

No usaría la swap como solución, porque no es RAM adicional. Además, mi disco es un HDD mecánico, y si el sistema empieza a usar swap en él, todo el laboratorio se volvería muy lento. Como mucho serviría de último recurso para evitar que el sistema se cierre por falta de memoria. La solución real sería volver a ampliar la RAM.

[Volver al índice](../README.md)
