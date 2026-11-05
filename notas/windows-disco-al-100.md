# El disco está al 100% y todo va lentísimo

**Síntoma:** el equipo tarda minutos en abrir cualquier cosa; el Administrador de tareas muestra el disco al 100% de forma constante, sobre todo en portátiles con HDD.

## Diagnóstico

1. Administrador de tareas → Rendimiento → Disco: ¿100%? ¿Con cuántos MB/s reales? Luego Procesos ordenados por columna Disco. Si no queda claro, Resource Monitor (`resmon`) → pestaña Disco muestra los ficheros concretos.

2. Culpables habituales que aparecerán: `MsMpEng.exe` (Defender analizando), `SearchIndexer.exe` (indexado), `SysMain`/Superfetch, Windows Update, OneDrive sincronizando.

3. Salud del disco, que es lo que de verdad decide el diagnóstico:

```powershell
Get-PhysicalDisk | Get-StorageReliabilityCounter | Select-Object Wear,ReadErrorsTotal,Temperature
wmic diskdrive get model,status            # o CrystalDiskInfo para verlo todo
```

## Causas habituales

- HDD lento con varios servicios comiéndolo a la vez (Defender + indexado + update): es ruido, no enfermedad.
- Disco muriéndose: sectores reasignados hacen que cada lectura tarde y la actividad se dispare sin transferir nada.
- Minero/malware de fondo.
- Un proceso congelado haciendo I/O constante (logs en bucle).

## Solución

Según el culpable identificado: programar los escaneos de Defender, limitar el indexado de Windows Search a las carpetas útiles, desactivar SysMain (`services.msc`) en HDD, o pausar OneDrive.

Si SMART muestra errores o sectores reasignados: **reemplazo del disco, prioridad absoluta**, y copia de datos antes de nada.

**Ojo:** 100% de actividad con pocos MB/s transferidos y tiempos de respuesta altos (resmon los muestra en ms) = disco enfermo. Los procesos que aparecen consumiendo disco son víctimas, no causantes; cambiarlos de nada.

## Prevención

- SSD como solución definitiva para equipos con HDD.
- Vigilar SMART de vez en cuando (CrystalDiskInfo con alertas).
- Excluir del indexado lo que no se busca.
