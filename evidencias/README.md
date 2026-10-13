# Evidencias

Salidas de comandos de la Fase 1, guardadas en `capturas-o-salidas/`. Se obtuvieron el 13/10/2026 desde Debian Live en el equipo físico. Se han eliminado números de serie, identificadores de disco (WWN) y el UUID del sistema de archivos antes de publicarlas.

| Fichero | Comando | Qué demuestra | Documento que lo usa |
|---|---|---|---|
| [lscpu.txt](capturas-o-salidas/lscpu.txt) | `lscpu` | CPU Pentium G4560, 2 núcleos y 4 hilos, VT-x y flag `vmx` | [Inventario](../docs/01-inventario-hardware.md) |
| [free.txt](capturas-o-salidas/free.txt) | `free -h` | 7,6 GiB de RAM visibles y swap a 0 en el Live | [Inventario](../docs/01-inventario-hardware.md) |
| [memory.txt](capturas-o-salidas/memory.txt) | `dmidecode -t memory` | 2 módulos de 4 GB DDR4 a 2400 MT/s en dual channel | [Inventario](../docs/01-inventario-hardware.md) |
| [lsblk.txt](capturas-o-salidas/lsblk.txt) | `lsblk -o NAME,TYPE,SIZE,MODEL,ROTA,TRAN` | Disco TOSHIBA DT01ACA050 rotacional (`ROTA=1`) por SATA | [Inventario](../docs/01-inventario-hardware.md) |
| [df.txt](capturas-o-salidas/df.txt) | `df -h` | Partición `sda1` con 317 GiB usados y arranque en UEFI (`efivarfs`) | [Inventario](../docs/01-inventario-hardware.md) |
| [smartctl.txt](capturas-o-salidas/smartctl.txt) | `smartctl -i -H -A /dev/sda` | 7200 rpm, SMART PASSED, 0 sectores reasignados, 5444 horas de uso | [Inventario](../docs/01-inventario-hardware.md) |
| [ipbr.txt](capturas-o-salidas/ipbr.txt) | `ip -br addr` | Interfaz `enp2s0` en estado UP | [Inventario](../docs/01-inventario-hardware.md) |
| [lspci.txt](capturas-o-salidas/lspci.txt) | `lspci -nnk` | Placa ASUS H110I-PLUS y Realtek RTL8111H con driver `r8169` | [Inventario](../docs/01-inventario-hardware.md) |
| [velocidad.txt](capturas-o-salidas/velocidad.txt) | `dmesg` filtrado por `link is up` | Enlace a 1 Gbps en dúplex completo | [Inventario](../docs/01-inventario-hardware.md) |
| [bios.txt](capturas-o-salidas/bios.txt) | `dmidecode -t bios` | BIOS AMI versión 0810 con UEFI soportado | [Inventario](../docs/01-inventario-hardware.md) |
| [systemd.txt](capturas-o-salidas/systemd.txt) | `systemd-detect-virt` | `none`: equipo físico, no una VM | [Inventario](../docs/01-inventario-hardware.md) |

[Volver al índice](../README.md)
