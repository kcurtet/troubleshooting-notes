# Disco lleno en Linux

**Síntoma:** errores de "No space left on device", aplicaciones que no guardan, actualizaciones que fallan.

## Diagnóstico

```bash
df -h                                  # ¿qué partición está llena?
du -xh --max-depth=2 / 2>/dev/null | sort -rh | head -15   # ¿qué carpeta ocupa más?
df -i                                  # ¿se han acabado los inodos? (muchos archivos pequeños)
```

O directamente: [`espacio-disco.sh`](https://github.com/kcurtet/sysadmin-scripts) de mi repo de scripts.

## Causas habituales

| Dónde | Por qué |
| --- | --- |
| `/var/log`, journal | Logs que crecen sin límite |
| `/var/cache/pacman/pkg` | Arch guarda todas las versiones de los paquetes descargados |
| `~/.cache` | Cachés de navegador y aplicaciones |
| Imágenes de Podman/Docker | Imágenes y contenedores viejos que nadie borra |
| Papelera | Archivos "borrados" que siguen ocupando |

## Solución

```bash
sudo journalctl --vacuum-size=200M     # limita el journal a 200 MB
sudo paccache -r                       # Arch: deja solo las 3 últimas versiones
rm -rf ~/.cache/*                      # se regenera sola
podman system prune                    # contenedores parados e imágenes sin usar
```

**Ojo:** si `df` dice que está lleno pero `du` no encuentra nada, probablemente un proceso tiene abierto un archivo ya borrado. `sudo lsof +L1` lo muestra; reiniciar ese proceso libera el espacio.

## Prevención

Fijar `SystemMaxUse=500M` en `/etc/systemd/journald.conf` y activar `paccache.timer` para que se limpie solo.
