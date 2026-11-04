# Wi-Fi conectado pero sin Internet

**Síntoma:** el equipo dice "Conectado" a la Wi-Fi, pero no carga ninguna web.

## Diagnóstico (de abajo arriba)

1. **¿Tiene IP válida?**
   - Linux: `ip -4 addr` · Windows: `ipconfig`
   - Si la IP empieza por `169.254.` → el DHCP no ha respondido. Ver [ip-169-254.md](ip-169-254.md).
2. **¿Llega al router?** `ping 192.168.1.1` (la puerta de enlace: `ip route` o `ipconfig`).
3. **¿Sale a Internet sin DNS?** `ping 1.1.1.1`
4. **¿Resuelve nombres?** `ping google.com`
   - Si el paso 3 funciona y el 4 no → es DNS. Ver [dns-no-resuelve.md](dns-no-resuelve.md).

## Causas habituales

| Falla en el paso | Causa probable |
| --- | --- |
| 1 | DHCP del router saturado o caído, filtro MAC, contraseña guardada mal |
| 2 | Interferencias, señal débil, aislamiento de clientes en la Wi-Fi de invitados |
| 3 | El router no tiene salida: fibra caída, ONT apagada, incidencia del operador |
| 4 | DNS del router o del operador caídos |

## Solución

- **Paso 3 falla para todos los equipos:** revisar luces del router/ONT (LOS en rojo = corte de fibra) y llamar al operador.
- **Solo falla un equipo:** olvidar la red y volver a conectar; renovar IP (`sudo dhclient -r && sudo dhclient` / `ipconfig /release` + `/renew`).
- **Señal débil o cortes:** en el router, cambiar de canal en 2,4 GHz (1, 6 u 11, los que no se solapan) o pasar el equipo a 5 GHz si está cerca.

## 2,4 GHz vs 5 GHz

| | 2,4 GHz | 5 GHz |
| --- | --- | --- |
| Alcance | Mayor, atraviesa mejor paredes | Menor |
| Velocidad | Menor | Mayor |
| Interferencias | Muchas (vecinos, microondas, Bluetooth) | Pocas |
