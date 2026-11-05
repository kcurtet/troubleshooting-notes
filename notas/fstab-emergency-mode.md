# fstab con una entrada muerta: emergency mode al arrancar

**Síntoma:** al arrancar, systemd se para en "You are in emergency mode" / "Give root password for maintenance", o un trabajo de montaje aguanta el boot 90 segundos y sigue.

## Diagnóstico

1. Desde la shell de emergencia:

```bash
systemctl list-jobs                        # qué job de montaje está corriendo/colgado
journalctl -xb | grep -iE 'timed out|mount|fsck' | tail
lsblk -f                                   # UUIDs reales de los discos presentes
grep -v '^#' /etc/fstab                    # qué espera fstab que exista
```

2. Cruzar: hay alguna línea de fstab cuyo UUID ya no aparece en `lsblk -f`. Esa es la culpable.

## Causas habituales

- Se retiró o formateó un disco/USB que estaba montado permanente: su UUID ya no existe, systemd espera el device hasta el timeout y cae en emergency.
- fstab editado a mano con un error (UUID mal copiado, tipo de filesystem mal escrito).
- fsck fallando en el arranque (disco dañado o desconexión sucia).

## Solución

En la shell de emergencia:

```bash
mount -o remount,rw /
nano /etc/fstab            # comentar o arreglar la línea culpable
systemctl daemon-reload
exit                       # continuar el arranque
```

Para discos extraíbles que sí deban estar en fstab, opciones seguras:

```fstab
UUID=xxxx  /mnt/datos  ext4  nofail,x-systemd.automount,x-systemd.device-timeout=10s  0 2
```

**Ojo:** `nofail` a secas aún bloquea a quien lea ese punto de montaje antes de que aparezca; la pareja con `x-systemd.automount` es lo cómodo.

## Prevención

- Tras editar fstab, validar siempre: `findmnt --verify`.
- Nada extraíble en fstab sin `nofail`; para USB, mejor units de montaje de systemd.
