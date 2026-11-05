# No arranca tras actualizar el kernel

**Síntoma:** tras un `pacman -Syu` que actualizó el kernel y reiniciar, GRUB da "file not found", hay kernel panic a los segundos, o cae en una shell de emergencia del initramfs ("You are now being dropped into an emergency shell").

## Diagnóstico

1. En el menú de GRUB, entrar en "Advanced options" y probar el kernel anterior. Si arranca, el problema es el kernel/initramfs nuevo, no el disco ni GRUB.

2. Con el kernel viejo (o desde un USB de instalación), ver qué pasó:

```bash
grep -E 'linux|mkinitcpio|dracut' /var/log/pacman.log | tail -20
ls -lh /boot                          # ¿existen vmlinuz-* e initramfs-* del kernel nuevo?
journalctl -b -1 -p err --no-pager | tail -20
```

3. Con módulos externos (NVIDIA, VirtualBox, ZFS), comprobar para qué kernels compiló DKMS:

```bash
dkms status
```

## Causas habituales

- Transacción cortada a medias: kernel instalado pero initramfs no generado (ver [pacman con la base de datos bloqueada](pacman-bloqueo-base-datos.md)).
- El hook de regeneración falló, típico por falta de espacio justo al escribir el initramfs (ver [disco lleno](disco-lleno-linux.md)).
- DKMS no compiló los módulos para el kernel nuevo (faltan los headers).
- `grub.cfg` apunta a un kernel que ya no está en `/boot`.

## Solución

Arrancar con el kernel anterior o desde USB de instalación y, dentro del sistema (desde USB: montar y `arch-chroot`):

```bash
# desde el USB de instalación
mount /dev/nvme0n1p2 /mnt            # la partición raíz
arch-chroot /mnt

# reinstalar el kernel limpio y regenerar todo
pacman -S linux linux-headers
mkinitcpio -P                        # en distros con dracut: dracut --regenerate-all --force
grub-mkconfig -o /boot/grub/grub.cfg
exit && reboot
```

**Ojo:** tener siempre un segundo kernel (`linux-lts`) instalado y su entrada en GRUB. Es el salvavidas que evita tirar de USB.

## Prevención

- Mantener `linux-lts` como plan B; no desinstalar el kernel anterior justo tras actualizar.
- Comprobar espacio en `/` y `/boot` antes de actualizaciones grandes.
- Leer la salida del `-Syu`: si el hook del initramfs da un warning, no reiniciar sin resolverlo.
