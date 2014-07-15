# Capítulo 11: Prevención — Buenas Prácticas de Envío

[← Anterior](10-estrategias-desliste.md) | [Índice](00-indice.md) | [Siguiente →](12-monitoreo.md)

---

## 11.1 Configuración Técnica

### Registros DNS Esenciales

```txt
; MX
ejemplo.com.    IN  MX  10  mail.ejemplo.com.

; PTR (rDNS) — configurado por el proveedor
50.113.0.203.in-addr.arpa.  PTR  mail.ejemplo.com.

; SPF
ejemplo.com.    IN  TXT  "v=spf1 mx include:_spf.google.com -all"

; DKIM
mail._domainkey.ejemplo.com.  IN  TXT  "v=DKIM1; h=sha256; k=rsa; p=..."

; DMARC
_dmarc.ejemplo.com.  IN  TXT  "v=DMARC1; p=quarantine; rua=mailto:dmarc@ejemplo.com"
```

### Postfix

```bash
# Rate limiting
anvil_rate_time_unit = 60s
smtpd_client_connection_rate_limit = 30

# DNSBL
smtpd_recipient_restrictions =
    permit_mynetworks
    permit_sasl_authenticated
    reject_rbl_client zen.spamhaus.org
    reject_rbl_client bl.spamcop.net
    permit
```

## 11.2 Gestión de Listas

### Double Opt-In
1. Usuario ingresa email
2. Se envía confirmación
3. Usuario hace clic
4. Solo entonces se añade

### Bajas
```bash
# Cabeceras obligatorias
List-Unsubscribe: <mailto:unsubscribe@ejemplo.com>
List-Unsubscribe: <https://ejemplo.com/unsubscribe>
List-Unsubscribe-Post: List-Unsubscribe=One-Click
```

### Higiene Mensual
- Eliminar suscriptores sin apertura en 6+ meses
- Eliminar hard bounces
- Campañas de re-engagement antes de eliminar

## 11.3 Segmentación

| Segmento | Definición | Estrategia |
|----------|------------|------------|
| Activos | Abrieron en 30 días | Envío normal |
| Tibios | 31-90 días | Envío reducido |
| Inactivos | 90-180 días | Re-engagement |
| Dormidos | 180+ días | No enviar |

## 11.4 Métricas de Salud

| Métrica | Bueno | Alerta | Crítico |
|---------|-------|--------|---------|
| Apertura | >25% | 15-25% | <15% |
| Clics | >3% | 1-3% | <1% |
| Rebotes | <2% | 2-5% | >5% |
| Quejas spam | <0.1% | 0.1-0.5% | >0.5% |

## 11.5 Feedback Loops (FBL)

| Proveedor | Programa |
|-----------|----------|
| Google | Postmaster Tools |
| Microsoft | JMRP + SNDS |
| Yahoo | Sender Hub |

> 📌 Tasa de quejas >0.1% es problemática. Gmail es especialmente sensible.
