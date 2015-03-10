# Capítulo 15: Glosario Completo de Términos

[← Anterior](14-estudios-de-caso.md) | [Índice](README.md)

---

Este glosario contiene los términos técnicos más importantes relacionados con listas negras de correo electrónico, autenticación, protocolos y entregabilidad. Los términos están ordenados alfabéticamente para consulta rápida.

---

## A

**Alignment (Alineación DMARC)**
En el contexto de DMARC, la alineación se refiere a la condición en la que el dominio presente en la cabecera `From:` del correo coincide con el dominio autenticado por SPF (a través del `MAIL FROM`) o por DKIM (a través de la firma `d=`). DMARC permite dos modos de alineación: relajada (`r`), donde el dominio de `From:` puede ser un subdominio del dominio autenticado; y estricta (`s`), donde deben coincidir exactamente.

**ARC (Authenticated Received Chain)**
Estándar definido en los RFC 8615, 8616 y 8617 que permite preservar los resultados de autenticación (SPF, DKIM) a través de servicios de reenvío de correo legítimos, como listas de correo o sistemas de forwarding automático. Sin ARC, cuando un correo es reenviado, las verificaciones SPF y DKIM pueden fallar porque el servidor que reenvía no está autorizado. ARC añade una cadena de sellos criptográficos que documentan el estado de autenticación en cada salto.

**ASN (Autonomous System Number)**
Número único que identifica a una red o conjunto de redes bajo una administración común en internet. Los ASN son asignados por los Registros Regionales de Internet (RIR). Algunas listas negras agresivas (como UCEPROTECT Level 2 y 3) listan ASN completos, lo que significa que todas las IPs de ese ASN son bloqueadas independientemente del comportamiento individual de cada servidor.

**Autenticación de Correo**
Conjunto de estándares (SPF, DKIM, DMARC) que permiten verificar que un correo electrónico fue enviado desde un servidor autorizado por el dominio del remitente y que su contenido no fue alterado durante el tránsito.

---

## B

**Backscatter**
Correos de no entrega (NDR — Non-Delivery Report) que son enviados a direcciones falsas. Ocurre cuando un servidor de correo acepta un mensaje (porque no verifica la existencia del remitente) y luego intenta enviar un NDR al remitente, que resulta ser una dirección falsa. El backscatter es un problema porque satura los servidores y puede hacer que el servidor emisor sea marcado como fuente de spam.

**BIMI (Brand Indicators for Message Identification)**
Estándar emergente (RFC 8601 en desarrollo) que permite a las marcas mostrar su logotipo oficial junto a sus correos electrónicos en los buzones de los proveedores que lo soportan. Actualmente, Gmail y Apple Mail son los principales adoptantes de BIMI. Requiere DMARC configurado en `p=quarantine` o `p=reject`. Además del beneficio de marca, BIMI mejora la entregabilidad porque obliga a tener una configuración de autenticación sólida.

**Bounce (Rebote)**
Notificación generada por el servidor receptor cuando no puede entregar un mensaje. Se clasifican en:
- **Hard Bounce:** Error permanente (la dirección no existe, el dominio no acepta correo). No debe reintentarse.
- **Soft Bounce:** Error temporal (el buzón está lleno, el servidor está sobrecargado). Puede reintentarse.

---

## C

**CAN-SPAM Act**
Ley federal de Estados Unidos (Controlling the Assault of Non-Solicited Pornography And Marketing Act de 2003) que establece los requisitos para el correo comercial. Exige: identificación clara del remitente, dirección física válida, opción de baja funcional, asuntos no engañosos, y etiquetado de contenido para adultos. Las violaciones pueden resultar en multas de hasta $43,792 por correo.

**CASL (Canadian Anti-Spam Legislation)**
Ley antispam canadiense, vigente desde 2014, considerada una de las más estrictas del mundo. Requiere consentimiento explícito (opt-in) antes de enviar correos comerciales, y establece requisitos de identificación y mecanismos de baja muy detallados. Las multas pueden alcanzar los 10 millones de dólares canadienses.

**CIDR (Classless Inter-Domain Routing)**
Notación para especificar rangos de direcciones IP. Se escribe como una dirección IP seguida de una barra y un número (ej: `203.0.113.0/24`). El número indica cuántos bits de la máscara de red son fijos. `/24` equivale a 256 direcciones, `/16` a 65,536. Algunas listas negras listan rangos CIDR completos.

**Content Filter**
Software o sistema que analiza el contenido del correo electrónico (cabeceras, cuerpo, adjuntos) para determinar si es spam, phishing o contiene malware. Los content filters pueden ser basados en reglas (SpamAssassin), basados en firmas (ClamAV), o basados en Machine Learning (filtros de Gmail, SmartScreen de Microsoft).

---

## D

**DBL (Domain Block List)**
Lista negra de dominios operada por Spamhaus. A diferencia de la SBL y XBL, que listan IPs, la DBL lista nombres de dominio que han sido encontrados en el cuerpo de correos spam (como enlaces, direcciones de respuesta, o dominios en la cabecera `From:`). Zona de consulta: `dbl.spamhaus.org`.

**DKIM (DomainKeys Identified Mail)**
Estándar de autenticación (RFC 6376) que permite firmar digitalmente los correos electrónicos. El remitente firma el mensaje con una clave privada, y el receptor verifica la firma usando una clave pública publicada en el DNS del dominio del remitente. DKIM garantiza que el mensaje no fue alterado durante el tránsito y que fue enviado por el propietario legítimo del dominio.

**DMARC (Domain-based Message Authentication, Reporting & Conformance)**
Estándar de autenticación (RFC 7489) que unifica SPF y DKIM. DMARC permite al propietario de un dominio publicar una política que indica a los servidores receptores qué hacer cuando un correo falla las verificaciones SPF y DKIM. Las políticas posibles son: `none` (solo monitorear), `quarantine` (marcar como spam), o `reject` (rechazar). Además, DMARC proporciona reportes de retroalimentación que permiten al remitente ver quién está enviando correo en su nombre.

**DNS (Domain Name System)**
Sistema distribuido que traduce nombres de dominio legibles por humanos (como `ejemplo.com`) a direcciones IP numéricas (como `203.0.113.50`). Es fundamental para el funcionamiento del correo electrónico: los registros MX determinan qué servidores reciben correo, los registros PTR verifican la identidad inversa de las IPs, y los registros TXT almacenan SPF, DKIM y DMARC.

**DNSBL (DNS-based Blackhole List)**
Lista negra de direcciones IP o dominios que puede ser consultada mediante el protocolo DNS. Es la arquitectura más común para listas negras de correo electrónico. La consulta se realiza invirtiendo los octetos de la IP, concatenando con el dominio de la lista, y realizando una consulta DNS tipo A. Si hay respuesta, la IP está listada.

**Double Opt-In**
Proceso de suscripción a una lista de correo que requiere dos pasos: (1) el usuario ingresa su dirección en un formulario, y (2) recibe un correo de confirmación con un enlace que debe hacer clic para activar la suscripción. Es el estándar de oro para la gestión de listas porque verifica que la dirección es válida y que el usuario realmente desea recibir los correos.

---

## E

**EHLO (Extended HELO)**
Comando del protocolo SMTP que reemplaza a HELO cuando el servidor soporta extensiones SMTP (ESMTP). Es el primer comando que envía un servidor al conectar con otro. Debe ir seguido del FQDN (Fully Qualified Domain Name) del servidor emisor, que debe coincidir con su registro PTR.

**Envelope (Sobre SMTP)**
Información de transporte del correo, que incluye la dirección del remitente (`MAIL FROM` o return-path) y la del destinatario (`RCPT TO`). Esta información no es visible para el usuario final. Las verificaciones SPF se realizan sobre el dominio del `MAIL FROM`, NO sobre el `From:` visible.

---

## F

**FBL (Feedback Loop)**
Servicio ofrecido por los principales proveedores de correo (Google, Microsoft, Yahoo) que notifica a los remitentes cuando un usuario marca sus correos como spam. Configurar FBL permite detectar problemas de quejas antes de que resulten en listados en listas negras o bloqueos de cuenta.

**FQDN (Fully Qualified Name Domain)**
Nombre de dominio completo que incluye todas las partes de la jerarquía DNS, incluyendo el punto final. Ejemplo: `mail.ejemplo.com.` (con el punto final). Los servidores de correo deben usar un FQDN válido en su comando EHLO, y ese FQDN debe tener un registro A que resuelva a la IP del servidor.

**False Positive (Falso Positivo)**
Situación en la que un sistema de filtrado clasifica incorrectamente un correo legítimo como spam, o una lista negra incluye a un remitente que no debería estar listado. Los falsos positivos son el principal problema de las listas negras y los filtros antispam demasiado agresivos.

---

## G

**GDPR (General Data Protection Regulation)**
Reglamento General de Protección de Datos de la Unión Europea, vigente desde mayo de 2018. Establece requisitos estrictos para el tratamiento de datos personales, incluyendo el consentimiento explícito para el envío de correos comerciales, el derecho al olvido (baja), y la obligación de notificar violaciones de datos.

**Greylisting**
Técnica antispam que consiste en rechazar temporalmente (código 450) un correo desconocido. Los servidores de correo legítimos reintentarán el envío después de un tiempo (de 15 minutos a 4 horas), mientras que los spammers típicamente no lo hacen. Es efectivo contra el spam oportunista pero puede retrasar la entrega de correos legítimos.

---

## H

**HELO**
Comando original del protocolo SMTP (anterior a ESMTP) utilizado para que el servidor emisor se identifique ante el receptor. Reemplazado por EHLO en servidores modernos. El comando HELO va seguido del nombre del servidor emisor. Ejemplo: `HELO mail.ejemplo.com`.

**Honeypot (Spamtrap)**
Dirección de correo electrónico falsa, creada específicamente para detectar spammers. Nunca se utiliza para comunicación legítima ni se publica en lugares donde usuarios reales puedan encontrarla. Cualquier correo que llegue a una dirección honeypot es, por definición, spam. Las organizaciones antispam como Spamhaus mantienen miles de honeypots para detectar y listar fuentes de spam.

**Hard Bounce**
Rebote permanente. El servidor receptor informa que la dirección del destinatario no existe, está desactivada, o el dominio no acepta correo. Las direcciones que generan hard bounces deben ser eliminadas inmediatamente de la lista de envío.

**HELO/EHLO (inspección de)**
Verificación que realizan los servidores receptores sobre el nombre proporcionado en el comando HELO/EHLO. Un HELO válido debe ser un FQDN que resuelva (tenga registro A o AAAA) a la IP que está realizando la conexión. Las IPs con HELO inválido o inexistente son frecuentemente penalizadas o rechazadas.

---

## L

**LGPD (Lei Geral de Proteção de Dados)**
Ley General de Protección de Datos de Brasil (Ley 13.709/2018), similar al GDPR europeo. Establece requisitos para el tratamiento de datos personales, incluyendo el consentimiento para comunicaciones comerciales y el derecho de los titulares a solicitar la eliminación de sus datos.

**Listwashing**
Práctica ilegal y poco ética que consiste en "limpiar" una lista de direcciones de correo compradas o raspadas verificando cuáles son válidas. Los listwashers envían un correo de prueba (a menudo camuflado como un mensaje legítimo) a todas las direcciones, y las que no rebotan son consideradas "válidas" y vendidas como "lista limpia". Esta práctica es especialmente dañina porque genera tráfico de verificación que las organizaciones antispam monitorean.

---

## M

**MTA (Mail Transfer Agent)**
Software que transfiere correos electrónicos entre servidores SMTP. Los MTA más comunes son Postfix (el más popular en Linux, conocido por su seguridad y flexibilidad), Exim (el MTA predeterminado en cPanel, también muy utilizado), y Sendmail (el histórico, aunque cada vez menos común por su complejidad de configuración y problemas de seguridad históricos).

**MSA (Mail Submission Agent)**
Software que recibe correos desde clientes de correo (MUAs) y los transfiere al MTA para su entrega. Opera típicamente en el puerto 587 con STARTTLS y requiere autenticación SMTP. La distinción entre MSA y MTA es importante porque los puertos y las reglas de seguridad son diferentes.

**MUA (Mail User Agent)**
Cliente de correo utilizado por el usuario final para leer, redactar y gestionar sus mensajes. Ejemplos: Microsoft Outlook, Mozilla Thunderbird, Apple Mail, Gmail interfaz web, y aplicaciones móviles como Spark o Edison Mail.

**MX (Mail eXchange)**
Registro DNS que especifica qué servidores están autorizados para recibir correo en nombre de un dominio. Cada registro MX tiene dos campos: la prioridad (un número, donde el valor más bajo tiene la máxima prioridad) y el nombre del servidor. Ejemplo: `ejemplo.com. IN MX 10 mail.ejemplo.com.`

---

## O

**Open Relay**
Servidor SMTP mal configurado que permite a cualquier persona (autorizada o no) enviar correo a través de él sin autenticación. Los open relays son el peor enemigo de la seguridad del correo: son rápidamente detectados por los spammers, utilizados masivamente para enviar correo no deseado, y casi garantizan la inclusión de la IP en todas las listas negras importantes.

**Opt-In**
Permiso explícito otorgado por un usuario para recibir correos electrónicos de un remitente específico. Puede ser simple (single opt-in: el usuario ingresa su dirección y queda suscrito inmediatamente) o doble (double opt-in: requiere confirmación mediante un enlace enviado por correo).

---

## P

**PBL (Policy Block List)**
Lista negra operada por Spamhaus que contiene direcciones IP que, según la política de Spamhaus, no deberían estar enviando correo SMTP directamente. Incluye la mayoría de las IPs de ISP residenciales (conexiones domésticas de fibra, ADSL, cable módem, LTE/4G/5G) y rangos de IPs dinámicas. No implica que hayas hecho algo malo, sino que tu IP no es adecuada para envío directo de correo.

**PTR (Pointer Record)**
Registro DNS inverso que asocia una dirección IP a un nombre de dominio. Es lo opuesto al registro A (que asocia un dominio a una IP). Ejemplo: la IP `203.0.113.50` tiene un PTR que apunta a `mail.ejemplo.com`. Los servidores receptores verifican el PTR de la IP remitente y lo comparan con el nombre declarado en el comando EHLO.

**Phishing**
Técnica de ingeniería social que busca obtener información confidencial (contraseñas, números de tarjeta de crédito, datos bancarios) haciéndose pasar por una entidad de confianza a través de correo electrónico. El phishing es una de las principales amenazas que las listas negras y los filtros antispam intentan combatir.

**Postmaster**
Dirección de correo estándar (postmaster@dominio.com) que todo dominio que maneje correo debe tener habilitada. Es la dirección de contacto para problemas relacionados con el correo, y es utilizada por las organizaciones antispam y los proveedores para comunicarse con los administradores del dominio.

---

## Q

**Queue (Cola de Correo)**
Área de almacenamiento temporal donde el MTA mantiene los mensajes que están pendientes de entrega. Las colas de correo se monitorizan para detectar acumulaciones anómalas que podrían indicar un servidor comprometido o un problema de configuración.

**Quarantine (Cuarentena)**
Acción de DMARC (política `p=quarantine`) que indica al servidor receptor que debe tratar los correos que fallan la autenticación como sospechosos, típicamente dirigiéndolos a la carpeta de spam del destinatario. Es una opción intermedia entre `none` (no hacer nada) y `reject` (rechazar completamente).

---

## R

**RBL (Realtime Blackhole List)**
Sinónimo de DNSBL. El término "Realtime" enfatiza que la lista puede ser consultada en tiempo real (a diferencia de las listas estáticas que se descargan periódicamente). "Blackhole" se refiere a que los correos de las IPs listadas son "tragados por un agujero negro": desaparecen sin llegar al destinatario.

**rDNS (Reverse DNS)**
Ver PTR. El rDNS es la funcionalidad del DNS que permite, dada una dirección IP, obtener el nombre de dominio asociado.

**Return-Path**
Cabecera del mensaje que contiene la dirección de correo a la que deben enviarse los rebotes (NDR). Normalmente coincide con el `MAIL FROM` del sobre SMTP, pero puede ser modificada por servicios de email transaccional para tracking de rebotes. Los servidores receptores pueden verificar que el Return-Path coincida con un dominio con SPF configurado.

**Rate Limiting**
Técnica que limita la velocidad o cantidad de operaciones que un cliente puede realizar en un período de tiempo. En el contexto del correo, se aplica para limitar cuántos mensajes puede enviar una IP por minuto/hora, evitando que un servidor comprometido pueda enviar spam masivo.

---

## S

**SBL (Spamhaus Block List)**
Lista principal de Spamhaus. Contiene direcciones IP que han sido identificadas como fuentes de spam directo, confirmado por analistas humanos de Spamhaus. Es la lista más selectiva y fiable de Spamhaus. Una IP en SBL requiere atención inmediata y un proceso de desliste manual.

**SMTP (Simple Mail Transfer Protocol)**
Protocolo estándar para la transferencia de correo electrónico en internet. Definido originalmente en RFC 821 (1982), actualizado en RFC 5321 (2008). Utiliza comandos en texto plano (EHLO, MAIL FROM, RCPT TO, DATA) y códigos de respuesta numéricos. Opera en el puerto 25 para transferencia entre servidores y en el puerto 587 (con STARTTLS) para envío desde clientes.

**SNDS (Smart Network Data Services)**
Programa de Microsoft que permite a los remitentes de correo consultar su reputación en los servicios de Outlook.com y Office 365. Proporciona métricas diarias de volumen, tasa de quejas, y estado de la IP. Es la herramienta más importante para monitorear la entregabilidad en el ecosistema Microsoft.

**SPF (Sender Policy Framework)**
Estándar de autenticación (RFC 7208) que permite al propietario de un dominio publicar una lista de servidores autorizados para enviar correo en su nombre. Se implementa mediante un registro TXT en el DNS del dominio. Los servidores receptores verifican que la IP del servidor remitente esté en la lista publicada por el dominio del `MAIL FROM`.

**Snowshoe Spam**
Técnica de spam que utiliza muchas direcciones IP (a veces cientos o miles) para enviar un volumen bajo de correos desde cada una, con el objetivo de evadir las listas negras que detectan por volumen. Spamhaus asigna el código `127.0.0.3` en Zen para IPs que muestran este patrón.

**Soft Bounce**
Rebote temporal. El servidor receptor informa que no puede entregar el mensaje en este momento, pero podría hacerlo más tarde. Causas típicas: buzón lleno, servidor sobrecargado, mensaje demasiado grande. Los soft bounces deben reintentarse con backoff exponencial.

---

## T

**TLD (Top-Level Domain)**
Nivel más alto en la jerarquía del DNS. Los TLDs se clasifican en genéricos (gTLD: .com, .org, .net), de código de país (ccTLD: .es, .mx, .ar), y patrocinados (sTLD: .edu, .gov). Algunos TLDs, especialmente los nuevos como .xyz, .top, .loan, tienen una alta concentración de spam y son tratados con especial sospecha por los filtros antispam.

**TLS (Transport Layer Security)**
Protocolo criptográfico que proporciona seguridad en las comunicaciones a través de internet. En el contexto del correo, STARTTLS permite actualizar una conexión SMTP no cifrada a una cifrada. Los servidores receptores modernos requieren o prefieren conexiones cifradas.

**TTL (Time To Live)**
Valor en segundos que indica cuánto tiempo un registro DNS puede ser almacenado en caché antes de ser descartado. Las respuestas de DNSBL suelen tener TTL bajos (300-3600 segundos) para permitir actualizaciones rápidas cuando cambia el estado de una IP.

**Transactional Email**
Correos electrónicos que se envían como resultado de una acción del usuario, como confirmaciones de registro, notificaciones de compra, recuperación de contraseñas, estados de envío, y notificaciones de cuenta. Se consideran críticos para el negocio y deben tener la máxima prioridad de entregabilidad.

---

## W

**Warm-up**
Proceso gradual de aumento del volumen de correo enviado desde una IP nueva o recién deslistada, permitiendo que los proveedores de correo construyan una reputación positiva basada en el comportamiento de envío. Un warm-up típico comienza con 50 correos al día y aumenta gradualmente durante 4 a 8 semanas hasta alcanzar el volumen objetivo.

**Whitelist (Lista Blanca)**
Base de datos de remitentes considerados de confianza. A diferencia de las listas negras, que bloquean, las listas blancas garantizan la entrega, saltándose la mayoría de los filtros antispam. La lista blanca más prestigiosa es la Spamhaus Whitelist (SWL).

**WHOIS**
Base de datos pública que contiene información de registro de nombres de dominio y asignación de direcciones IP. Los datos WHOIS incluyen el nombre del propietario, fecha de registro, fecha de expiración, y datos de contacto. Los dominios recién registrados o con datos WHOIS ocultos suelen ser tratados con mayor sospecha por los filtros antispam.

---

## X

**XBL (Exploits Block List)**
Lista negra operada por Spamhaus que contiene direcciones IP que muestran evidencia de estar infectadas por malware, operar redes de bots (botnets), proxies abiertos, o servidores SOCKS comprometidos. A diferencia de la SBL (spam directo), la XBL se enfoca en el comportamiento técnico comprometido.

**X-Headers (Cabeceras Extendidas)**
Cabeceras de correo que no están definidas en los estándares RFC pero que los servidores y filtros añaden para proporcionar información adicional. Ejemplos: `X-Spam-Status` (resultado del filtro antispam), `X-Mailer` (software utilizado para enviar), `X-Originating-IP` (IP del remitente original). Aunque no son estándar, son ampliamente utilizadas para diagnóstico.

---

## Z

**Zen**
Lista compuesta de Spamhaus (zona `zen.spamhaus.org`) que integra las listas SBL, XBL y PBL en una sola consulta DNS. Desde su lanzamiento en 2006, Zen simplificó drásticamente la configuración de los servidores de correo. Una sola consulta a Zen reemplaza hasta tres consultas separadas. La IP de retorno (127.0.0.x) indica qué sub-lista específica contiene la entrada y por lo tanto la razón del listado.
