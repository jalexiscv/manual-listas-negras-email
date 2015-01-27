# Capítulo 12: Monitoreo Continuo y Herramientas

[← Anterior](11-prevencion.md) | [Índice](README.md) | [Siguiente →](13-listas-blancas.md)

---

## 12.1 Ecosistema de Herramientas de Verificación

El mercado de herramientas de monitoreo de entregabilidad es amplio y diverso, con opciones que van desde scripts gratuitos de código abierto hasta plataformas empresariales que cuestan miles de dólares al año. La elección de la herramienta adecuada depende del tamaño de tu operación, tu presupuesto y tus necesidades específicas de monitoreo.

### 12.1.1 Herramientas de Verificación Puntual (Gratuitas)

Estas herramientas son ideales para verificaciones rápidas y diagnósticos iniciales. No ofrecen monitoreo continuo, pero son excelentes para confirmar sospechas o verificar el estado después de un desliste.

| Herramienta | URL | Característica principal | Lo que la distingue |
|-------------|-----|--------------------------|---------------------|
| **MXToolbox** | https://mxtoolbox.com/blacklists.aspx | Verifica 100+ listas | La más completa, con enlaces directos a desliste |
| **Spamhaus Check** | https://check.spamhaus.org/ | Verificación oficial Spamhaus | Información detallada del motivo del listado |
| **WhatIsMyIPAddress** | https://whatismyipaddress.com/blacklist-check | Simple y rápido | Ideal para no técnicos |
| **DNSBL.info** | https://www.dnsbl.info/ | Técnico y detallado | Muestra el código de retorno exacto |
| **BlacklistAlert** | https://www.blacklistalert.org/ | Notificaciones por email | Te avisa cuando cambia tu estado |

### 12.1.2 Herramientas de Monitoreo Continuo (SaaS)

Estas herramientas ofrecen monitoreo 24/7, alertas automáticas y dashboards centralizados.

**GlockApps (https://glockapps.com/) — Desde $15/mes**
GlockApps es probablemente la herramienta de monitoreo más popular para pequeñas y medianas empresas. Te permite configurar verificaciones periódicas contra 10+ listas negras, y te envía alertas por email cuando detecta un listado. También incluye pruebas de entregabilidad que envían correos de prueba a buzones reales de Gmail, Outlook, Yahoo y otros proveedores para verificar dónde llegan realmente.

**250ok (https://250ok.com/) — Desde $50/mes**
250ok es una plataforma más completa orientada a empresas con equipos de email marketing dedicados. Ofrece analytics avanzados de entregabilidad, monitoreo de reputación de IP y dominio, análisis de campañas comparativas, y reportes detallados de la competencia. Su fortaleza está en la visualización de datos y las alertas configurables.

**DMARC Analyzer (https://www.dmarcanalyzer.com/) — Desde $20/mes**
Especializado exclusivamente en DMARC. Si tu principal preocupación es la autenticación y los reportes DMARC, esta herramienta ofrece el mejor análisis del mercado. Procesa los reportes XML que Google, Microsoft y otros proveedores envían a tu dirección `rua`, y los presenta en dashboards visuales con gráficos y tablas.

## 12.2 Script de Monitoreo Avanzado

Para administradores que prefieren mantener el control total sin depender de servicios externos, aquí hay un script completo que integra verificación de listas negras, DNS, puertos SMTP, colas de correo, y autenticación.

```bash
#!/bin/bash
# comprehensive-email-monitor.sh
# Monitoreo completo: listas negras, DNS, autenticacion, entregabilidad

# CONFIGURACION
DOMAIN="ejemplo.com"
IP_LIST=("203.0.113.50" "198.51.100.20")
SERVER_HOSTNAME="mail.ejemplo.com"
ALERT_EMAIL="admin@ejemplo.com"
LOGFILE="/var/log/email-monitor.log"
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")

log() {
    echo "[$TIMESTAMP] $1" | tee -a "$LOGFILE"
}

alert() {
    log "ALERTA: $1"
    echo "$1" | mail -s "ALERTA EMAIL: $1" "$ALERT_EMAIL"
}

# 1. VERIFICACION DE LISTAS NEGRAS
check_blacklists() {
    local ip="$1"
    local rev_ip=$(echo "$ip" | awk -F. '{print $4"."$3"."$2"."$1}')
    local lists=("zen.spamhaus.org" "bl.spamcop.net" "b.barracudacentral.org")
    for list in "${lists[@]}"; do
        result=$(dig +short "$rev_ip.$list" A 2>/dev/null)
        [ -n "$result" ] && alert "IP $ip LISTADA en $list (codigo: $result)"
    done
}

# 2. VERIFICACION DNS
check_dns() {
    # SPF
    spf=$(dig +short TXT "$DOMAIN" | grep "v=spf1")
    [ -z "$spf" ] && alert "SPF NO CONFIGURADO para $DOMAIN" || log "SPF: OK"
    # DMARC
    dmarc=$(dig +short TXT "_dmarc.$DOMAIN" | grep "v=DMARC1")
    [ -z "$dmarc" ] && alert "DMARC NO CONFIGURADO para $DOMAIN" || log "DMARC: OK"
    # DKIM
    dkim=$(dig +short TXT "mail._domainkey.$DOMAIN" | grep "v=DKIM1")
    [ -z "$dkim" ] && alert "DKIM NO CONFIGURADO para $DOMAIN" || log "DKIM: OK"
    # PTR
    for ip in "${IP_LIST[@]}"; do
        ptr=$(dig +short -x "$ip" 2>/dev/null)
        [ "$ptr" != "$SERVER_HOSTNAME." ] && \
            alert "PTR incorrecto para $ip: $ptr (esperado: $SERVER_HOSTNAME)" \
            || log "PTR correcto para $ip: $ptr"
    done
}

# 3. VERIFICACION DE COLAS
check_queues() {
    local queue_count=$(mailq 2>/dev/null | grep -c "^[0-9A-F]")
    [ "$queue_count" -gt 100 ] && alert "Cola grande: $queue_count mensajes" || \
        log "Cola: $queue_count mensajes"
}

# EJECUCION
log "=== Iniciando monitoreo ==="
for ip in "${IP_LIST[@]}"; do check_blacklists "$ip"; done
check_dns
check_queues
log "=== Monitoreo completado ==="
```

## 12.3 Dashboard de Monitoreo con Grafana y Prometheus

Para un monitoreo más visual y en tiempo real, puedes montar un stack de monitoreo con Prometheus y Grafana.

```yaml
# docker-compose.yml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana

  postfix-exporter:
    image: prometheuscommunity/postfix-exporter:latest
    ports:
      - "9154:9154"
    volumes:
      - /var/log/mail.log:/var/log/mail.log:ro

volumes:
  grafana-data:
```

### Métricas Clave para el Dashboard

| Métrica | Fuente de datos | Umbral de alerta | Acción |
|---------|----------------|-------------------|--------|
| IP en lista negra | Script personalizado | Cualquier listado | Iniciar proceso de desliste |
| SPF/DKIM/DMARC | Consulta DNS | Cualquier fallo | Reconfigurar inmediatamente |
| Tasa de rebotes | Análisis de logs | >5% | Limpiar lista de correos |
| Tasa de quejas | FBL/Postmaster | >0.1% | Reducir frecuencia, revisar segmentación |
| Volumen de envío | Logs de Postfix | Pico >3x promedio | Investigar posible compromiso |
| Cola de correo | mailq | >100 mensajes | Revisar si hay envíos no autorizados |
| Fallos de autenticación | Logs de Postfix | >50/hora | Posible ataque de diccionario |
| PTR correcto | Consulta DNS | Incorrecto | Solicitar cambio al proveedor |
| Certificado TLS | openssl check | <30 días para expirar | Renovar certificado |

## 12.4 Servicios SaaS de Monitoreo Comparados

| Servicio | Precio mensual | Listas negras | Entregabilidad | Alertas | Analytics |
|----------|:--------------:|:-------------:|:--------------:|:-------:|:---------:|
| **GlockApps** | $15+ | ✅ | ✅ | ✅ | ✅ Básico |
| **250ok** | $50+ | ✅ | ✅ | ✅ | ✅ Avanzado |
| **DMARC Analyzer** | $20+ | ❌ | ❌ | ✅ | ✅ DMARC |
| **Mailgun** | $35+ | Incluido | Incluido | ✅ | ✅ |
| **Postmark** | $10+ | Incluido | Incluido | ✅ | ✅ Básico |
