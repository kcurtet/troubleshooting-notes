# Btrfs sin espacio: balance con un USB temporal

**Síntoma:** a mitad de un `pacman -Syu` todo empieza a fallar con "No queda espacio en el dispositivo". Al reiniciar fallan `ldconfig`, `systemd-sysusers` y otros servicios porque no pueden escribir en `/etc`, y el gestor de login no llega a arrancar.

## Diagnóstico

1. Confirmar en el journal que el fallo es por espacio:

```bash
journalctl -b -u ldconfig.service -u systemd-sysusers.service --no-pager | tail
grep ALPM /var/log/pacman.log | tail -15     # ¿se cortó una actualización?
```

2. En btrfs, `df` no basta: puede mostrar espacio libre y aun así dar ENOSPC. Hay que mirar cuánto queda **sin asignar**:

```bash
sudo btrfs filesystem usage /
```

Lo que importa:
- **`Device unallocated`**: si está cerca de 0, btrfs no puede crear chunks nuevos.
- **`Data` / `Metadata`**: comparar `Size` (asignado) con `Used`. Muchos chunks de datos medio vacíos ocupan espacio asignado que no se puede usar para metadatos.

## Causa

Btrfs reparte el disco en chunks de datos y de metadatos. Cuando todo el disco está asignado a chunks, aunque estén medio vacíos, no se pueden crear chunks nuevos de metadatos y cualquier escritura falla.

El arreglo normal es un `btrfs balance`, que compacta los chunks medio vacíos y devuelve espacio sin asignar. Pero **el balance también necesita espacio libre para trabajar**, así que con el disco lleno del todo no arranca.

## Solución

Añadir temporalmente otro dispositivo al filesystem (un USB vale) para que el balance tenga sitio, compactar y luego retirarlo.

```bash
# 1. Añadir el USB al filesystem (BORRA lo que haya en el USB)
sudo btrfs device add -f /dev/sdX1 /

# 2. Balance solo de chunks poco usados (rápido); subir el porcentaje si hace falta
sudo btrfs balance start -dusage=10 /
sudo btrfs balance start -dusage=50 /

# 3. Retirar el USB: btrfs mueve sus datos de vuelta al disco principal
sudo btrfs device remove /dev/sdX1 /
```

**Ojo:** el USB no se puede desconectar sin más. Mientras forma parte del filesystem tiene datos (en mi caso 2 GiB). Hay que hacer siempre `btrfs device remove` y esperar a que termine.

Comprobar que se retiró bien:

```bash
sudo btrfs filesystem show /      # debe decir "Total devices 1"
sudo btrfs device stats /         # todo a 0
sudo dmesg | grep -i btrfs | tail # "relocating block group..." y "device deleted: /dev/sdX1"
```

Después, si se cortó una actualización por falta de espacio, hay que terminarla. Si no, el sistema queda a medias (en mi caso `glibc` actualizado y `lib32-glibc` no):

```bash
sudo pacman -Syu
sudo ldconfig
```

## Prevención

- Mirar `btrfs filesystem usage /` de vez en cuando, no solo `df`.
- Hacer un balance ligero periódico (`btrfs balance start -dusage=50 /`) o activar el timer de `btrfsmaintenance` para que no se acumulen chunks medio vacíos.
- No lanzar un `pacman -Syu` grande con poco espacio sin asignar.
