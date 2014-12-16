# Capítulo 8: Cómo Detectar si Estás en una Lista Negra

[← Anterior](07-motivos-de-inclusion.md) | [Índice](README.md) | [Siguiente →](09-spf-dkim-dmarc.md)

---

## 8.1 Herramientas Web de Verificación

La forma más rápida de saber si tu IP o dominio está en una lista negra es utilizar herramientas web especializadas que consultan múltiples listas simultáneamente y presentan los resultados en una interfaz clara. Estas herramientas son el punto de partida para cualquier diagnóstico de entregabilidad.

### MXToolbox — La Herramienta Más Completa

**URL:** https://mxtoolbox.com/blacklists.aspx

MXToolbox es probablemente la herramienta de verificación más conocida y utilizada por administradores de sistemas en todo el mundo. Te permite ingresar una dirección IP o nombre de dominio y verifica su presencia en más de 100 listas negras diferentes, presentando los resultados codificados por colores: verde para "no listado", rojo para "listado", y amarillo para "advertencia".

Una de las características más valiosas de MXToolbox es que no solo te dice si estás listado, sino que también proporciona enlaces directos a la página de información de cada lista y, cuando está disponible, al formulario de desliste correspondiente. Esto acelera significativamente el proceso de resolución.

**Cómo usar MXToolbox efectivamente:**
1. Ingresa tu dirección IP en el campo de búsqueda
2. Selecciona "Blacklist Check" en el menú desplegable
3. Haz clic en "Check" y espera 10-30 segundos mientras consulta todas las listas
4. Revisa los resultados: cualquier lista marcada en rojo requiere atención
5. Haz clic en el enlace de cada lista listada para ir directamente a su página de información

```bash
# MXToolbox también ofrece una API limitada para consultas programáticas
$ curl -s "https://mxtoolbox.com/api/v1/lookup/blacklist/203.0.113.50"
```

### Spamhaus Check — El Verificador Oficial

**URL:** https://check.spamhaus.org/

El verificador oficial de Spamhaus es la herramienta más precisa para determinar tu estado en las listas de Spamhaus (SBL, XBL, PBL, DBL). A diferencia de MXToolbox, que consulta muchas listas, Spamhaus Check se enfoca exclusivamente en las listas de Spamhaus pero proporciona información mucho más detallada.

Cuando ingresas tu IP, la herramienta muestra:
- El código de retorno exacto (127.0.0.x)
- La lista específica de Spamhaus donde estás incluido
- Una explicación detallada del motivo del listado
- Ejemplos de los correos que activaron la detección (en algunos casos)
- Enlaces a la documentación relevante
- El formulario de solicitud de desliste si está disponible

### Otras Herramientas Web Útiles

**Barracuda Center (https://www.barracudacentral.org/lookup):** Verificador oficial de Barracuda. Simple, rápido y directo. Solo te dice si estás o no en BRBL.

**MultiBL (https://multibl.org/):** Similar a MXToolbox pero con un enfoque más técnico y detallado. Muestra las IPs de retorno exactas y proporciona explicaciones para cada lista.

**WhatIsMyIPAddress (https://whatismyipaddress.com/blacklist-check):** Herramienta popular y fácil de usar, ideal para verificaciones rápidas. No es tan completa como MXToolbox pero es suficiente para la mayoría de los casos.

**BlacklistAlert (https://www.blacklistalert.org/):** Ofrece verificación y notificaciones por email cuando detecta cambios en el estado de tu IP en las listas negras.

## 8.2 Verificación Manual con Dig y Nslookup

Para los administradores que prefieren la línea de comandos o necesitan integrar la verificación en scripts automatizados, las consultas manuales con `dig` ofrecen el control más preciso.

### Script de Verificación contra Múltiples Listas

```bash
#!/bin/bash
# Verificacion manual contra las listas mas importantes

IP="203.0.113.50"
REV_IP=$(echo $IP | awk -F. '{print $4"."$3"."$2"."$1}')

echo "Verificando IP: $IP"
echo "========================"

# Listas a verificar
LISTS=(
    "zen.spamhaus.org"
    "bl.spamcop.net"
    "b.barracudacentral.org"
    "psbl.surriel.com"
    "dnsbl-1.uceprotect.net"
    "dnsbl.sorbs.net"
)

for LIST in "${LISTS[@]}"; do
    RESULT=$(dig +short "$REV_IP.$LIST" A 2>/dev/null)
    if [ -n "$RESULT" ]; then
        echo "LISTADO en $LIST (codigo: $RESULT)"
    else
        echo "Limpio en $LIST"
    fi
done
```

### Interpretación de Resultados

Cuando ejecutas una consulta DNSBL, el resultado puede ser:

- **Una dirección IP en el rango 127.0.0.0/8:** La IP está listada. El valor específico indica la categoría (por ejemplo, 127.0.0.2 = spam directo en Spamhaus).
- **NXDOMAIN (sin respuesta):** La IP no está listada en esa lista en este momento.
- **SERVFAIL:** El servidor DNS de la lista negra está teniendo problemas. En este caso, la mayoría de los servidores de correo tratan el resultado como "no listado" para evitar falsos positivos por problemas técnicos de la lista.
- **Timeout:** La consulta no recibió respuesta. Similar a SERVFAIL, se trata como "no listado".

## 8.3 Verificación Desde los Logs del Servidor de Correo

Los logs del servidor de correo son la fuente más fiable de información sobre rechazos por listas negras. Mientras que las herramientas web te dicen si estás listado, los logs te dicen quién te está rechazando por estar listado.

### En Postfix

```bash
# Buscar todos los rechazos por DNSBL
grep 'rejected' /var/log/mail.log | grep -i 'dnsbl\|spamhaus\|zen\|spamcop\|barracuda'

# Buscar rechazos contra una lista especifica
grep 'zen.spamhaus.org' /var/log/mail.log

# Ver los mensajes de error completos de los ultimos rechazos
grep 'rejected' /var/log/mail.log | tail -20

# Buscar por IP de origen especifica
grep '203.0.113.50' /var/log/mail.log | grep 'rejected'
```

**Ejemplo de línea de log con rechazo por DNSBL:**
```
postfix/smtpd[12345]: NOQUEUE: reject: RCPT from unknown[203.0.113.50]:
550 5.7.1 Service unavailable; Client host [203.0.113.50] blocked using
zen.spamhaus.org; from=<bot@ejemplo.com> to=<user@destino.com>
proto=ESMTP helo=<mail.ejemplo.com>
```

Esta línea contiene toda la información que necesitas: la IP rechazada (`203.0.113.50`), la lista que la bloqueó (`zen.spamhaus.org`), y los detalles del remitente y destinatario.

### En Exim

```bash
# Buscar rechazos DNSBL
grep 'rejected by DNSBL' /var/log/exim/mainlog

# Ver el log de rechazos detallado
grep 'spamhaus\|spamcop' /var/log/exim/rejectlog
```

## 8.4 Verificación de Entregabilidad en Proveedores Específicos

Además de las listas negras públicas, debes verificar tu reputación directamente con los grandes proveedores, que mantienen sus propios sistemas internos de reputación.

| Servicio | URL | Qué muestra |
|----------|-----|-------------|
| **Google Postmaster Tools** | https://postmaster.google.com/ | Tasa de spam en Gmail, entregabilidad, autenticación, reputación de IP y dominio |
| **Microsoft SNDS** | https://sendersupport.olc.protection.outlook.com/snds/ | Reputación de IP, tasa de quejas, volumen de correo enviado a Outlook |
| **Yahoo Sender Hub** | https://senders.yahooinc.com/ | Feedback de Yahoo Mail sobre tu reputación |
| **Mail-Tester** | https://www.mail-tester.com/ | Puntaje general de 0 a 10 con recomendaciones específicas |
| **GlockApps** | https://glockapps.com/ | Verificación contra múltiples proveedores en tiempo real |

**Mail-Tester — La Prueba Rápida de Entregabilidad**

Mail-Tester es una herramienta gratuita que te permite evaluar la calidad de un correo específico. El proceso es simple: te dan una dirección de correo temporal, envías un correo de prueba a esa dirección, y la herramienta analiza el mensaje y te devuelve un puntaje del 0 al 10 con recomendaciones detalladas.

```bash
# Ejemplo: Obtener una direccion de prueba desde la linea de comandos
$ curl -s "https://www.mail-tester.com/api/v1/check/..."
```

## 8.5 Monitoreo Proactivo Automatizado

La mejor estrategia no es esperar a que los usuarios te digan que sus correos no llegan, sino implementar un sistema de monitoreo que te alerte automáticamente cuando tu IP aparece en una lista negra.

```bash
#!/bin/bash
# blacklist-monitor.sh — Programar en crontab: */30 * * * *

IP_LIST=("203.0.113.50" "198.51.100.20")
ALERT_EMAIL="admin@ejemplo.com"
LISTS=(
    "zen.spamhaus.org"
    "bl.spamcop.net"
    "b.barracudacentral.org"
    "psbl.surriel.com"
)

for IP in "${IP_LIST[@]}"; do
    REV_IP=$(echo $IP | awk -F. '{print $4"."$3"."$2"."$1}')
    for LIST in "${LISTS[@]}"; do
        RESULT=$(dig +short "$REV_IP.$LIST" A 2>/dev/null)
        if [ -n "$RESULT" ]; then
            echo "ALERTA: IP $IP listada en $LIST (codigo: $RESULT)" \
                | mail -s "ALERTA: IP en lista negra" "$ALERT_EMAIL"
        fi
    done
done
```

Este script debe ejecutarse cada 30 minutos desde cron. La detección temprana es crítica: una IP listada durante horas puede causar un daño reputacional significativo, mientras que una detectada y resuelta en minutos tendrá un impacto mínimo.

> 📌 **Dato clave:** Programa este script en cron cada 30 minutos en todos tus servidores de correo. La diferencia entre detectar un listado a los 30 minutos y detectarlo a las 8 horas puede significar la diferencia entre una resolución rápida y un daño reputacional que tarde semanas en repararse.
