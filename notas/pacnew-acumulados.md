# Configuraciones .pacnew acumuladas (pacman nunca sobrescribe /etc)

**Síntoma:** meses después de una actualización grande, un servicio no arranca o se comporta raro y su config no cuadra con la versión instalada. O simplemente: `find /etc -name '*.pacnew'` devuelve una lista que nadie había mirado.

## Diagnóstico

```bash
sudo find /etc -name '*.pacnew' -o -name '*.pacsave' | sort
grep 'installed as' /var/log/pacman.log | tail -20
pacman -Q pacman-contrib                # trae pacdiff
```

## Causas habituales

- Al actualizar, pacman instala la config nueva como `/etc/algo.pacnew` y nunca sobrescribe la vigente. Si se ignoran, la config local diverge del formato que la versión nueva del programa espera: claves renombradas, secciones que ya no existen, valores por defecto distintos.
- `*.pacsave` es lo inverso: la config conservada al desinstalar un paquete.

## Solución

Revisar y mergear cada fichero con `pacdiff`:

```bash
sudo pacdiff                     # interactivo, uno a uno
sudo DIFFPROG=vimdiff pacdiff    # con merge real de dos/tres paneles
```

Para cada fichero: quedarse lo nuevo, lo local, o mezclar. Borrar el `.pacnew` sin mirarlo es quedarse con una config vieja para un binario nuevo: justo el fallo que explicaba el síntoma.

**Ojo:** justo tras el `-Syu` la divergencia es mínima y mergear cuesta un minuto; años después es doloroso y propenso a errores.

## Prevención

- Leer los avisos de pacman: `warning: /etc/... installed as /etc/....pacnew` es la señal.
- Rutina fija tras actualizar: `pacdiff` en el mismo momento, no "algún día".
