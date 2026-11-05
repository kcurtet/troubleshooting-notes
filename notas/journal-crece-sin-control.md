# El journal de systemd se come el disco

**Síntoma:** `/var/log/journal` ocupa gigas y el espacio libre en `/` baja sin explicación; puede desembocar en un [disco lleno](disco-lleno-linux.md).

## Diagnóstico

1. Tamaño real y quién escribe:

```bash
journalctl --disk-usage
du -sh /var/log/journal
journalctl -b --no-pager | awk '{print $5}' | sort | uniq -c | sort -rn | head   # top de unidades
```

## Causas habituales

- Un servicio roto que loguea cada segundo: el tamaño es el síntoma, el spammer es la causa.
- journald persistente sin límite ajustado: por defecto se permite hasta el 10% del filesystem (tope 4 GiB), que en discos grandes son 4 GiB de journal.
- Modo debug dejado activado en algún servicio o unidad.

## Solución

Recorte inmediato y límite permanente:

```bash
sudo journalctl --vacuum-size=500M --vacuum-time=2week

# /etc/systemd/journald.conf
#   SystemMaxUse=500M
#   MaxRetentionSec=2week
sudo systemctl restart systemd-journald
```

Y arreglar el servicio que spammea; si no, el journal vuelve a crecer al mismo ritmo.

## Prevención

- Fijar `SystemMaxUse` en la instalación del servidor, no cuando ya duele.
- Ante una caída de espacio libre, `journalctl --disk-usage` es de las tres primeras comprobaciones.
