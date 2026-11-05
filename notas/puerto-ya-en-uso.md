# "Address already in use": el puerto ya está cogido

**Síntoma:** un servicio no arranca y el log dice `Address already in use` (`EADDRINUSE`), o el servidor de desarrollo falla al hacer bind en su puerto.

## Diagnóstico

1. Ver quién escucha en ese puerto:

```bash
sudo ss -tlnp | grep :8000           # -t tcp, -l listening, -n sin resolver, -p proceso
sudo fuser -v 8000/tcp               # alternativa directa
sudo lsof -i :8000                   # la más verbosa
```

2. Si nadie aparece en LISTEN, mirar otros estados:

```bash
sudo ss -tan | grep :8000            # TIME_WAIT / CLOSE_WAIT masivos
systemctl status mi-servicio.socket  # socket activation de systemd
```

## Causas habituales

- Una instancia anterior sigue viva: se lanzó a mano con `&` y systemd (o tú) arranca otra encima.
- Systemd socket activation: el puerto lo retiene la unidad `.socket`, no el servicio.
- TIME_WAIT tras reiniciar rápido el servicio en desarrollo: el kernel retiene el puerto ~60 s.
- Otro servicio distinto con el mismo puerto (5000 y 8080 son clásicos de todo).

## Solución

```bash
# proceso suelto (no gestionado por systemd)
sudo fuser -k 8000/tcp

# si es de systemd, pararlo por su unidad
systemctl stop mi-servicio

# socket activation: hay que parar el socket
systemctl stop mi-servicio.socket
```

Si es TIME_WAIT en desarrollo: esperar un minuto o activar SO_REUSEADDR en la aplicación.

**Ojo:** `fuser -k` mata sin preguntar. Verificar antes con `ss -tlnp` que el proceso retenedor no es otro servicio importante.

## Prevención

- Gestionar los servicios con systemd, no lanzarlos a mano con `&`.
- Antes de fijar un puerto en una config nueva, comprobar con `ss -tlnp` que está libre.
