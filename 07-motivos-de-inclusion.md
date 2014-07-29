# Capítulo 7: Motivos Comunes para Ser Incluido

[← Anterior](06-como-funcionan.md) | [Índice](README.md) | [Siguiente →](08-como-detectar.md)

---

## 7.1 Comportamiento del Servidor

### Volumen Inusual
- Cambios bruscos: 1.000/día → 100.000/día
- Picos >10x el volumen normal
- Volumen alto desde IP nueva

### Alta Tasa de Rebotes (Hard Bounces)
- >5% = problemático. >10% = listado casi seguro.
- Causas: listas compradas, no limpiar inactivos, mal manejo de bajas

### Formatos Incorrectos
- `HELO` con IP en lugar de nombre
- `HELO` con nombre que no resuelve
- Falta de registro PTR (rDNS)

### Conexiones Sin Cifrado
- Sin STARTTLS
- Envío por puerto 25 en lugar de 587

## 7.2 Contenido del Mensaje

### Asuntos Problemáticos
- TODO EN MAYÚSCULAS
- Signos de exclamación!!!
- "GRATIS", "GANE DINERO", "CLICK AQUÍ"
- Sujetos que imitan respuestas ("Re:", "Fwd:")

### Relación Texto/Imagen
- Una sola imagen grande = sospechoso
- <20% texto es mala señal
- Texto invisible (color = fondo)

## 7.3 Infraestructura

### IP Residencial o Dinámica (PBL)
IPs de rangos ISP domésticos NO deben enviar correo directo.

**Solución:** Usa servicio transaccional (SendGrid, Mailgun, Amazon SES) o IP dedicada.

### Falta de Configuración DNS
- Sin PTR
- PTR no coincide con HELO
- Sin MX
- Sin SPF

## 7.4 Prácticas de Marketing Cuestionables

| Práctica | Problema |
|----------|----------|
| **Listas compradas** | La forma más rápida de ser marcado como spam |
| **Scraping** | Violación legal (GDPR, CCPA, LGPD) |
| **Single opt-in** | Altas tasas de queja |
| **Sin enlace de baja** | Ilegal en la mayoría de jurisdicciones |

## 7.5 Seguridad Comprometida

### Servidor Comprometido
- Colas de correo inusualmente largas
- Tráfico SMTP en horas sin actividad
- Scripts PHP/CGI desconocidos

### Formularios Explotados
**Protección:** Captcha, rate limiting, campos honeypot, tokens CSRF.
