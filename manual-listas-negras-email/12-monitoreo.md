# Capítulo 12: Monitoreo Continuo y Herramientas

[← Anterior](11-prevencion.md) | [Índice](00-indice.md) | [Siguiente →](13-listas-blancas.md)

---

## 12.1 Herramientas de Verificación

| Herramienta | URL | Función |
|-------------|-----|---------|
| MXToolbox | https://mxtoolbox.com/blacklists.aspx | 100+ listas |
| Spamhaus Check | https://check.spamhaus.org/ | Spamhaus oficial |
| WhatIsMyIPAddress | https://whatismyipaddress.com/blacklist-check | Simple |
| DNSBL.info | https://www.dnsbl.info/ | Técnico |
| BlacklistAlert | https://www.blacklistalert.org/ | Notificaciones email |

## 12.2 Monitoreo Continuo

| Herramienta | Precio | Características |
|-------------|--------|-----------------|
| GlockApps | Desde $15/mes | 10+ verificadores, alertas |
| 250ok | Desde $50/mes | Plataforma completa |
| DMARC Analyzer | Desde $20/mes | Solo DMARC |
| Mailgun | Desde $35/mes | API + monitoreo |

## 12.3 Dashboard con Grafana

```yaml
# docker-compose.yml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus
    ports: ["9090:9090"]
  grafana:
    image: grafana/grafana
    ports: ["3000:3000"]
  postfix-exporter:
    image: prometheuscommunity/postfix-exporter
    ports: ["9154:9154"]
    volumes:
      - /var/log/mail.log:/var/log/mail.log:ro
```

## 12.4 Métricas Clave

| Métrica | Fuente | Umbral de alerta |
|---------|--------|-------------------|
| IP en lista negra | Script + API | Cualquiera |
| SPF/DKIM/DMARC | DNS check | FAIL |
| Tasa de rebotes | Logs | >5% |
| Tasa de quejas | FBL | >0.1% |
| Cola de correo | mailq | >100 |
| Fallos de autenticación | Logs | >50/hora |
| PTR correcto | DNS check | Incorrecto |
| Certificado TLS | openssl | <30 días expirar |

## 12.5 Servicios SaaS

| Servicio | Precio base |
|----------|-------------|
| GlockApps | $15/mes |
| 250ok | $50/mes |
| DMARC Analyzer | $20/mes |
| Postmark | $10/mes |
