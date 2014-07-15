# Apéndice B: Códigos de Error SMTP Relacionados con Listas Negras

[← Anterior](apendice-a-dnsbl-comparativa.md) | [Índice](00-indice.md) | [Siguiente →](apendice-c-scripts.md)

---

## Rechazos por Lista Negra

### Código 550 (Permanente)

```
550 5.7.1 Service unavailable; Client host [203.0.113.50] blocked
      using zen.spamhaus.org
```
Spamhaus Zen.

```
550 5.7.1 Message rejected because <203.0.113.50> is in a black list
      at bl.spamcop.net
```
SpamCop.

```
550 5.7.1 Access denied. IP 203.0.113.50 is listed in Barracuda BRBL
```
Barracuda.

### Código 554 (Transacción Fallida)

```
554 5.7.1 Your access to this mail system has been rejected due to
      the sending MTA's poor reputation.
```
Reputación pobre (común en Microsoft 365).

### Código 450 (Temporal)

```
450 4.7.1 Service temporarily unavailable; Client host [203.0.113.50]
      blocked using zen.spamhaus.org
```
Temporal. Reintentar más tarde.

## Rechazos por Autenticación

```
550 5.7.26 This mail has been blocked because the sender is
      unauthenticated. Gmail requires SPF or DKIM.
```
Falta SPF/DKIM (Gmail).

```
550 5.7.24 Message rejected due to DMARC policy for example.com.
```
Rechazo por DMARC.

```
550 5.7.1 Messages from [203.0.113.50] weren't sent using a system
      that meets Microsoft 365 requirements.
```
Falta TLS o autenticación (Microsoft 365).

## Cómo Interpretar un Log de Rechazo

```bash
# Log de Postfix
postfix/smtpd[12345]: NOQUEUE: reject:
  RCPT from unknown[203.0.113.50]: 550 5.7.1
  Service unavailable; Client host [203.0.113.50]
  blocked using zen.spamhaus.org;
  from=<bot@ejemplo.com> to=<user@destino.com>
  proto=ESMTP helo=<mail.ejemplo.com>
```

**Claves:**
1. IP: `203.0.113.50`
2. Lista: `zen.spamhaus.org`
3. Remitente: `bot@ejemplo.com`
4. Destinatario: `user@destino.com`
5. HELO: `mail.ejemplo.com`
