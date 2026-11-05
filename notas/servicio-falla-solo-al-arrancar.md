# Un servicio falla al arrancar pero en manual funciona

**Síntoma:** `systemctl start mi-servicio` funciona perfecto con el sistema ya arrancado, pero tras un reinicio el servicio falla o intenta conectar antes de que la red esté lista.

## Diagnóstico

```bash
systemctl status mi-servicio
journalctl -u mi-servicio -b --no-pager | tail
systemd-analyze critical-chain mi-servicio.service   # qué esperó y qué no
systemctl list-dependencies mi-servicio.service --after
```

El patrón típico en el log: `connection refused` o DNS que no resuelve en los primeros segundos del boot.

## Causas habituales

- `After=network.target` **no** significa "la red está lista": solo que la configuración de red se ha aplicado. Hace falta `network-online.target`.
- El servicio `*-wait-online` no está habilitado (systemd-networkd-wait-online o NetworkManager-wait-online), así que `network-online.target` se cumple antes de tiempo.
- Carrera con otro servicio (base de datos, broker) que aún no escucha.

## Solución

```ini
# en la unit del servicio
[Unit]
After=network-online.target
Wants=network-online.target
```

```bash
sudo systemctl enable systemd-networkd-wait-online.service   # o NetworkManager-wait-online
sudo systemctl daemon-reload
sudo reboot        # validar en boot real, no con start
```

Para depender de otro servicio concreto, mejor `Requires=` + `After=` de ese servicio: más preciso que cualquier network-online.

**Ojo:** wait-online con una interfaz sin usar (Ethernet sin cable) puede colgar el boot un minuto o más. Deshabilitar esa interfaz o usar `systemd-networkd-wait-online --ignore=enoX`.

## Prevención

- Validar los servicios con un reinicio completo, no solo con `start`.
- Para servicios que reintentan solos, `Restart=on-failure` con `RestartSec=5` aguanta carreras menores sin tocar ordering.
