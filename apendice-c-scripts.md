# Apéndice C: Scripts Útiles para Administradores

[← Anterior](apendice-b-codigos-smtp.md) | [Índice](README.md)

---

## C.1 Diagnóstico Completo (Bash)

```bash
#!/bin/bash
# email-diagnostic.sh — Uso: ./email-diagnostic.sh <IP> <DOMINIO>

IP="${1:-203.0.113.50}"
DOMAIN="${2:-ejemplo.com}"
REV_IP=$(echo "$IP" | awk -F. '{print $4"."$3"."$2"."$1}')

echo "=== DIAGNOSTICO DE ENTREGABILIDAD ==="
echo "IP: $IP | Dominio: $DOMAIN"

# 1. PTR
PTR=$(dig +short -x "$IP")
[ -n "$PTR" ] && echo "PTR: $PTR" || echo "PTR: NO CONFIGURADO"

# 2. SPF
SPF=$(dig +short TXT "$DOMAIN" | grep "v=spf1")
[ -n "$SPF" ] && echo "SPF: OK" || echo "SPF: NO CONFIGURADO"

# 3. DKIM
DKIM=$(dig +short TXT "mail._domainkey.$DOMAIN" | grep "v=DKIM1")
[ -n "$DKIM" ] && echo "DKIM: OK" || echo "DKIM: NO CONFIGURADO"

# 4. DMARC
DMARC=$(dig +short TXT "_dmarc.$DOMAIN" | grep "v=DMARC1")
if [ -n "$DMARC" ]; then
    POLICY=$(echo "$DMARC" | grep -oP 'p=\K\w+')
    echo "DMARC: OK (politica: $POLICY)"
else
    echo "DMARC: NO CONFIGURADO"
fi

# 5. Listas negras
for entry in "zen.spamhaus.org:Spamhaus" "bl.spamcop.net:SpamCop" \
             "b.barracudacentral.org:Barracuda" "psbl.surriel.com:PSBL"; do
    list="${entry%%:*}"
    name="${entry##*:}"
    result=$(dig +short "$REV_IP.$list" A)
    [ -n "$result" ] && echo "$name: LISTADO ($result)" || echo "$name: limpio"
done

# 6. Puertos
for port in 25 587 465; do
    timeout 2 bash -c "echo > /dev/tcp/$IP/$port" 2>/dev/null \
        && echo "Puerto $port: abierto" || echo "Puerto $port: cerrado"
done
```

## C.2 Monitoreo de Reputación (Python)

```python
#!/usr/bin/env python3
"""domain-reputation-monitor.py"""
import json, subprocess, logging
from datetime import datetime

DOMAINS = ["ejemplo.com"]
log = logging.getLogger(__name__)

def dig(h, t="A"):
    r = subprocess.run(["dig", "+short", h, t],
        capture_output=True, text=True, timeout=10)
    return r.stdout.strip() or None

def check_spf(d):
    r = dig(d, "TXT")
    if r and "v=spf1" in r:
        return {"status": "PASS", "detail": r[:100]}
    return {"status": "FAIL", "detail": "No SPF"}

def check_dkim(d, sel="mail"):
    r = dig(f"{sel}._domainkey.{d}", "TXT")
    if r and "v=DKIM1" in r:
        return {"status": "PASS", "detail": f"Selector {sel} OK"}
    return {"status": "FAIL", "detail": f"Selector {sel} no encontrado"}

def check_dmarc(d):
    r = dig(f"_dmarc.{d}", "TXT")
    if r and "v=DMARC1" in r:
        p = "unknown"
        for part in r.split(";"):
            if part.strip().startswith("p="):
                p = part.strip()[2:]
        return {"status": "PASS", "detail": f"Policy: {p}"}
    return {"status": "FAIL", "detail": "No DMARC"}

def check_bl(d):
    listed = []
    for zone, name in {"dbl.spamhaus.org": "Spamhaus DBL",
                        "multi.surbl.org": "SURBL"}.items():
        r = dig(f"{d}.{zone}")
        if r:
            listed.append({"list": name, "code": r})
    return {"total_listed": len(listed), "lists": listed}

if __name__ == "__main__":
    report = {"timestamp": datetime.utcnow().isoformat(), "domains": {}}
    for d in DOMAINS:
        dr = {"spf": check_spf(d), "dkim": check_dkim(d),
              "dmarc": check_dmarc(d), "blacklists": check_bl(d)}
        score = 100
        for k in ["spf", "dkim", "dmarc"]:
            if dr[k]["status"] == "FAIL": score -= 20
        score -= dr["blacklists"]["total_listed"] * 10
        dr["score"] = max(0, min(100, score))
        report["domains"][d] = dr
    print(json.dumps(report, indent=2))
```

## C.3 Configuración Cron

```bash
# crontab
# Monitoreo cada 30 minutos
*/30 * * * * /usr/local/bin/blacklist-monitor.sh

# Diagnostico cada 6 horas
0 */6 * * * /usr/local/bin/email-diagnostic.sh 203.0.113.50 ejemplo.com

# Verificacion de colas cada 15 minutos
*/15 * * * * /usr/local/bin/check-queue.sh

# Reporte semanal
0 9 * * 1 python3 /usr/local/bin/domain-reputation-monitor.py
```
