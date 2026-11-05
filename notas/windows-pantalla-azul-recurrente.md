# Pantallas azules (BSOD) recurrentes

**Síntoma:** pantallas azules aleatorias con códigos como `KERNEL_SECURITY_CHECK_FAILURE`, `IRQL_NOT_LESS_OR_EQUAL` o `PAGE_FAULT_IN_NONPAGED_AREA`. A veces con un patrón: siempre con un juego, siempre al despertar de la suspensión.

## Diagnóstico

1. Comprobar que se generan minidumps (Configuración avanzada del sistema → Inicio y recuperación → Volcado de memoria pequeño) y verlos:

```powershell
dir C:\Windows\Minidump
Get-WinEvent -LogName System -MaxEvents 20 | Where-Object Id -eq 1001 | Format-List Message
```

2. Analizar el dump con WinDbg Preview (Microsoft Store): abrir el `.dmp` y ejecutar `!analyze -v`. Alternativa rápida y gráfica: WhoCrashed o BlueScreenView. Lo importante es el **módulo** que causó el fallo: `nvlddmkm.sys` (driver NVIDIA), `ntfs.sys` (disco/sistema), `tcpip.sys` (red), etc.

3. Buscar el patrón: ¿empezó tras actualizar un driver? ¿con XMP activado? ¿al despertar de suspensión? ¿al conectar un periférico concreto?

## Causas habituales

- Driver (la gran mayoría: GPU, red, audio), a menudo en su versión más reciente.
- RAM defectuosa o con timings inestables (XMP).
- Disco con sectores dañados.
- Overclock inestable, archivo de página corrupto, software kernel (antivirus, VPN).

## Solución

Según el módulo del dump:

- Driver: actualizar, o revertir a la versión estable anterior. Para GPU, desinstalado limpio con Display Driver Uninstaller y reinstalar.
- RAM: Windows Memory Diagnostic (`mdsched.exe`) o MemTest86; para confirmar, desactivar XMP y ver si cesan.
- Disco: `chkdsk C: /f /r` y revisar SMART (ver [disco al 100%](windows-disco-al-100.md)).
- Sistema: `sfc /scannow` y `DISM /Online /Cleanup-Image /RestoreHealth`.

**Ojo:** el código del BSOD dice poco (el mismo código da RAM o driver); el módulo del minidump dice casi todo. Sin dump, se investiga a ciegas.

## Prevención

- Drivers del fabricante, no de mezcladores automáticos.
- Probar estabilidad (memtest) tras activar XMP o cambiar RAM.
- No overclock en máquinas de trabajo.
