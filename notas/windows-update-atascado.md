# Windows Update atascado o fallando con error genérico

**Síntoma:** Windows Update se queda eternamente en "Buscando actualizaciones..." o en 0% de descarga, o falla con un código genérico (0x80070002, 0x8007000D, 0x8024402F). Reiniciar no arregla nada.

## Diagnóstico

1. Estado de los servicios implicados y log legible:

```powershell
Get-Service wuauserv,bits,cryptsvc
Get-WindowsUpdateLog                     # genera el log en el escritorio
```

2. Mirar la caché de descarga: `C:\Windows\SoftwareDistribution\Download` lleno de `.tmp` y descargas a medias = caché corrupta.

3. Descartar causas ajenas: hora del sistema desviada rompe TLS/WSUS (ver [la hora mal al alternar con Windows](hora-mal-doble-boot-windows.md)); en equipos de dominio, comprobar el WSUS configurado.

## Causas habituales

- Caché de descarga corrupta en `SoftwareDistribution` (lo más común tras un corte a mitad de descarga).
- BITS o el servicio de cifrado atascados.
- Almacén de componentes dañado (CBS): los arreglos de siempre (`sfc`, `DISM`) son el siguiente paso.

## Solución

Vaciar la caché con los servicios parados (renombrar, no borrar, para poder volver atrás):

```cmd
net stop wuauserv
net stop bits
net stop cryptsvc
ren C:\Windows\SoftwareDistribution SoftwareDistribution.old
ren C:\Windows\System32\catroot2 catroot2.old
net start cryptsvc
net start bits
net start wuauserv
```

Volver a "Buscar actualizaciones": la carpeta se regenera limpia. Si sigue fallando, reparar el almacén de componentes y luego reintentar:

```powershell
sfc /scannow
DISM /Online /Cleanup-Image /RestoreHealth
```

**Ojo:** no tocar `SoftwareDistribution` con el update en marcha ni borrar directamente; el renombrado permite restaurar si algo empeora.

## Prevención

- No forzar apagados con actualizaciones descargando o instalando.
- Mantener la hora del sistema correcta (rompe las conexiones TLS del servicio).
