# "No space left on device" con df mostrando espacio libre

**Síntoma:** crear cualquier fichero nuevo falla con `No space left on device`, pero `df -h` muestra gigas libres. Los ficheros existentes se pueden seguir escribiendo.

## Diagnóstico

1. Comparar espacio con inodos:

```bash
df -h                                 # espacio: hay libre
df -i                                 # IUse% al 100% en el mismo filesystem: ese es el problema
```

2. Localizar la acumulación de ficheros:

```bash
sudo du --inodes -x / 2>/dev/null | sort -rn | head -15
```

## Causas habituales

- Millones de ficheros pequeños acumulados: sesiones PHP viejas (`/var/lib/php/sessions`), temporales de una app que no limpia, maildir gigante, cachés rotas.
- Un servicio que crea un fichero por segundo (logs sin rotar, métricas).
- Ficheros borrados pero retenidos abiertos por un proceso: el inodo no se libera hasta cerrarlo, y `df` no cuadra con `du`.

## Solución

```bash
# ejemplo: sesiones con más de una semana
sudo find /var/lib/php/sessions -type f -mtime +7 -delete

# si df -i no cuadra con du --inodes: borrados retenidos abiertos
sudo lsof +L1                         # ficheros sin link pero abiertos
sudo systemctl restart <servicio>     # reiniciar quien los retiene
```

**Ojo:** antes del `-delete`, lanzar el mismo `find` con `| wc -l` y revisar el directorio. Borrar a ciegas con inodos agotados es fácil y duele.

## Prevención

- Mirar `df -i` junto a `df -h`; es la mitad del diagnóstico que siempre se olvida.
- Montar `/tmp` en tmpfs: se vacía en cada reinicio.
- Timers de limpieza para lo que se sepa que acumula (systemd-tmpfiles o un timer).
