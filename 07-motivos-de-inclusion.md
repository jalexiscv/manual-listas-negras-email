# Capítulo 7: Motivos Comunes para Ser Incluido en una Lista Negra

[← Anterior](06-como-funcionan.md) | [Índice](README.md) | [Siguiente →](08-como-detectar.md)

---

## 7.1 Comportamiento del Servidor SMTP

El comportamiento de tu servidor de correo durante el proceso de conexión SMTP es una de las primeras cosas que los sistemas antispam evalúan. Pequeños errores de configuración pueden desencadenar listados automáticos antes siquiera de que el contenido del mensaje sea examinado.

### 7.1.1 Volumen de Envío Inusual

Los sistemas de detección de anomalías de los proveedores de correo y las listas negras monitorean constantemente el volumen de correo que envía cada IP. Cuando detectan un cambio brusco y significativo en el volumen, lo interpretan como una señal de comportamiento abusivo, incluso si el contenido del correo es legítimo. Esto se debe a que los spammers a menudo rotan IPs: comienzan con volumen bajo para "calentar" la IP y luego, de repente, explotan el volumen para enviar millones de correos antes de que la IP sea detectada y bloqueada.

**Ejemplos de cambios que activan alarmas:**
- Un servidor que envía 1.000 correos al día de forma consistente durante meses, y de repente envía 100.000 en un solo día
- Una IP nueva (con menos de 30 días de antigüedad) que comienza a enviar más de 10.000 correos diarios desde el primer día
- Un aumento del 1.000% o más en el volumen horario sin una razón justificada (como el lanzamiento de una campaña importante)

**Cómo evitarlo:** Si planeas aumentar tu volumen de envío, hazlo de forma gradual siguiendo un protocolo de warm-up (ver Capítulo 10). Un incremento del 20-30% diario es considerado aceptable. Cualquier incremento superior al 50% en un solo día debe ir acompañado de un monitoreo intensivo de las métricas de quejas y rebotes.

### 7.1.2 Alta Tasa de Rebotes (Hard Bounces)

Un hard bounce ocurre cuando el servidor receptor informa que la dirección de destino no existe o no puede recibir correo de forma permanente. Una tasa elevada de hard bounces es el indicador más claro de que estás enviando a direcciones que no han dado su consentimiento o que fueron recolectadas sin permiso.

**Los umbrales críticos son:**
- **Menos del 2%:** Considerado normal y aceptable. Ocurre incluso en las listas mejor mantenidas debido a cambios de dominio, cuentas eliminadas, o errores tipográficos.
- **Entre 2% y 5%:** Zona de advertencia. Debes investigar y limpiar tu lista de suscriptores problemáticos.
- **Entre 5% y 10%:** Zona de peligro. Tu IP está en riesgo inminente de ser listada. Muchos proveedores de servicios de email transaccional (como SendGrid o Mailgun) suspenden cuentas que superan este umbral.
- **Más del 10%:** Tu IP será listada casi con certeza. Este nivel de rebotes indica que estás enviando a listas compradas, raspadas o gravemente desactualizadas.

**Causas comunes de alta tasa de rebotes:**
- **Listas compradas o alquiladas:** Los vendedores de listas suelen inflar sus números con direcciones inválidas o generadas aleatoriamente.
- **Scraping de direcciones web:** Extraer direcciones de sitios web, foros o redes sociales sin consentimiento produce altas tasas de invalidez porque muchas direcciones son antiguas, están formateadas para evitar el scraping (como "usuario AT ejemplo PUNTO com"), o son honeypots.
- **Falta de verificación de suscripción (single opt-in):** Sin confirmación por email, los errores tipográficos y las direcciones falsas se cuelan en tu lista.
- **No limpiar direcciones inactivas:** Los suscriptores que no han abierto ningún correo en 6 meses tienen alta probabilidad de que su dirección haya sido abandonada o desactivada.

### 7.1.3 Configuraciones Incorrectas del Protocolo SMTP

**HELO/EHLO incorrecto:** El comando EHLO (o HELO) es el primer mensaje que envía tu servidor al conectarse a un servidor receptor. Debe incluir un FQDN (Fully Qualified Domain Name) válido que resuelva a la IP de tu servidor. Los errores más comunes son:

- Usar la dirección IP en lugar del nombre: `EHLO 203.0.113.50` (incorrecto)
- Usar un nombre que no resuelve en DNS: `EHLO mail.ejemplo.com` cuando `mail.ejemplo.com` no tiene registro A (incorrecto)
- Usar un nombre genérico como `EHLO localhost` o `EHLO unknown` (incorrecto)

**Falta de registro PTR (rDNS):** El registro PTR es la contraparte inversa del registro A: asocia una IP a un nombre de dominio. Los servidores receptores verifican que el PTR de tu IP coincida con el nombre que declaraste en EHLO. Si no hay PTR, o si el PTR no coincide con el EHLO, muchos servidores rechazarán la conexión o aplicarán penalizaciones de reputación.

```bash
# Verificar tu PTR
$ dig +short -x 203.0.113.50
mail.ejemplo.com.
```

> 📌 **Dato clave:** La configuración del PTR no la puedes hacer tú directamente; debes solicitarla a tu proveedor de hosting o ISP. Es uno de los pasos más olvidados y uno de los que más impacto tiene en la entregabilidad.

### 7.1.4 Conexiones Sin Cifrado (Falta de STARTTLS)

Aunque las listas negras no penalizan directamente las conexiones sin cifrar, los servidores receptores sí lo hacen. Enviar correo por el puerto 25 sin STARTTLS, o desde una IP que no soporta conexiones cifradas, es una señal de que el servidor está mal configurado o es antiguo. Muchos servidores (especialmente ProtonMail y Microsoft 365) penalizan o rechazan conexiones que no ofrecen STARTTLS.

## 7.2 Contenido del Mensaje

El contenido del mensaje es analizado por sistemas de filtrado como SpamAssassin, los filtros bayesianos de los proveedores, y los algoritmos de Machine Learning de Google y Microsoft. Ciertas características del contenido son fuertes indicadores de spam.

### 7.2.1 Líneas de Asunto Problemáticas

Los filtros de contenido analizan el asunto del correo en busca de patrones típicos de spam:
- **Texto completamente en mayúsculas:** "COMPRA AHORA!!!" es una señal clásica.
- **Uso excesivo de signos de puntuación:** "Gana dinero ya!!!!!!!!" o "Abre esto!!! urgente???"
- **Palabras y frases disparadoras:** "GRATIS", "GANE DINERO", "CLICK AQUÍ", "OFERTA POR TIEMPO LIMITADO", "NO MARQUE COMO SPAM".
- **Sujetos engañosos:** Usar "Re:" o "Fwd:" en correos que no son respuestas ni reenvíos. Esto engaña al receptor haciéndole pensar que el correo es parte de una conversación existente.
- **Promesas exageradas:** Cualquier asunto que prometa resultados extraordinarios ("Gane $10,000 por semana desde casa") será puntuado como spam casi con certeza.

### 7.2.2 Relación Texto/Imagen Problemática

Los spammers clásicos solían enviar correos consistentes en una única imagen grande que contenía todo el mensaje, porque los filtros de texto no podían analizar el contenido de las imágenes. Aunque los filtros modernos también analizan imágenes (con OCR y reconocimiento de patrones), una relación texto/imagen muy baja sigue siendo una señal de alerta:

- **Menos del 20% de texto:** El correo es principalmente una imagen. Esto es típico de spam visual.
- **Sin texto alternativo:** Las imágenes sin atributos `alt` descriptivos son sospechosas.
- **Texto invisible:** Usar CSS para hacer el texto del mismo color que el fondo (por ejemplo, `color: white; font-size: 1px`) es una técnica de ocultación que los filtros detectan y penalizan severamente.

### 7.2.3 Enlaces Sospechosos

La cantidad, calidad y presentación de los enlaces en un correo son analizadas en detalle:
- **Demasiados enlaces:** Un correo con más enlaces que texto es típicamente spam.
- **URLs acortadas sin contexto:** Enlaces a bit.ly, tinyurl, ow.ly sin indicar claramente el destino son sospechosos.
- **Enlaces a dominios recién registrados:** Los spammers suelen usar dominios nuevos (menos de 30 días) que rotan constantemente.
- **Texto del enlace engañoso:** Mostrar "https://banco-seguro.com" pero enlazar a "http://malicious-site.com" es una técnica clásica de phishing.
- **Enlaces a IPs en lugar de dominios:** Los enlaces a direcciones IP directamente en lugar de nombres de dominio son típicos de spam.

## 7.3 Problemas de Infraestructura

### 7.3.1 IP Residencial o Dinámica (PBL)

Spamhaus mantiene la Policy Block List (PBL) que incluye millones de direcciones IP que pertenecen a rangos asignados a ISP de acceso a internet residencial. La lógica es que estas IPs no deberían estar ejecutando servidores de correo SMTP directo. Si envías correo directamente desde tu conexión de fibra óptica doméstica, tu IP casi con certeza está en PBL.

**La solución no es solicitar desliste** (aunque Spamhaus permite solicitar la exclusión en algunos casos). La solución correcta es usar un relay SMTP autenticado proporcionado por tu ISP, o contratar un servicio de email transaccional como SendGrid, Mailgun o Amazon SES.

### 7.3.2 Falta de Configuración de Autenticación DNS

La ausencia o mala configuración de los registros de autenticación es una de las causas más comunes de problemas de entregabilidad:

- **Sin registro SPF:** Tu dominio es fácilmente falsificable (spoofing). Los servidores receptores no pueden verificar que tu servidor está autorizado para enviar correo en tu nombre.
- **SPF con `+all` o `?all`:** Básicamente le dices al mundo que cualquiera puede enviar correo desde tu dominio. Esto desactiva la protección.
- **Sin DKIM:** Tus correos no tienen firma digital, lo que los hace más fáciles de falsificar y menos confiables.
- **DMARC en p=none:** No estás aplicando ninguna política. Aunque es aceptable durante la fase de implementación, no debe ser permanente.

## 7.4 Prácticas de Marketing Cuestionables

### 7.4.1 Listas Compradas, Alquiladas o Raspadas

Esta es, sin duda, la forma más rápida de destruir la reputación de tu dominio y tus IPs. Las listas compradas:
- Contienen direcciones honeypot (trampas de spam) que garantizan tu listado en Spamhaus y otras listas.
- Tienen altísimas tasas de hard bounce (30-50% o más).
- Generan reportes de spam masivos porque los destinatarios no recuerdan haberse suscrito.
- Incluyen direcciones de personas que nunca dieron su consentimiento, violando leyes como GDPR, CAN-SPAM, CASL y LGPD.

### 7.4.2 Suscripción sin Confirmación (Single Opt-In)

El single opt-in (el usuario ingresa su email y queda suscrito inmediatamente) es mejor que nada, pero muy inferior al double opt-in (el usuario ingresa su email, recibe un correo de confirmación, y solo al hacer clic en el enlace queda suscrito). El double opt-in elimina:
- Direcciones mal escritas por errores tipográficos
- Suscripciones de bots y scripts automatizados
- Suscripciones con direcciones falsas o temporales
- La gran mayoría de reportes de spam por "no haber solicitado esto"

### 7.4.3 Ausencia de Enlace de Baja Visible y Funcional

Cada correo que envíes debe incluir un enlace de baja (unsubscribe) claramente visible, funcionando, y que procese la baja inmediatamente (sin pedir inicio de sesión, sin confirmación adicional). Los servidores receptores verifican periódicamente la presencia y funcionalidad del enlace de baja. Su ausencia:
- Es ilegal en prácticamente todas las jurisdicciones (CAN-SPAM, GDPR, CASL, LGPD).
- Es una señal de spam para los filtros de contenido.
- Obliga a los usuarios frustrados a usar el botón "Reportar spam", que es mucho más dañino para tu reputación que una baja voluntaria.

## 7.5 Seguridad del Servidor Comprometida

### 7.5.1 Servidor Infectado o Comprometido

Un servidor con vulnerabilidades de seguridad puede ser utilizado para enviar spam sin que el administrador lo sepa. Los atacantes instalan scripts PHP, CGI, o incluso modifican el propio MTA para enviar correos masivos. Las señales de un servidor comprometido incluyen:

- **Colas de correo inusualmente largas:** Revisa `mailq` regularmente. Si ves miles de mensajes que no reconoces, tu servidor puede estar siendo usado como relay.
- **Tráfico SMTP saliente en horas sin actividad:** Si tu servidor envía correos a las 3:00 AM cuando no hay actividad humana programada, investiga.
- **Archivos PHP sospechosos:** Los scripts de spam suelen ocultarse en `/tmp/`, `/var/tmp/`, o en directorios web con nombres aleatorios.
- **Usuarios del sistema no autorizados:** Revisa `/etc/passwd` y `/etc/shadow` en busca de cuentas que no creaste.

### 7.5.2 Formularios Web Explotados

Los formularios de contacto, registro y comentarios son vectores comunes de ataque. Un atacante puede usar tu formulario para enviar miles de correos, y como salen desde la IP de tu servidor, tú terminas listado.

**Medidas de protección esenciales:**
- Captcha en todos los formularios públicos (reCAPTCHA v3 o hCaptcha).
- Rate limiting por IP (máximo 3-5 envíos por hora desde la misma IP).
- Campos honeypot ocultos (campos invisibles que los humanos no ven pero los bots sí completan).
- Validación CSRF en todos los formularios.
- Notificaciones automáticas cuando el volumen de envío desde formularios supere un umbral.
