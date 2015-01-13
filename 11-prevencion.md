# Capítulo 11: Prevención — Buenas Prácticas de Envío

[← Anterior](10-estrategias-desliste.md) | [Índice](README.md) | [Siguiente →](12-monitoreo.md)

---

## 11.1 Configuración Técnica del Servidor de Correo

La prevención comienza con una configuración técnica sólida. Antes de enviar el primer correo, asegúrate de que tu servidor cumple con todos los requisitos técnicos que los servidores receptores esperan.

### 11.1.1 Registros DNS Esenciales (Lista de Verificación)

Cada dominio desde el que envíes correo debe tener los siguientes registros DNS configurados y verificados:

```txt
; 1. Registro MX — Indica que servidor recibe correo
ejemplo.com.        IN  MX  10  mail.ejemplo.com.

; 2. Registro A del servidor de correo
mail.ejemplo.com.   IN  A   203.0.113.50

; 3. Registro PTR (rDNS) — LO DEBES SOLICITAR A TU PROVEEDOR
; 50.113.0.203.in-addr.arpa.  PTR  mail.ejemplo.com.

; 4. SPF — Autoriza servidores de envio
ejemplo.com.        IN  TXT  "v=spf1 mx include:_spf.google.com -all"

; 5. DKIM — Firma digital (selector: mail)
mail._domainkey.ejemplo.com.  IN  TXT  "v=DKIM1; h=sha256; k=rsa; p=..."

; 6. DMARC — Politica de autenticacion
_dmarc.ejemplo.com.  IN  TXT  "v=DMARC1; p=quarantine; rua=mailto:dmarc@ejemplo.com"

; 7. BIMI (opcional pero recomendado)
default._bimi.ejemplo.com.  IN  TXT  "v=BIMI1; l=https://ejemplo.com/logo.svg"
```

**Cómo verificar cada registro:**

```bash
# Verificar MX
$ dig +short MX ejemplo.com

# Verificar que el PTR coincide con el HELO
$ dig +short -x 203.0.113.50
$ dig +short mail.ejemplo.com A

# Verificar SPF
$ dig +short TXT ejemplo.com | grep "v=spf1"

# Verificar DKIM
$ dig +short TXT mail._domainkey.ejemplo.com | grep "v=DKIM1"

# Verificar DMARC
$ dig +short TXT _dmarc.ejemplo.com | grep "v=DMARC1"
```

### 11.1.2 Configuración de Postfix para Máxima Entregabilidad

```bash
# /etc/postfix/main.cf — Configuracion optimizada para deliverability

# Identificacion del servidor
myhostname = mail.ejemplo.com
myorigin = $myhostname

# Seguridad basica del protocolo SMTP
smtpd_helo_required = yes
strict_rfc821_envelopes = yes
disable_vrfy_command = yes

# Rate limiting para prevenir abusos
anvil_rate_time_unit = 60s
smtpd_client_connection_rate_limit = 30
smtpd_client_message_rate_limit = 10

# Limites para proteger contra ataques de diccionario
smtpd_soft_error_limit = 10
smtpd_hard_error_limit = 20

# TLS obligatorio
smtpd_tls_security_level = may
smtpd_tls_protocols = !SSLv2, !SSLv3
smtp_tls_security_level = may
smtp_tls_protocols = !SSLv2, !SSLv3

# Consulta de DNSBL (lista negra) para correo entrante
smtpd_recipient_restrictions =
    permit_mynetworks
    permit_sasl_authenticated
    reject_invalid_hostname
    reject_non_fqdn_hostname
    reject_non_fqdn_sender
    reject_non_fqdn_recipient
    reject_unauth_destination
    reject_rhsbl_reverse_client dbl.spamhaus.org
    reject_rbl_client zen.spamhaus.org
    reject_rbl_client bl.spamcop.net
    permit
```

### 11.1.3 Conexiones Salientes Seguras

Los servidores receptores (especialmente Microsoft 365 y ProtonMail) penalizan las conexiones entrantes que no soportan STARTTLS. Para tu servidor emisor:

```bash
# En /etc/postfix/mainf.cf (parametros de envio)
smtp_tls_security_level = may
smtp_tls_protocols = !SSLv2, !SSLv3
smtp_tls_ciphers = high
```

## 11.2 Gestión Profesional de Listas de Correo

### 11.2.1 Double Opt-In: El Estándar de Oro

El double opt-in no es opcional: es la práctica mínima aceptable para cualquier lista de correo profesional. El proceso es:

1. El usuario ingresa su dirección de correo en un formulario de suscripción
2. El sistema envía un correo de confirmación a esa dirección con un enlace único
3. El usuario debe hacer clic en el enlace para confirmar su suscripción
4. Solo después de la confirmación, la dirección se añade a la lista activa

**Beneficios medibles del double opt-in:**
- Elimina direcciones mal escritas (errores tipográficos)
- Elimina suscripciones fraudulentas (bots, direcciones temporales)
- Elimina suscripciones de personas que no recuerdan haberse registrado
- Reduce la tasa de quejas de spam del 0.5% al 0.05% o menos
- Mejora la tasa de apertura entre un 15% y un 30%

**Ejemplo de implementación simplificada en Python:**

```python
import hashlib, hmac
from datetime import datetime, timedelta

SECRET_KEY = "clave-secreta-cambiar-en-produccion"

def generar_token(email, lista_id):
    """Genera un token de confirmacion valido por 48 horas."""
    expiry = int((datetime.utcnow() + timedelta(hours=48)).timestamp())
    mensaje = f"{email}:{lista_id}:{expiry}"
    firma = hmac.new(SECRET_KEY.encode(), mensaje.encode(), hashlib.sha256).hexdigest()
    return f"{firma}:{expiry}"

def verificar_token(token, email, lista_id):
    """Verifica que el token sea valido y no haya expirado."""
    try:
        firma_recibida, expiry = token.rsplit(":", 1)
        mensaje = f"{email}:{lista_id}:{expiry}"
        firma_esperada = hmac.new(SECRET_KEY.encode(), mensaje.encode(), hashlib.sha256).hexdigest()
        if not hmac.compare_digest(firma_recibida, firma_esperada):
            return False
        return datetime.utcnow().timestamp() < float(expiry)
    except (ValueError, TypeError):
        return False
```

### 11.2.2 Gestión de Bajas (Unsubscribe)

Cada correo que envíes debe cumplir con estos requisitos mínimos:

1. **Enlace de baja visible y funcional:** No oculto en la letra pequeña del pie de página. Debe ser un enlace de texto claramente identificable.
2. **Procesamiento inmediato:** La baja debe procesarse en menos de 24 horas (idealmente en tiempo real).
3. **Sin barreras:** No pedir inicio de sesión, ni confirmación adicional, ni llenar formularios.
4. **Cabecera List-Unsubscribe:** Estándar RFC 2369 para baja automática.
5. **One-Click Unsubscribe:** RFC 8058, soportado por Gmail y Outlook.

```bash
# Cabeceras de baja en el correo
List-Unsubscribe: <mailto:unsubscribe@ejemplo.com?subject=baja>
List-Unsubscribe: <https://ejemplo.com/unsubscribe?token=abc123>
List-Unsubscribe-Post: List-Unsubscribe=One-Click
```

### 11.2.3 Higiene Periódica de la Lista

Una lista de correo no se mantiene sola. Requiere limpieza periódica:

**Cada mes:**
- Identificar suscriptores que no han abierto ningún correo en los últimos 6 meses
- Enviar una campaña de re-engagement a esos suscriptores ("¿Sigues interesado en recibir nuestros correos?")
- Eliminar los que no respondan a la campaña de re-engagement
- Eliminar direcciones que han rebotado (hard bounce) más de una vez

**Cada trimestre:**
- Analizar la tasa de crecimiento de la lista vs. la tasa de bajas
- Verificar contra listas de direcciones honeypot conocidas (si tienes acceso)
- Revisar la segmentación actual y ajustar si es necesario

## 11.3 Segmentación por Engagement

No todos los suscriptores son iguales. Enviar el mismo mensaje con la misma frecuencia a todos es la receta para tener altas tasas de quejas.

| Segmento | Definición | Frecuencia de envío | Contenido |
|----------|------------|---------------------|-----------|
| **Activos** | Abrieron en los últimos 30 días | Normal (según programa) | Contenido completo |
| **Tibios** | Abrieron entre 31 y 90 días | Reducida (50% de la frecuencia normal) | Contenido selecto, re-engagement suave |
| **Inactivos** | Abrieron entre 90 y 180 días | Muy reducida (1 vez al mes máximo) | Solo campañas de re-engagement |
| **Dormidos** | No abrieron en más de 180 días | No enviar | Solo un correo final de reactivación antes de eliminar |

## 11.4 Métricas de Salud para Monitoreo Continuo

Establece umbrales de alerta para estas métricas y actúa inmediatamente cuando se superen:

| Métrica | Excelente | Alerta | Crítico | Acción requerida |
|---------|:---------:|:------:|:-------:|------------------|
| Tasa de apertura | >30% | 15-30% | <15% | Revisar asuntos, segmentación, frecuencia |
| Tasa de clics | >5% | 2-5% | <2% | Revisar contenido, llamadas a la acción |
| Tasa de rebotes | <2% | 2-5% | >5% | Limpiar lista urgentemente |
| Tasa de quejas | <0.05% | 0.05-0.1% | >0.1% | Revisar consentimiento, frecuencia, contenido |
| Tasa de bajas | <0.2% | 0.2-0.5% | >0.5% | Revisar propuesta de valor, frecuencia |

## 11.5 Feedback Loops: La Línea Directa con los Proveedores

Los Feedback Loops (FBL) son programas que los proveedores de correo ofrecen a los remitentes para notificarles cuando un usuario marca sus correos como spam. Configurar FBL es una de las medidas preventivas más importantes que puedes tomar, porque te permite detectar problemas de quejas antes de que resulten en listados en listas negras.

| Proveedor | Programa | URL de registro |
|-----------|----------|-----------------|
| **Google** | Postmaster Tools | https://postmaster.google.com/ |
| **Microsoft** | SNDS + JMRP | https://sendersupport.olc.protection.outlook.com/snds/ |
| **Yahoo** | Sender Hub | https://senders.yahooinc.com/ |
| **AOL** | AOL Postmaster | https://postmaster.aol.com/ |

> 📌 **Dato clave:** Una tasa de quejas superior a 0.1% (1 queja por cada 1.000 correos entregados) es considerada problemática por la mayoría de los proveedores. Gmail es particularmente sensible: si tu tasa de quejas supera el 0.1% en Gmail, tu reputación caerá a "Mala" en Postmaster Tools y tus correos irán sistemáticamente a spam. La tasa de quejas es, con diferencia, la métrica más importante para mantener tu reputación con los grandes proveedores.
