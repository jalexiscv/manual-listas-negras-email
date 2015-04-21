# Apéndice C: Scripts Útiles para Administradores

[← Anterior](apendice-b-codigos-smtp.md) | [Índice](README.md)

---

Este apéndice contiene scripts listos para usar que todo administrador de servidores de correo debería tener en su caja de herramientas. Cada script incluye instrucciones de uso, configuración y notas importantes.

---

## C.1 Script de Diagnóstico Completo de Entregabilidad (Bash)

Este script realiza un diagnóstico exhaustivo de tu servidor de correo, verificando todos los elementos que afectan la entregabilidad: PTR, SPF, DKIM, DMARC, listas negras, puertos SMTP, y banner del servidor. Es la primera herramienta que debes ejecutar cuando sospechas de un problema de entregabilidad.

```bash
#!/bin/bash
# ============================================================
# email-diagnostic.sh — Diagnostico completo de entregabilidad
# Uso: ./email-diagnostic.sh [IP] [DOMINIO]
# ============================================================

IP="${1:-203.0.113.50}"
DOMAIN="${2:-ejemplo.com}"
REV_IP=$(echo "$IP" | awk -F. '{print $4"."$3"."$2"."$1}')

echo "=========================================="
echo "  DIAGNOSTICO DE ENTREGABILIDAD DE EMAIL"
echo "  IP: $IP | Dominio: $DOMAIN"
echo "=========================================="
echo ""

# 1. VERIFICACION DE PTR (REVERSE DNS)
echo "[1] VERIFICACION DE PTR (rDNS)"
echo "------------------------------"
PTR=$(dig +short -x "$IP")
if [ -n "$PTR" ]; then
    echo "   PTR encontrado: $PTR"
    if [[ "$PTR" == *"$DOMAIN"* ]]; then
        echo "   OK: PTR coincide con el dominio"
    else
        echo "   AVISO: PTR no menciona el dominio $DOMAIN"
    fi
else
    echo "   ERROR: PTR NO CONFIGURADO"
fi
echo ""

# 2. VERIFICACION DE SPF
echo "[2] VERIFICACION DE SPF"
echo "-----------------------"
SPF=$(dig +short TXT "$DOMAIN" | grep "v=spf1")
if [ -n "$SPF" ]; then
    echo "   SPF configurado:"
    echo "   $SPF"
else
    echo "   ERROR: SPF NO CONFIGURADO"
fi
echo ""

# 3. VERIFICACION DE DKIM
echo "[3] VERIFICACION DE DKIM"
echo "------------------------"
DKIM=$(dig +short TXT "mail._domainkey.$DOMAIN" | grep "v=DKIM1")
if [ -n "$DKIM" ]; then
    echo "   DKIM configurado (selector: mail)"
else
    echo "   ERROR: DKIM NO CONFIGURADO (selector: mail)"
fi
echo ""

# 4. VERIFICACION DE DMARC
echo "[4] VERIFICACION DE DMARC"
echo "-------------------------"
DMARC=$(dig +short TXT "_dmarc.$DOMAIN" | grep "v=DMARC1")
if [ -n "$DMARC" ]; then
    POLICY=$(echo "$DMARC" | grep -oP 'p=\K\w+')
    echo "   DMARC configurado"
    echo "   Politica: $POLICY"
else
    echo "   ERROR: DMARC NO CONFIGURADO"
fi
echo ""

# 5. VERIFICACION DE LISTAS NEGRAS
echo "[5] VERIFICACION DE LISTAS NEGRAS"
echo "---------------------------------"
LISTS=(
    "zen.spamhaus.org:Spamhaus Zen"
    "bl.spamcop.net:SpamCop"
    "b.barracudacentral.org:Barracuda BRBL"
    "psbl.surriel.com:PSBL"
    "dnsbl-1.uceprotect.net:UCEPROTECT L1"
)
for entry in "${LISTS[@]}"; do
    list="${entry%%:*}"
    name="${entry##*:}"
    result=$(dig +short "$REV_IP.$list" A 2>/dev/null)
    if [ -n "$result" ]; then
        echo "   LISTADO en $name (codigo: $result)"
    else
        echo "   Limpio en $name"
    fi
done
echo ""

# 6. VERIFICACION DE PUERTOS SMTP
echo "[6] VERIFICACION DE PUERTOS SMTP"
echo "--------------------------------"
for port in 25 587 465; do
    timeout 3 bash -c "echo > /dev/tcp/$IP/$port" 2>/dev/null && \
        echo "   Puerto $port: ABIERTO" || \
        echo "   Puerto $port: cerrado/filtrado"
done
echo ""

echo "=========================================="
echo "  DIAGNOSTICO COMPLETADO"
echo "=========================================="
```

---

## C.2 Script de Monitoreo de Reputación de Dominio (Python)

Este script en Python está diseñado para ejecutarse periódicamente (por ejemplo, desde cron una vez al día) y generar un reporte JSON con el estado de la reputación de uno o más dominios. Puede integrarse fácilmente con sistemas de monitoreo como Nagios, Zabbix o Grafana.

```python
#!/usr/bin/env python3
"""
domain-reputation-monitor.py
Monitorea la reputacion de dominios: SPF, DKIM, DMARC y listas negras.
Uso: python3 domain-reputation-monitor.py
"""

import json
import subprocess
import logging
from datetime import datetime

# CONFIGURACION
DOMAINS = ["ejemplo.com", "marketing.ejemplo.com"]
ALERT_THRESHOLD = 1  # Numeros de listas negras antes de alertar

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s"
)
log = logging.getLogger(__name__)


def dig(hostname: str, rtype: str = "A") -> str or None:
    """Realiza una consulta DNS con dig."""
    try:
        result = subprocess.run(
            ["dig", "+short", hostname, rtype],
            capture_output=True, text=True, timeout=10
        )
        output = result.stdout.strip()
        return output if output else None
    except Exception as e:
        log.error(f"Error consultando {hostname}: {e}")
        return None


def check_spf(domain: str) -> dict:
    """Verifica la configuracion SPF de un dominio."""
    result = dig(domain, "TXT")
    if result and "v=spf1" in result:
        return {"status": "PASS", "detail": result[:100]}
    return {"status": "FAIL", "detail": "No se encontro registro SPF"}


def check_dkim(domain: str, selector: str = "mail") -> dict:
    """Verifica la configuracion DKIM de un dominio."""
    host = f"{selector}._domainkey.{domain}"
    result = dig(host, "TXT")
    if result and "v=DKIM1" in result:
        return {"status": "PASS", "detail": f"Selector '{selector}' OK"}
    return {"status": "FAIL", "detail": f"Selector '{selector}' no encontrado"}


def check_dmarc(domain: str) -> dict:
    """Verifica la configuracion DMARC de un dominio."""
    host = f"_dmarc.{domain}"
    result = dig(host, "TXT")
    if result and "v=DMARC1" in result:
        policy = "unknown"
        for part in result.split(";"):
            part = part.strip()
            if part.startswith("p="):
                policy = part[2:]
        return {"status": "PASS", "detail": f"Policy: {policy}"}
    return {"status": "FAIL", "detail": "No se encontro registro DMARC"}


def check_blacklists(domain: str) -> dict:
    """Verifica si el dominio esta en listas negras de dominio."""
    lists = {
        "dbl.spamhaus.org": "Spamhaus DBL",
        "multi.surbl.org": "SURBL",
    }
    listed = []
    for zone, name in lists.items():
        result = dig(f"{domain}.{zone}")
        if result:
            listed.append({"list": name, "code": result})
    return {"total_listed": len(listed), "lists": listed}


def run() -> dict:
    """Ejecuta la verificacion completa para todos los dominios."""
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "domains": {}
    }
    for domain in DOMAINS:
        log.info(f"Verificando dominio: {domain}")
        dr = {
            "spf": check_spf(domain),
            "dkim": check_dkim(domain),
            "dmarc": check_dmarc(domain),
            "blacklists": check_blacklists(domain)
        }
        # Score: 100 - penalizaciones
        score = 100
        for key in ["spf", "dkim", "dmarc"]:
            if dr[key]["status"] == "FAIL":
                score -= 20
        score -= dr["blacklists"]["total_listed"] * 10
        dr["score"] = max(0, min(100, score))
        dr["status"] = "GOOD" if score >= 80 else ("WARNING" if score >= 50 else "CRITICAL")
        report["domains"][domain] = dr
        if dr["blacklists"]["total_listed"] >= ALERT_THRESHOLD:
            log.warning(f"Dominio {domain} en {dr['blacklists']['total_listed']} lista(s)")
    return report


if __name__ == "__main__":
    report = run()
    print(json.dumps(report, indent=2))
    with open("domain-reputation-report.json", "w") as f:
        json.dump(report, f, indent=2)
    log.info("Reporte guardado en domain-reputation-report.json")
```

---

## C.3 Configuración de Cron para Monitoreo Automático

Agrega estas líneas a tu crontab (`crontab -e`) para automatizar el monitoreo de tu servidor de correo:

```bash
# ============================================================
# MONITOREO DE EMAIL — CRONTAB
# ============================================================

# Verificacion de listas negras cada 30 minutos
*/30 * * * * /usr/local/bin/blacklist-monitor.sh > /dev/null 2>&1

# Diagnostico completo cada 6 horas (enviado por email al admin)
0 */6 * * * /usr/local/bin/email-diagnostic.sh 203.0.113.50 ejemplo.com | \
    mail -s "[MONITOREO] Diagnostico de Email" admin@ejemplo.com

# Verificacion de colas de correo cada 15 minutos
*/15 * * * * /usr/local/bin/check-queue.sh

# Analisis de logs de autenticacion (diario, 2:00 AM)
0 2 * * * /usr/local/bin/analyze-auth-logs.sh

# Reporte semanal de reputacion de dominio (lunes 9:00 AM)
0 9 * * 1 python3 /usr/local/bin/domain-reputation-monitor.py

# Rotacion de logs de monitoreo (semanal)
0 3 * * 0 logrotate /etc/logrotate.d/email-monitor
```

---

## C.4 Prueba Manual de Conexión SMTP con OpenSSL

Este script permite probar manualmente una conexión SMTP a un servidor remoto, útil para verificar que tu servidor puede conectarse correctamente y que el servidor destino responde adecuadamente.

```bash
#!/bin/bash
# smtp-test.sh — Prueba manual de conexion SMTP
# Uso: ./smtp-test.sh <servidor-destino> <puerto>

SERVER="${1:-gmail-smtp-in.l.google.com}"
PORT="${2:-25}"

echo "=========================================="
echo "  PRUEBA DE CONEXION SMTP"
echo "  Servidor: $SERVER"
echo "  Puerto: $PORT"
echo "=========================================="
echo ""

{
    echo "EHLO mail.ejemplo.com"
    sleep 1
    echo "MAIL FROM:<test@ejemplo.com>"
    sleep 1
    echo "RCPT TO:<test@gmail.com>"
    sleep 1
    echo "DATA"
    sleep 1
    echo "Subject: Prueba de conexion SMTP"
    echo "From: test@ejemplo.com"
    echo "To: test@gmail.com"
    echo ""
    echo "Este es un correo de prueba de conectividad SMTP."
    echo "."
    sleep 2
    echo "QUIT"
} | timeout 15 openssl s_client -starttls smtp -connect "$SERVER:$PORT" -crlf 2>/dev/null

echo ""
echo "=========================================="
echo "  PRUEBA COMPLETADA"
echo "=========================================="
```
