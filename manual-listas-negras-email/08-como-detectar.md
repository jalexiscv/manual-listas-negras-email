# Capítulo 8: Cómo Detectar si Estás en una Lista Negra

[← Anterior](07-motivos-de-inclusion.md) | [Índice](00-indice.md) | [Siguiente →](09-spf-dkim-dmarc.md)

---

## 8.1 Herramientas Web

| Herramienta | URL |
|-------------|-----|
| **MXToolbox** | https://mxtoolbox.com/blacklists.aspx |
| **Spamhaus Check** | https://check.spamhaus.org/ |
| **Barracuda Center** | https://www.barracudacentral.org/lookup |
| **MultiBL** | https://multibl.org/ |
| **WhatIsMyIPAddress** | https://whatismyipaddress.com/blacklist-check |

## 8.2 Verificación Manual con Dig

```bash
IP="203.0.113.50"
REV_IP=$(echo $IP | awk -F. '{print $4"."$3"."$2"."$1}')

for LIST in zen.spamhaus.org bl.spamcop.net b.barracudacentral.org; do
    RESULT=$(dig +short "$REV_IP.$LIST" A)
    if [ -n "$RESULT" ]; then
        echo "LISTADO en $LIST (codigo: $RESULT)"
    else
        echo "Limpio en $LIST"
    fi
done
```

## 8.3 Desde el Servidor de Correo

### Postfix
```bash
grep 'rejected.*dnsbl\|rejected.*spamhaus' /var/log/mail.log
grep '203.0.113.50' /var/log/mail.log | grep 'rejected'
```

### Exim
```bash
grep 'rejected by DNSBL' /var/log/exim/mainlog
```

## 8.4 Verificación de Entregabilidad

| Servicio | URL | Función |
|----------|-----|---------|
| Google Postmaster | https://postmaster.google.com/ | Reputación en Gmail |
| Microsoft SNDS | https://snds.microsoft.com/ | Reputación en Outlook |
| Yahoo Sender Hub | https://senders.yahooinc.com/ | Feedback Yahoo |
| Mail-Tester | https://www.mail-tester.com/ | Puntaje general |
| GlockApps | https://glockapps.com/ | Multi-buzón |

## 8.5 Monitoreo Automatizado

```bash
#!/bin/bash
# blacklist-monitor.sh — Ejecutar cada 30 min via cron

IP_LIST=("203.0.113.50")
ALERT_EMAIL="admin@ejemplo.com"
LISTS=(zen.spamhaus.org bl.spamcop.net b.barracudacentral.org)

for IP in "${IP_LIST[@]}"; do
    REV_IP=$(echo $IP | awk -F. '{print $4"."$3"."$2"."$1}')
    for LIST in "${LISTS[@]}"; do
        RESULT=$(dig +short "$REV_IP.$LIST" A)
        if [ -n "$RESULT" ]; then
            echo "ALERTA: IP $IP listada en $LIST" | mail -s "ALERTA" "$ALERT_EMAIL"
        fi
    done
done
```

> 📌 Cuanto antes sepas que estás listado, antes puedes actuar. Una IP listada durante horas daña significativamente la reputación.
