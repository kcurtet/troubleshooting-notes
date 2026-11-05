# "Ha iniciado sesión con un perfil temporal"

**Síntoma:** el usuario entra con su cuenta de siempre y Windows muestra "Ha iniciado sesión con un perfil temporal". El escritorio está vacío, sin sus ficheros ni su configuración, y todo lo que haga ahí se pierde al cerrar sesión.

## Diagnóstico

1. Confirmar que es perfil temporal: escribir `%USERPROFILE%` en Ejecutar; apunta a `C:\Users\TEMP`. El visor de eventos lo cuenta:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Application'; ProviderName='Microsoft-Windows-User Profile Service'} -MaxEvents 10
```

(Eventos 1511/1515: perfil no se cargó / ya se estaba cargando.)

2. Registro, donde vive el diagnóstico real: `regedit` → `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList`. Buscar el SID del usuario. Lo típico: dos entradas con el mismo SID, una normal y otra con sufijo `.bak`. Comprobar en cada una el valor `ProfileImagePath` y si su carpeta existe en `C:\Users`.

## Causas habituales

- `NTUSER.DAT` del perfil dañado por un apagón o cierre forzado durante la sesión (el síntoma al día siguiente de un corte es casi diagnóstico).
- Entrada duplicada en `ProfileList`: la buena quedó renombrada a `.bak` y Windows carga una huérfana.
- Carpeta del perfil renombrada o borrada a mano.

## Solución

Caso `.bak` (el clásico): en `ProfileList`, borrar la entrada del SID sin sufijo, quitarle el `.bak` a la buena, reiniciar. El usuario recupera su perfil entero.

Caso carpeta/NTUSER.DAT corrupto: crear un perfil nuevo. Entrar como admin, crear el usuario nuevo ( o apuntar `ProfileImagePath` a una carpeta nueva) y copiarle los datos del viejo: Documentos, Escritorio, favoritos. Nunca copiar `NTUSER.DAT`.

**Ojo:** tocar `ProfileList` mal deja la cuenta sin poder entrar: exportar la clave antes de cambiar nada. Y no borrar la carpeta del perfil viejo sin rescatar primero los datos del usuario.

## Prevención

- No apagar con el botón ni cortar la sesión a lo bruto.
- Vigilar la salud del disco: NTUSER.DAT corrupto suele ser primer síntoma.
