# Capítulo 14: Estudios de Caso Reales

[← Anterior](13-listas-blancas.md) | [Índice](README.md) | [Siguiente →](15-glosario.md)

---

## 14.1 Caso 1: La Migración Fallida al Cloud

### Escenario
Una empresa mediana migró su servidor de correo de un datacenter on-premise a AWS EC2. A los tres días, todas sus IPs estaban en Spamhaus PBL.

### Causa
Las IPs de AWS EC2 están en rangos que Spamhaus clasifica como "IPs que no deberían enviar correo directamente".

### Solución
1. Configuraron Amazon SES como relay de salida.
2. Reconfiguraron Postfix para usar SES.
3. Resultado: Entregabilidad restaurada en 24 horas.

### Lección
**No envíes SMTP directo desde IPs de cloud público.** Usa servicios transaccionales.

---

## 14.2 Caso 2: El Formulario de Contacto Explotado

### Escenario
Un sitio de ecommerce tenía un formulario de contacto sin protección. Un atacante envió 50.000 correos a través del formulario en 2 horas.

### Solución
1. **Inmediata:** Desactivar el formulario.
2. **Corrección:** reCAPTCHA v3, rate limiting, honeypot, tokens CSRF.
3. **Desliste:** SpamCop salió automático en 48h. Spamhaus aceptó en 72h.

### Lección
**Todo formulario web que envíe correos debe tener protección antispam.**

---

## 14.3 Caso 3: La Lista Comprada

### Escenario
Un departamento de marketing compró una "lista de 100.000 leads calificados". Al día siguiente su dominio estaba en 8 listas negras.

### Causa
- 40% direcciones inválidas
- 15% honeypots
- 10% reportes de spam inmediatos
- Solo 5% de engagement real

### Solución
1. **Dominio quemado:** Crearon uno nuevo.
2. **Warm-up:** 8 semanas desde IPs limpias.
3. **Lista desde cero:** Double opt-in. En 6 meses tenían 15.000 suscriptores reales.
4. **Costo:** Aproximadamente $45,000 en ventas perdidas.

### Lección
**Nunca compres listas de correo.** El costo a largo plazo supera cualquier beneficio inmediato.

---

## 14.4 Caso 4: El Servidor Comprometido por Malware

### Escenario
Un servidor con WHM/cPanel fue infectado por malware que enviaba spam desde cuentas legítimas.

### Causa
Software desactualizado (Exim con vulnerabilidad CVE-2019-10149).

### Solución
1. **Aislar:** Desconectar el servidor inmediatamente.
2. **Limpieza:** Actualizar Exim, cambiar todas las contraseñas, instalar CSF.
3. **Desliste:** La IP estaba en 12 listas. Proceso de 2 semanas.

### Lección
**Mantén todo el software actualizado.** Un servidor de correo desactualizado es un objetivo prioritario.

---

## 14.5 Caso 5: El Newsletter sin Segmentación

### Escenario
Una tienda online enviaba su newsletter a 50.000 suscriptores sin segmentar. Cada semana 200-300 personas marcaban como spam.

### Solución
1. **Segmentación implementada** por engagement.
2. **Campaña de re-engagement** a inactivos.
3. **Resultado:** Perdieron 20.000 suscriptores pero la tasa de quejas cayó de 0.5% a 0.03%.

### Lección
**Menos es más.** Una lista más pequeña pero comprometida es mejor que una grande con baja calidad.

---

## 14.6 Caso 6: La Recuperación de un Dominio "Quemado"

### Escenario
Una startup hizo marketing agresivo: listas compradas, pop-ups agresivos, sin autenticación. En 3 meses su dominio principal estaba en 20+ listas negras.

### Solución (6 meses)
| Fase | Mes | Acción |
|------|-----|--------|
| 1 | 1 | Detener envío, configurar SPF/DKIM/DMARC, nuevo dominio |
| 2 | 2 | IPs limpias, warm-up con proveedor transaccional |
| 3 | 3-4 | Warm-up gradual (50 a 5.000/día) |
| 4 | 5-6 | Volumen objetivo (50.000/día), métricas en verde |

### Lección
**Tu dominio es tu reputación.** La prevención es siempre más barata que la cura.
