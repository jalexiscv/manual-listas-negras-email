# Capítulo 2: Fundamentos Técnicos del Correo Electrónico

[← Anterior](01-introduccion.md) | [Índice](README.md) | [Siguiente →](03-que-es-una-lista-negra.md)

---

## 2.1 Anatomía de un Correo Electrónico

### 2.1.1 El Sobre vs. El Contenido

**El sobre (SMTP envelope):** `MAIL FROM` (return-path) y `RCPT TO`. No visible para el usuario.

**El contenido (message header + body):** `From:`, `To:`, `Subject:`, `Date:`, `Message-ID:`, cuerpo del mensaje.

> 📌 **Dato clave:** El `MAIL FROM` y el `From:` pueden ser diferentes. Las listas negras operan sobre la IP del remitente o el dominio del `MAIL FROM`.

### 2.1.2 El Viaje de un Correo: Paso a Paso

1. Composición en MUA (Outlook, Gmail, Thunderbird)
2. Envío al MSA (puerto 587, STARTTLS)
3. Transferencia al MTA (Postfix, Exim, Sendmail)
4. Resolución MX (consulta DNS)
5. Transferencia SMTP al destino
6. Verificaciones antispam: listas negras, SPF, DKIM, DMARC
7. Entrega o rechazo

## 2.2 SMTP: El Protocolo del Correo

### Comandos SMTP Esenciales

| Comando | Función | Ejemplo |
|---------|---------|---------|
| `HELO/EHLO` | Saludo | `EHLO mail.ejemplo.com` |
| `MAIL FROM` | Remitente | `MAIL FROM:<bot@ejemplo.com>` |
| `RCPT TO` | Destinatario | `RCPT TO:<user@destino.com>` |
| `DATA` | Contenido | `DATA` |
| `QUIT` | Cierre | `QUIT` |

### Códigos de Respuesta SMTP

| Código | Significado |
|--------|-------------|
| 220 | Servicio listo |
| 250 | Acción completada |
| 354 | Iniciar entrada de mensaje |
| 450 | Error temporal |
| 550 | Error permanente |
| 554 | Transacción fallida |

> ⚠️ **Advertencia:** Códigos 5xx = error permanente. Códigos 4xx = temporal (reintentar después).

## 2.3 El DNS y las Listas Negras

### Cómo una DNSBL Usa el DNS

1. IP a verificar: `192.0.2.50`
2. Invertir octetos: `50.2.0.192`
3. Concatenar con zona DNSBL: `50.2.0.192.zen.spamhaus.org`
4. Consulta tipo A
5. Si hay respuesta → IP listada. Si NXDOMAIN → IP limpia.

```bash
dig +short 2.0.0.127.zen.spamhaus.org
# 127.0.0.2 = listado
```

> 📌 **Dato clave:** Esta arquitectura permite que **cualquier servidor DNS** consulte listas negras sin instalar software especial.
