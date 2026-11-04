# Windows: la impresora no imprime y la cola está atascada

**Síntoma:** los documentos se quedan en la cola "Imprimiendo" o "Error" y no sale nada, aunque la impresora está encendida.

## Diagnóstico

1. ¿La impresora imprime una **página de prueba desde su propio panel**? Si no → es la impresora (papel, tóner, atasco), no Windows.
2. ¿Imprimen otros equipos? Si nadie puede → impresora o red. Si solo falla uno → ese equipo.
3. Impresora de red: `ping <IP de la impresora>`. Si no responde, ha cambiado de IP o está fuera de la red.

## Solución: vaciar la cola a mano

Desde una terminal **como administrador**:

```bat
net stop spooler
del /Q /F %systemroot%\System32\spool\PRINTERS\*
net start spooler
```

Esto para el servicio de cola de impresión, borra los trabajos atascados y lo vuelve a arrancar.

## Otras causas habituales

- La impresora de red ha cambiado de IP por DHCP → **reservar su IP en el router** para que no cambie.
- Driver corrupto tras una actualización → quitar la impresora, desinstalar el driver y añadirla de nuevo.
- Impresora predeterminada equivocada (Windows a veces cambia a la última usada: *Configuración → Impresoras → "Permitir que Windows administre la impresora predeterminada"*).
