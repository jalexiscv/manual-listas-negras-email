# Capítulo 6: Cómo Funcionan las DNSBL por Dentro

[← Anterior](05-principales-dnsbl.md) | [Índice](README.md) | [Siguiente →](07-motivos-de-inclusion.md)

---

## 6.1 Arquitectura Técnica

### Consulta DNS

```bash
# IP: 192.0.2.45 → invertir: 45.2.0.192 → concatenar: 45.2.0.192.zen.spamhaus.org
dig 45.2.0.192.zen.spamhaus.org A
# Respuesta (cualquier 127.0.0.0/8) = LISTADO
# NXDOMAIN = limpio
```

### Tipo A vs TXT

```bash
# TXT da información descriptiva
dig 45.2.0.192.zen.spamhaus.org TXT
# "https://www.spamhaus.org/query/ip/192.0.2.45"
```

### TTL y Caching

Las DNSBL usan TTL bajos (5-60 min) para actualizaciones rápidas.

> 💡 Si usas un resolver local (unbound, dnsmasq), configura TTL mínimos apropiados para no usar información desactualizada.

## 6.2 Scores y Ponderaciones

La mayoría de servidores **ponderan** en lugar de rechazar automáticamente:

```perl
# SpamAssassin
score RCVD_IN_SPAMHAUS  3.0
score RCVD_IN_SPAMCOP   2.5
score RCVD_IN_BARRACUDA 1.0
score RCVD_IN_PSBL      1.0
# Umbral típico: 5.0
```

## 6.3 Falsos Positivos

### Causas
1. IP reasignada (cloud) que antes usó un spammer
2. Reportes masivos incorrectos de usuarios
3. Listas demasiado agresivas (rangos CIDR)
4. Honeypots mal configurados
5. Asociación con proveedor abusivo

### Impacto
- Interrupción del negocio (facturas, confirmaciones no llegan)
- Ventas perdidas
- Carga administrativa (horas de diagnóstico)
- Daño reputacional persistente

## 6.4 Grandes Proveedores y DNSBL

| Proveedor | Usa DNSBL? | Factor principal |
|-----------|:----------:|------------------|
| **Gmail** | Sí (parcial) | Algoritmo ML propio + Postmaster Tools |
| **Outlook** | Sí (Spamhaus) | SmartScreen + SNDS |
| **Yahoo** | Históricamente | Algoritmo propio + DMARC estricto |
| **ProtonMail** | Sí (múltiples) | Conexiones cifradas obligatorias |

> 📌 Estar limpio en Spamhaus es necesario pero no suficiente para Gmail. Necesitas también buena reputación de dominio, autenticación y baja tasa de quejas.
