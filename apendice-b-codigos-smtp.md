# Apéndice B: Códigos de Error SMTP Relacionados con Listas Negras

[← Anterior](apendice-a-dnsbl-comparativa.md) | [Índice](README.md) | [Siguiente →](apendice-c-scripts.md)

---

Este apéndice proporciona una guía de referencia rápida para interpretar los códigos de error SMTP más comunes generados por listas negras y problemas de autenticación. Cada entrada incluye el código exacto, una explicación de su significado, la acción que debes tomar, y el proveedor o lista que típicamente genera ese error.

---

## Rechazos por Lista Negra (DNSBL)

### Código 550 — Error Permanente: Listado en Spamhaus Zen

```
550 5.7.1 Service unavailable; Client host [203.0.113.50] blocked using
      zen.spamhaus.org; https://check.spamhaus.org/query/ip/203.0.113.50
```

**Significado:** La IP `203.0.113.50` está listada en una de las listas de Spamhaus (SBL, XBL o PBL). El enlace proporcionado lleva a la página de información donde puedes ver el motivo exacto.

**Acción requerida:** Acceder al enlace, identificar qué sub-lista (SBL, XBL, PBL) contiene tu IP, leer la explicación del motivo, corregir la causa raíz, y solicitar el desliste a través del formulario de Spamhaus.

---

### Código 550 — Error Permanente: Listado en SpamCop

```
550 5.7.1 Message rejected because <203.0.113.50> is in a black list at
      bl.spamcop.net
```

**Significado:** La IP ha sido reportada como fuente de spam por múltiples usuarios a través del sistema SpamCop.

**Acción requerida:** Identificar por qué los usuarios están reportando tus correos. Si es un falso positivo, esperar 24-48 horas para el desliste automático o solicitar desliste manual. Si son reportes legítimos, revisar tus prácticas de envío (frecuencia, segmentación, consentimiento).

---

### Código 550 — Error Permanente: Listado en Barracuda BRBL

```
550 5.7.1 Access denied. IP 203.0.113.50 is listed in Barracuda BRBL
```

**Significado:** La IP está listada en la Barracuda Reputation Block List por comportamiento abusivo detectado por los sensores de Barracuda Networks.

**Acción requerida:** Cesar inmediatamente cualquier actividad abusiva. El desliste es automático cuando el comportamiento cesa. No hay formulario de desliste manual público.

---

### Código 554 — Error de Transacción: Reputación Pobre (Microsoft 365)

```
554 5.7.1 Your access to this mail system has been rejected due to
      the sending MTA's poor reputation. If you believe this failure
      is in error, please contact the intended recipient via alternate means.
```

**Significado:** La IP tiene mala reputación en el sistema de Microsoft (Outlook/Office 365). Microsoft no especifica qué lista negra utilizó ni el motivo exacto.

**Acción requerida:** Verificar tu IP en las principales listas públicas, revisar tu reputación en Microsoft SNDS (https://sendersupport.olc.protection.outlook.com/snds/), corregir problemas de autenticación, y reducir la tasa de quejas.

---

### Código 450 — Error Temporal: Listado en Lista Negra

```
450 4.7.1 Service temporarily unavailable; Client host [203.0.113.50]
      blocked using zen.spamhaus.org
```

**Significado:** Similar al código 550, pero el servidor está utilizando un código temporal (4xx) en lugar de permanente. Esto puede deberse a que el servidor está usando greylisting combinado con verificación de listas, o a que la configuración del servidor está usando un código temporal para dar oportunidad de corrección.

**Acción requerida:** La misma que para el 550, pero con menos urgencia porque el servidor receptor podría aceptar el correo en un reintento posterior. Sin embargo, es mejor resolver el listado de todas formas.

---

## Rechazos por Autenticación (SPF, DKIM, DMARC)

### Código 550 — Error Permanente: SPF/DKIM no configurado (Gmail)

```
550 5.7.26 This mail has been blocked because the sender is unauthenticated.
      Gmail requires all senders to authenticate with either SPF or DKIM.
      See https://support.google.com/mail/answer/81126
```

**Significado:** El dominio del remitente no tiene SPF ni DKIM configurados. Gmail exige al menos uno de los dos.

**Acción requerida:** Configurar SPF en el DNS del dominio (registro TXT con `v=spf1 ... -all`) y/o DKIM (generar par de claves y publicar la clave pública en el DNS).

---

### Código 550 — Error Permanente: Política DMARC (Rechazo por DMARC)

```
550 5.7.24 Message rejected due to DMARC policy for example.com.
      See https://support.google.com/mail/?p=dmarc_rejection
```

**Significado:** El dominio `ejemplo.com` tiene DMARC configurado con política `p=reject`, y este correo no pasó la verificación SPF ni DKIM.

**Acción requerida:** Si eres el remitente, asegúrate de que tus servidores estén correctamente configurados con SPF y DKIM, y que la alineación DMARC sea correcta. Si eres el destinatario, contacta al remitente del dominio para que revise su configuración.

---

### Código 550 — Error Permanente: Requisitos de Microsoft 365

```
550 5.7.1 Unfortunately, messages from [203.0.113.50] weren't sent using
      a system that meets Microsoft 365 requirements.
      Please visit https://protection.office.com/
```

**Significado:** La IP no cumple con los requisitos técnicos de Microsoft 365, que incluyen: tener PTR configurado, soportar STARTTLS, tener SPF configurado, y no estar en listas negras conocidas.

**Acción requerida:** Verificar y corregir cada uno de los requisitos: PTR, TLS, SPF, DKIM, y estado en listas negras.

---

## Cómo Interpretar una Línea Completa de Log con Rechazo

Cuando un servidor Postfix rechaza un correo, la línea de log contiene toda la información necesaria para diagnosticar el problema:

```bash
postfix/smtpd[12345]: NOQUEUE: reject: RCPT from unknown[203.0.113.50]:
550 5.7.1 Service unavailable; Client host [203.0.113.50] blocked using
zen.spamhaus.org; from=<bot@ejemplo.com> to=<user@destino.com>
proto=ESMTP helo=<mail.ejemplo.com>
```

**Elementos a extraer del log:**

| Elemento | Valor en el ejemplo | Qué indica |
|----------|---------------------|------------|
| **Componente** | `postfix/smtpd[12345]` | El proceso que generó el rechazo (Postfix SMTP daemon) |
| **Acción** | `NOQUEUE: reject` | El mensaje fue rechazado antes de entrar a la cola |
| **IP del remitente** | `[203.0.113.50]` | La IP que está siendo rechazada |
| **Código SMTP** | `550 5.7.1` | Error permanente: Service unavailable |
| **Lista negra** | `zen.spamhaus.org` | La lista que contiene la IP |
| **Remitente** | `bot@ejemplo.com` | La dirección en MAIL FROM |
| **Destinatario** | `user@destino.com` | El destinatario al que se intentaba enviar |
| **HELO** | `mail.ejemplo.com` | El nombre declarado en el saludo SMTP |
