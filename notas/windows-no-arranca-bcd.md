# Windows no arranca: reparar el arranque (BCD/EFI)

**Síntoma:** "Operating system not found" / "No bootable device", o bootloop en "Preparing Automatic Repair" que acaba en "Automatic Repair couldn't repair your PC", o `INACCESSIBLE_BOOT_DEVICE` tras una actualización. Es el equivalente Windows de [no arrancar tras actualizar el kernel](no-arranca-tras-actualizar-kernel.md).

## Diagnóstico

1. Arrancar desde un USB de instalación de Windows → Reparar el equipo → Solucionar problemas → Símbolo del sistema.

2. Ver qué queda del arranque:

```cmd
bcdedit /enum                    # ¿hay entradas? ¿apuntan a la partición buena?
diskpart
  list disk
  list vol                       # ¿existe la partición EFI (FAT32, ~100 MB)? ¿qué letra tiene C:?
  list part
dir C:\Windows                   # confirmar que C: es la partición de Windows
```

## Causas habituales

- BCD dañado por un corte durante una actualización.
- Se añadió o quitó un disco y el orden de arranque de la BIOS cambió (culpa de la BIOS, no de Windows: comprobarlo primero).
- Partición EFI dañada o sin formato tras un fallo.
- MBR dañado en equipos BIOS viejos.

## Solución

```cmd
bootrec /fixmbr
bootrec /fixboot
bootrec /scanos
bootrec /rebuildbcd
```

Si `bootrec /fixboot` da "Acceso denegado" (típico en UEFI con Windows 10/11), reconstruir la partición EFI a mano:

```cmd
diskpart
  list vol                       # identificar la FAT32 de ~100 MB
  sel vol X
  assign letter=Z
exit
format Z: /FS:FAT32              # ¡BORRA lo que hay en la EFI!
bcdboot C:\Windows /s Z: /f UEFI
```

**Ojo:** con doble boot Linux/Windows, GRUB vive en esa misma partición EFI. No formatearla sin rescatar: regenerar después GRUB desde el Linux (con `grub-install`) o usar el `bcdboot` sin formatear si la EFI está sana.

## Prevención

- Tras añadir o quitar discos, revisar el orden de arranque en la BIOS antes de culpar al sistema.
- Tener siempre a mano un USB de instalación de Windows.
- Restaurar sistema activado: a veces salva más rápido que la consola.
