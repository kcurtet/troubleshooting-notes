# Hay Internet pero no se resuelven nombres (DNS)

**Síntoma:** `ping 1.1.1.1` funciona, pero `ping google.com` da "Name or service not known" / "No se pudo encontrar el host".

## Diagnóstico

```bash
# ¿Qué servidores DNS usa el equipo?
cat /etc/resolv.conf          # Linux
resolvectl status             # Linux con systemd-resolved
ipconfig /all                 # Windows

# Preguntar a un DNS concreto, saltándose el configurado
dig google.com @1.1.1.1       # o: nslookup google.com 1.1.1.1
```

- Si responde preguntando a `1.1.1.1` pero no con el DNS configurado → el DNS configurado está caído o mal puesto.

## Causas habituales

- El DNS del router o del operador ha caído.
- DNS puesto a mano con una IP que ya no existe.
- VPN que cambia el DNS y no lo restaura al desconectarse.
- Caché DNS con una respuesta antigua.

## Solución

1. Vaciar caché: `resolvectl flush-caches` (Linux) · `ipconfig /flushdns` (Windows).
2. Probar con un DNS público (1.1.1.1 o 8.8.8.8) en la configuración de red del equipo.
3. Si funciona, el problema es del router o del operador: cambiar el DNS en el DHCP del router para toda la red.

## Prevención

Documentar qué DNS usa cada red. Si la empresa tiene DNS interno (Active Directory), **no** poner DNS públicos en los equipos del dominio: dejarían de encontrar el controlador de dominio.
