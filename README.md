# troubleshooting-notes

Mis notas de resolución de incidencias: cada nota es un problema real o practicado, con cómo diagnosticarlo, la causa y la solución. Las escribo para no tener que investigar dos veces lo mismo.

> *My IT troubleshooting notes: symptom → diagnosis → cause → fix, for networking, Linux and Windows issues.*

## Índice

### Redes
- [Wi-Fi conectado pero sin Internet](notas/wifi-conectado-sin-internet.md)
- [Hay Internet pero no se resuelven nombres (DNS)](notas/dns-no-resuelve.md)
- [El equipo tiene una IP 169.254.x.x](notas/ip-169-254.md)

### Linux
- [Disco lleno](notas/disco-lleno-linux.md)
- [Btrfs sin espacio: balance con un USB temporal](notas/btrfs-sin-espacio-balance.md)

### Windows
- [La impresora no imprime y la cola está atascada](notas/windows-impresora-cola-atascada.md)

## Cómo está escrita cada nota

Todas siguen la misma [plantilla](PLANTILLA.md):

1. **Síntoma**: lo que dice o ve el usuario.
2. **Diagnóstico**: comandos y comprobaciones, en orden.
3. **Causas habituales**.
4. **Solución**.
5. **Prevención**: qué hacer para que no vuelva a pasar.

Scripts relacionados en [sysadmin-scripts](https://github.com/kcurtet/sysadmin-scripts).
