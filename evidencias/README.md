# Evidencias

## Fase 1

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

## Fases 3 y 4 en VirtualBox (Debian)

Salidas y capturas de cuando pasé el laboratorio a mi portátil con Debian y VirtualBox, guardadas en `virtualbox/`. Las salidas las saqué desde Debian o por SSH al nodo (`ssh root@192.168.20.10`). En las capturas no sale ninguna contraseña.

| Fichero | Comando o captura | Qué demuestra | Documento que lo usa |
|---|---|---|---|
| [host-lscpu.txt](virtualbox/host-lscpu.txt), [host-free.txt](virtualbox/host-free.txt), [host-lsblk.txt](virtualbox/host-lsblk.txt), [host-df.txt](virtualbox/host-df.txt), [host-debian.txt](virtualbox/host-debian.txt) | `lscpu`, `free -h`, `lsblk`, `df -h`, `cat /etc/debian_version` en Debian | Portátil con i5-10210U (VT-x), 7,6 GiB de RAM, NVMe de 930 GB y Debian 13.7 | [Instalación](../docs/03-instalacion-proxmox.md) |
| [host-vbox.txt](virtualbox/host-vbox.txt), [host-secureboot.txt](virtualbox/host-secureboot.txt) | `VBoxManage --version`, `mokutil --sb-state` | VirtualBox 7.2.22 y Secure Boot activado | [Instalación](../docs/03-instalacion-proxmox.md) |
| [iso-sha256.txt](virtualbox/iso-sha256.txt) | `sha256sum` de la ISO | La ISO de Proxmox 9.2 coincide con la de la web oficial | [Instalación](../docs/03-instalacion-proxmox.md) |
| [vbox-nested.txt](virtualbox/vbox-nested.txt), [vbox-vminfo.txt](virtualbox/vbox-vminfo.txt), [vbox-disco-fijo.txt](virtualbox/vbox-disco-fijo.txt) | `VBoxManage showvminfo` y `showmediuminfo` | VT-x anidado, paravirtualización KVM, virtio, modo promiscuo y disco fijo | [Instalación](../docs/03-instalacion-proxmox.md) |
| [pve-fase3.txt](virtualbox/pve-fase3.txt) | `hostnamectl`, `ip -br addr`, `pveversion`, `df -h`, `ls -l /dev/kvm` | Proxmox instalado como `pve-pico` y con KVM | [Instalación](../docs/03-instalacion-proxmox.md) |
| [pve-red-antes.txt](virtualbox/pve-red-antes.txt), [pve-red-despues.txt](virtualbox/pve-red-despues.txt) | `ip -br addr`, `ip route`, `/etc/network/interfaces`, `iptables` | La red antes y después de configurar `vmbr0`, `vmbr1` y el NAT | [Instalación](../docs/03-instalacion-proxmox.md) |
| [nat-simple-error.txt](virtualbox/nat-simple-error.txt) | `ping` al gateway con el NAT simple | El error del NAT simple de VirtualBox antes de cambiar a Red NAT | [Instalación](../docs/03-instalacion-proxmox.md) |
| [pve-conectividad.txt](virtualbox/pve-conectividad.txt), [pve-apt-update.txt](virtualbox/pve-apt-update.txt) | `ping`, `getent`, `apt update` | Salida a Internet, DNS y repositorios (primero con 401 y luego sin errores) | [Instalación](../docs/03-instalacion-proxmox.md) |
| [pve-almacenamiento.txt](virtualbox/pve-almacenamiento.txt), [pve-discos-fase4.txt](virtualbox/pve-discos-fase4.txt) | `pvesm status`, `lsblk`, `findmnt`, `pvs`, `vgs`, `lvs`, `storage.cfg` | Los discos, el LVM, los tres almacenamientos y `backup-lab` limitado a mi nodo | [Instalación](../docs/03-instalacion-proxmox.md) |
| [pve-mejoras.txt](virtualbox/pve-mejoras.txt) | `ls /sys/bus/virtio/drivers`, `lsblk`, `ip -d link`, `free -h` | Drivers virtio, modo promiscuo y la RAM que ve el nodo (3,8 GiB) | [Instalación](../docs/03-instalacion-proxmox.md) |
| [pve-upgrade.txt](virtualbox/pve-upgrade.txt), [pve-tras-upgrade.txt](virtualbox/pve-tras-upgrade.txt) | `apt full-upgrade` y comprobación después de reiniciar | Nodo actualizado a 9.2.32 y todo funcionando después del reinicio | [Instalación](../docs/03-instalacion-proxmox.md) |
| [pve-cuenta-lectura.txt](virtualbox/pve-cuenta-lectura.txt) | `pveum user list`, `acl list`, `user permissions`... | La cuenta `alumno-lectura@pve`, el rol `AulaLectura`, el permiso y su retirada | [Instalación](../docs/03-instalacion-proxmox.md) |
| Capturas `vbox_*.png` | VirtualBox | La red host-only, la Red NAT, la creación de la VM y su configuración | [Instalación](../docs/03-instalacion-proxmox.md) |
| Capturas `instalador_*.png` y `panel_proxmox.png` | Instalador y panel | La instalación y el primer acceso al panel | [Instalación](../docs/03-instalacion-proxmox.md) |
| Capturas `red_fase4.png`, `repos_despues.png`, `storage_fase4.png`, `discos_fase4.png`, `subir_iso_fase4.png` | Panel de Proxmox | Red, repositorios, almacenamientos, discos y la opción de subir una ISO | [Instalación](../docs/03-instalacion-proxmox.md) |
| Capturas `usuarios_fase4b.png`, `permisos_fase4.png`, `asignacion_permiso.png`, `acceso_*.png`, `firewall_fase4.png` | Panel y navegador en ventana privada | Cuentas, permisos y la prueba de acceso limitado (permitido y denegado con 403) | [Instalación](../docs/03-instalacion-proxmox.md) |

[Volver al índice](../README.md)
