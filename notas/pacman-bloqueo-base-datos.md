# Pacman se queda con la base de datos bloqueada (db.lck)

**Síntoma:** tras cortar una actualización a mitad (Ctrl+C, cuelgue, corte de luz, disco lleno), cualquier `pacman` falla con:

```text
error: could not lock database: File exists
if you're sure a package manager is not already running, you can remove /var/lib/pacman/db.lck
```

(en locale español: *no se pudo bloquear la base de datos: El fichero existe*).

## Diagnóstico

1. Confirmar que existe el lock y que **no hay ningún gestor de paquetes vivo** (pacman, paru, yay comparten el mismo lock):

```bash
ls -l /var/lib/pacman/db.lck
pgrep -a pacman || echo "sin pacman corriendo"
sudo fuser -v /var/lib/pacman/db.lck   # vacío = nadie lo tiene
```

2. Mirar hasta dónde llegó la transacción cortada:

```bash
tail -30 /var/log/pacman.log
grep ALPM /var/log/pacman.log | tail -15
```

## Causas habituales

- Una transacción anterior terminó a lo bruto (kill, OOM, reinicio a medias): pacman crea `db.lck` al empezar y lo borra al salir limpio, pero no puede limpiarlo si lo matan.
- Otro gestor sigue corriendo de verdad y el lock es legítimo (por eso se comprueba primero).
- La actualización se cortó por otra razón (p. ej. disco lleno, ver [disco lleno](disco-lleno-linux.md) y [btrfs sin espacio](btrfs-sin-espacio-balance.md)) y el lock es solo el síntoma residual.

## Solución

Si `fuser` no devolvió nada, el lock es un residuo: borrarlo y relanzar.

```bash
sudo rm /var/lib/pacman/db.lck
sudo pacman -Syu
```

**Ojo:** no borrar el lock jamás con un pacman corriendo de verdad; corromperías la base de datos.

Pacman rehace la transacción incompleta: los paquetes que no llegó a registrar se instalan enteros. Si sospechas de un paquete a medias registrado, verificarlo y reinstalarlo:

```bash
pacman -Qk 2>/dev/null | grep -v "0 archivos perdidos"   # lista paquetes con ficheros ausentes
sudo pacman -S <paquete>
```

## Prevención

- No interrumpir una transacción en curso; si hay que parar, dejarla acabar.
- Un solo gestor de paquetes a la vez (pacman y helpers AUR usan el mismo lock).
- Antes de un `pacman -Syu` grande: espacio libre suficiente y batería/enchufe.
