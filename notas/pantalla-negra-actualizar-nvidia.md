# Pantalla negra tras actualizar el driver NVIDIA

**Síntoma:** tras un `-Syu` que tocó `nvidia-utils` o el kernel, al arrancar la pantalla se queda negra tras GRUB, SDDM no aparece o la sesión se congela. En una TTY, `nvidia-smi` dice `Failed to initialize NVML: Driver/library version mismatch`.

## Diagnóstico

1. Entrar en una TTY (Ctrl+Alt+F3) y mirar qué cascó:

```bash
journalctl -b -p err --no-pager | tail -20
dmesg | grep -i NVRM                 # "NVRM: API mismatch" = módulo vs userspace
nvidia-smi
uname -r && pacman -Q | grep -E '^linux|nvidia'
dkms status                          # ¿compiló para el kernel que corre?
```

## Causas habituales

- `nvidia-utils` actualizado pero **sin reiniciar**: el módulo cargado en memoria sigue siendo el viejo. El mismatch de NVML sin reiniciar es exactamente esto.
- DKMS compiló para otro kernel: faltan los headers del kernel que arrancas (típico arrancando el LTS sin `linux-lts-headers`).
- Kernel nuevo desalineado con el paquete del driver precompilado (sin DKMS).
- KDE/Wayland necesita `nvidia-drm.modeset=1`; los drivers recientes lo traen por defecto, pero una vuelta atrás de versión puede romperlo.

## Solución

```bash
# mismatch tras actualizar sin reiniciar: basta reiniciar de verdad
sudo reboot

# DKMS sin compilar para el kernel actual: headers y recompilar
sudo pacman -S linux-headers          # el del kernel en uso (linux-lts-headers si es LTS)
sudo dkms install nvidia/575.57 -k $(uname -r)   # la versión que muestre dkms status
sudo reboot
```

Si no hay manera y hay que trabajar ya: desinstalar el driver y arrancar con nouveau o Wayland por software para no quedar bloqueado (ver también [no arranca tras actualizar el kernel](no-arranca-tras-actualizar-kernel.md)).

**Ojo:** no mezclar `nvidia` con `nvidia-dkms`, ni repos del sistema con AUR para el driver. Un solo origen o siempre habrá desalineaciones.

## Prevención

- Reiniciar siempre tras actualizar driver o kernel.
- Con DKMS: instalar los headers de cada kernel que se use.
- Mirar las noticias de Arch/CachyOS antes de actualizaciones grandes; los cambios de driver suelen avisar.
