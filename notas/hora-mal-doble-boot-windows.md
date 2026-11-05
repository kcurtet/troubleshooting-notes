# La hora va mal al alternar con Windows (doble boot)

**Síntoma:** al entrar en Linux tras usar Windows (o al revés), la hora está desviada 1-2 horas. Síntomas derivados engañosos: HTTPS falla con "certificado no válido aún/expirado", no se puede iniciar sesión en servicios web, errores GPG o TLS en repositorios, ficheros con fechas futuras.

## Diagnóstico

```bash
timedatectl
# La línea clave: "RTC in local TZ: yes"
# → Linux interpreta el reloj hardware (RTC) como hora local, pero espera UTC
date && date -u
timedatectl show-timesync | head          # ¿NTP sincronizando?
```

## Causas habituales

- Windows escribe el RTC en hora local; Linux por defecto lo interpreta como UTC. Cada arranque "corrige" al otro y la hora salta según la zona horaria.
- Sin NTP activo, el desfase no se autocorrige nunca.

## Solución

Elegir una sola convención. Recomendado: pasar Windows a UTC (una vez, como admin):

```powershell
reg add "HKLM\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /t REG_QWORD /d 1 /f
```

Alternativa (Linux a hora local; menos limpia por el horario de verano):

```bash
timedatectl set-local-rtc 1 --adjust-system-clock
```

Después, NTP activo y comprobación:

```bash
sudo timedatectl set-ntp true
timedatectl                             # "System clock synchronized: yes"
```

**Ojo:** systemd avisa de que el RTC en local "no está soportado" y puede desajustarse con los cambios de hora; por eso es preferible tocar el lado de Windows.

## Prevención

- Fijar la convención una vez y activar NTP en ambos sistemas.
- Ante errores raros de certificados TLS, mirar el reloj antes que nada lo demás.
