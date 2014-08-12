# Capítulo 10: Estrategias para Salir de una Lista Negra

[← Anterior](09-spf-dkim-dmarc.md) | [Índice](README.md) | [Siguiente →](11-prevencion.md)

---

## 10.1 Diagnóstico Inicial: Entender el Problema Antes de Actuar

Antes de solicitar el desliste de cualquier lista negra, es imperativo realizar un diagnóstico completo y entender las causas que originaron el listado. Saltarse este paso es la causa más común de fracaso en los procesos de desliste: muchas personas solicitan ser removidas sin haber corregido el problema subyacente, y la lista simplemente los vuelve a incluir horas o días después, a veces con períodos de exclusión más largos.

El proceso de diagnóstico debe seguir cuatro fases en orden estricto:

**Fase 1: Identificar qué lista o listas te incluyeron.** Utiliza herramientas como MXToolbox (https://mxtoolbox.com/blacklists.aspx) que consulta más de 100 listas simultáneamente, o el verificador oficial de Spamhaus en https://check.spamhaus.org/. No confíes en una sola fuente; verifica contra múltiples herramientas porque algunas listas solo son consultadas por ciertos proveedores de correo. Anota cada lista donde aparezcas, el código de retorno exacto (por ejemplo, 127.0.0.2, 127.0.0.4, 127.0.0.6) y la fecha aproximada en que comenzó el bloqueo según tus registros.

**Fase 2: Entender el motivo del listado.** Cada código de retorno en Spamhaus Zen tiene un significado preciso: 127.0.0.2 indica spam directo confirmado por analistas humanos (SBL); 127.0.0.4 indica actividad de malware, exploits o proxies abiertos (XBL); 127.0.0.6 señala que tu IP está en un rango que Spamhaus considera como no apto para envío directo de correo (PBL). En SpamCop, el simple hecho de estar listado indica que usuarios reportaron tus correos como spam en cantidad suficiente. En Barracuda, el listado responde a comportamiento abusivo detectado por sus sensores a nivel global. Revisa los logs de tu servidor de correo para encontrar los mensajes de rechazo exactos: busca en `/var/log/mail.log` líneas que contengan `rejected` junto al nombre de la lista negra. Cada mensaje de error SMTP incluye el código de respuesta y usualmente un enlace a la página de información de la lista.

**Fase 3: Corregir la causa raíz.** Esta es la fase más importante y la que requiere más trabajo. Si fuiste listado por tener una IP en un rango residencial (PBL), la solución es técnica: debes dejar de enviar correo directamente desde esa IP y utilizar un relay SMTP autenticado. Si fuiste listado por spam directo (SBL), necesitas investigar qué está generando ese spam: ¿tu servidor fue comprometido? ¿compraste una lista de correos? ¿tus formularios web están siendo explotados? ¿tienes una alta tasa de quejas de usuarios? Cada causa requiere una acción correctiva diferente. Haz una lista de todas las medidas que tomaste y guárdala, porque la mayoría de los formularios de desliste te pedirán que expliques qué acciones correctivas implementaste.

**Fase 4: Documentar cada medida tomada.** Crea un registro escrito con fechas, horas, comandos ejecutados, cambios de configuración y resultados. Esta documentación servirá para los formularios de desliste (que casi siempre preguntan "¿Qué ha hecho para resolver el problema?"), para el análisis posterior si el problema se repite, y para justificar ante tu jefe o clientes las acciones realizadas. Una buena documentación incluye capturas de pantalla de logs, resultados de comandos de verificación, y el antes/después de configuraciones críticas como SPF, DKIM y DMARC.

## 10.2 Spamhaus: El Proceso de Desliste Paso a Paso

Spamhaus es, con diferencia, la lista negra más importante del mundo. Estar listado en Spamhaus significa que aproximadamente el 80% de los servidores de correo que consultan DNSBL te están rechazando. Por suerte, Spamhaus ofrece un proceso de desliste gratuito, bien documentado y generalmente eficiente, siempre que hayas corregido la causa real del problema.

### 10.2.1 Desliste de SBL (Spamhaus Block List)

La SBL es la lista principal de Spamhaus e incluye direcciones IP que han sido identificadas como fuentes de spam directo, confirmado por analistas humanos de Spamhaus o mediante sistemas automatizados de detección. Ser incluido en SBL es la situación más grave, porque implica que Spamhaus tiene evidencia concreta de que desde tu IP se está enviando correo no solicitado.

**Paso 1: Confirmar el tipo de listado.** Ejecuta el siguiente comando y observa el código de retorno:

```bash
dig +short 50.113.0.203.zen.spamhaus.org
```

Si obtienes `127.0.0.2`, estás en SBL por spam directo. Si obtienes `127.0.0.3`, estás en SBL por snowshoe spam (una técnica donde se utilizan muchas IPs para enviar poco volumen desde cada una, evadiendo detección por volumen). Ambos casos requieren acción inmediata.

**Paso 2: Investigar la causa del listado.** Accede a https://check.spamhaus.org/ e ingresa tu IP. La página te mostrará detalles específicos del listado, incluyendo una descripción de por qué fuiste incluido y, en muchos casos, ejemplos de los correos que activaron la detección. Lee esta información con atención, porque contiene pistas críticas sobre qué estás haciendo mal. Las causas más comunes incluyen: un servidor comprometido que está siendo utilizado como relay de spam sin tu conocimiento (revisa las colas de correo con `mailq`), la compra de listas de correo de terceros que contienen direcciones trampa (honeypots) de Spamhaus, o una alta tasa de quejas de usuarios que marcan tus newsletters como spam porque no recuerdan haberse suscrito.

**Paso 3: Ejecutar las acciones correctivas.** Dependiendo de la causa identificada, tus acciones serán diferentes:

- **Si tu servidor fue comprometido:** desconéctalo de la red inmediatamente para detener el envío de spam. Realiza una auditoría completa de seguridad: actualiza todo el software (Postfix, Exim, Apache, PHP, MySQL), cambia todas las contraseñas (root, usuarios del sistema, FTP, SSH, bases de datos, paneles de control), busca archivos sospechosos en `/tmp/`, `/var/tmp/`, y directorios web con `find / -mtime -7 -type f -name "*.php" -o -name "*.pl" -o -name "*.cgi"`, revisa los logs de acceso en busca de patrones anómalos, instala un firewall como CSF (ConfigServer Security & Firewall) y configura alertas de login. Una vez que estés seguro de que el servidor está limpio, mantenlo desconectado hasta completar el desliste.

- **Si compraste listas de correo:** elimínalas inmediatamente y destrúyelas. No hay forma segura de usar listas compradas; siempre incluyen direcciones trampa que te mantendrán en listas negras. Implementa un sistema de double opt-in para todas las suscripciones futuras.

- **Si la causa es alta tasa de quejas:** revisa tus campañas de correo. ¿Estás enviando con demasiada frecuencia? ¿El contenido es relevante para tus suscriptores? ¿El enlace de baja es visible y funciona correctamente? Implementa segmentación por engagement y reduce la frecuencia para los segmentos menos activos.

**Paso 4: Solicitar el desliste.** Una vez que hayas corregido la causa raíz y hayas verificado que no se está enviando más spam desde tu IP (revisa la cola con `mailq` y asegúrate de que esté vacía o con solo mensajes legítimos), ve a https://www.spamhaus.org/lookup/, ingresa tu IP y haz clic en "Request Deletion" si el enlace está disponible. Completa el formulario con una explicación clara y honesta de las medidas correctivas que tomaste. Sé específico: no digas simplemente "ya limpiamos el servidor", explica qué encontraste, qué comandos ejecutaste, qué configuraciones cambiaste. Los analistas de Spamhaus procesan cientos de solicitudes al día; una explicación detallada y verosímil acelera tu caso. El tiempo de respuesta típico es de 24 a 48 horas hábiles.

### 10.2.2 Desliste de PBL (Policy Block List)

La PBL es diferente de la SBL: no implica que hayas hecho algo malo, sino que tu dirección IP pertenece a un rango que Spamhaus considera inapropiado para el envío directo de correo electrónico. Esto incluye prácticamente todas las IPs de ISP residenciales (como las conexiones de fibra óptica doméstica, ADSL, cable módem) y rangos de IPs dinámicas.

El código de retorno `127.0.0.6` indica específicamente que estás en PBL por ser una IP residencial o dinámica. No hay una solución única para todos los casos, pero las opciones son las siguientes, ordenadas de mejor a peor:

**Opción A (Altamente Recomendada): Usar un relay SMTP autenticado.** Esta es la solución profesional. Configura tu servidor de correo para que todo el correo saliente se entregue a un servicio de relay como SendGrid, Mailgun, Amazon SES, o el relay de tu propio ISP (muchos proveedores ofrecen un servidor SMTP autenticado para sus clientes de hosting). En Postfix, esto se configura en `/etc/postfix/main.cf` con las directivas `relayhost = [smtp.sendgrid.net]:587` y configurando SASL para la autenticación. De esta forma, tu servidor local solo procesa la recepción y el filtrado, mientras que la entrega sale desde IPs con buena reputación gestionadas por el proveedor de relay.

**Opción B: Solicitar a Spamhaus la exclusión de tu IP.** Spamhaus permite que los propietarios de IPs soliciten la exclusión de la PBL si pueden demostrar que la IP es estática, está asignada a un servidor legítimo y cumple con todas las prácticas de envío. Para ello, ve a https://www.spamhaus.org/pbl/ y sigue el proceso de "Remove an IP from the PBL". Necesitarás demostrar que tienes control sobre la IP y que estás cumpliendo con las políticas de envío. Este proceso es gratuito pero no siempre es exitoso, especialmente si tu IP está en un rango claramente residencial.

**Opción C: Contratar un servidor dedicado o VPS con IP limpia.** Si insistes en enviar correo directo desde tu propia infraestructura, necesitas una IP que no esté en rangos PBL. Los proveedores de hosting como DigitalOcean, Linode, Vultr o Hetzner ofrecen IPs que no están en PBL, pero debes verificar antes de contratar. Ten en cuenta que incluso con una IP limpia, si no configuras correctamente SPF, DKIM, DMARC y PTR, tendrás problemas de entregabilidad. Además, las IPs de cloud público (AWS, Google Cloud, Azure) a menudo tienen restricciones adicionales y pueden terminar en listas negras por el comportamiento de otros usuarios en el mismo rango.

### 10.2.3 Desliste de DBL (Domain Block List)

La DBL lista dominios, no IPs. Si tu dominio aparece en `dbl.spamhaus.org`, significa que tu nombre de dominio ha sido encontrado en el cuerpo de correos no solicitados (como enlaces o direcciones de respuesta). Esto es particularmente grave porque afecta a todos los correos que envíes desde cualquier IP.

Para verificar, ejecuta:

```bash
dig +short ejemplo.com.dbl.spamhaus.org
```

Si obtienes una respuesta, tu dominio está listado. Las acciones correctivas dependen de por qué tu dominio aparece en correos spam: si es phishing, debes resolver la vulnerabilidad de seguridad inmediatamente; si son enlaces en newsletters no solicitadas, debes detener el envío a listas no consentidas; si alguien está falsificando tu dominio, debes implementar DMARC con política `p=reject` para evitar que tus correos legítimos sean marcados como spam. El desliste se solicita a través del mismo formulario de Spamhaus, explicando las medidas correctivas tomadas.

## 10.3 SpamCop: Cómo Manejar un Listado por Reportes de Usuarios

SpamCop funciona de manera fundamentalmente diferente a Spamhaus: no tiene analistas humanos revisando evidencias, sino que lista automáticamente cualquier IP que reciba suficientes reportes de usuarios en un período de tiempo determinado. Esto lo hace más sensible pero también más propenso a falsos positivos.

**El mecanismo de SpamCop:** Cuando un usuario recibe un correo que considera no deseado, puede reenviarlo a SpamCop (o usar su plugin en clientes de correo). SpamCop analiza el correo, extrae la IP del remitente desde las cabeceras, y si esa IP acumula varios reportes en poco tiempo, es listada automáticamente en `bl.spamcop.net`. El umbral exacto de reportes no es público, pero se sabe que es relativamente bajo.

**Desliste automático:** La buena noticia es que SpamCop tiene un mecanismo de desliste automático. Si la IP deja de recibir reportes nuevos, el listado expira por sí solo en un período que generalmente oscila entre 24 y 48 horas. Esto significa que si el problema que causó los reportes ya no existe (por ejemplo, detuviste una campaña de mailing masivo que estaba generando quejas), lo más sensato es simplemente esperar. No hay necesidad de solicitar desliste manual a menos que la IP lleve varios días limpia pero siga listada.

**Desliste manual:** Si el automático no funciona o necesitas una solución más rápida, puedes solicitar el desliste manual en https://www.spamcop.net/fom-serve/cache/329.html. El formulario te pedirá tu dirección de correo y una explicación de por qué crees que el listado es incorrecto o ya no es aplicable. El equipo de SpamCop revisa estas solicitudes manualmente, pero el proceso puede tardar de 24 a 72 horas.

**Prevención de futuros reportes:** La mejor estrategia con SpamCop es evitar que los usuarios marquen tus correos como spam en primer lugar. Esto significa: usar double opt-in para confirmar suscripciones, incluir un enlace de baja visible y funcional en cada correo, segmentar tu lista por engagement para no enviar a usuarios inactivos, e incluir la cabecera `List-Unsubscribe` estándar que permite a los proveedores de correo mostrar un botón de baja automática. Educa a tus suscriptores a usar el enlace de baja en lugar del botón de "marcar como spam", aunque esto último es más fácil decirlo que lograrlo.

## 10.4 Barracuda BRBL: Desliste Automático y Control de Reputación

Barracuda Networks mantiene su Barracuda Reputation Block List (BRBL) basándose en los datos recogidos por más de 200.000 appliances de seguridad de correo desplegados globalmente. A diferencia de SpamCop, Barracuda es menos sensible y solo lista IPs con comportamiento claramente abusivo y sostenido en el tiempo.

**Proceso de desliste:** Barracuda no ofrece un formulario de desliste manual público. Su sistema es completamente automático: cuando la IP deja de mostrar comportamiento abusivo, es removida automáticamente de la lista en un plazo que puede ir de unas horas a varios días, dependiendo de la severidad y duración del incidente. No hay forma de acelerar este proceso, por lo que lo único que puedes hacer es asegurarte de que el abuso ha cesado completamente y esperar.

**Verificación de estado:** Puedes verificar si tu IP está en BRBL en https://www.barracudacentral.org/lookup. Si la IP aparece listada, la página te mostrará información adicional sobre el motivo, aunque generalmente es menos detallada que la de Spamhaus.

Si después de varios días de comportamiento limpio tu IP sigue listada en Barracuda, puedes intentar contactar a su soporte técnico a través de su portal de clientes, aunque sin una cuenta de Barracuda activa es difícil obtener respuesta. En la práctica, la expiración automática es confiable y no deberías necesitar intervención manual.

## 10.5 Cambio de IP: La Solución Rápida (Con Riesgos)

En situaciones extremas, cambiar la dirección IP de tu servidor de correo puede ser la solución más rápida para recuperar la capacidad de envío. Sin embargo, esta decisión no debe tomarse a la ligera, porque tiene implicaciones importantes tanto positivas como negativas.

**Cuándo SÍ tiene sentido cambiar de IP:**

Si tu IP actual tiene un historial largo y documentado de abuso que no fue causado por ti (por ejemplo, si adquiriste una IP "sucia" de un proveedor de cloud que antes perteneció a un spammer), cambiar de IP es una jugada inteligente. También tiene sentido si estás atrapado en listas extorsivas como UCEPROTECT Level 2 o 3, que listan rangos CIDR completos y cuyo proceso de desliste es prácticamente imposible sin pagar. Otra situación válida es cuando el proceso de desliste en múltiples listas se ha vuelto tan lento y engorroso que el costo de oportunidad de no poder enviar correo supera el costo y riesgo de migrar a una nueva IP.

**Cuándo NO tiene sentido cambiar de IP:**

Si la causa raíz del listado sigue presente, cambiar de IP es inútil porque la nueva IP terminará igualmente en las mismas listas negras en cuestión de días o incluso horas. Esto ocurre cuando el problema es tu comportamiento de envío (listas compradas, alta tasa de quejas, falta de autenticación) y no la IP en sí misma. Tampoco tiene sentido cambiar si tu reputación de dominio está intacta pero tu IP está listada por PBL (la solución es usar un relay, no cambiar de IP). Finalmente, si tu IP actual está en listas blancas o tiene una buena reputación acumulada con ciertos proveedores, cambiarla implica perder ese capital reputacional.

**Cómo ejecutar un cambio de IP de forma segura:**

Si decides que cambiar de IP es la mejor opción, sigue este protocolo:

1. Solicita a tu proveedor de hosting una IP limpia (no reasignada recientemente). Algunos proveedores permiten elegir IPs específicas o verificar su historial.
2. Antes de migrar, verifica que la nueva IP no esté en ninguna lista negra usando MXToolbox o consultas manuales con `dig`.
3. Configura el registro PTR (rDNS) de la nueva IP para que coincida con tu nombre de host (por ejemplo, `mail.ejemplo.com`).
4. Actualiza el registro MX de tu dominio para que apunte a la nueva IP.
5. Actualiza el registro SPF si es necesario para incluir la nueva IP.
6. Inicia el warm-up de la IP siguiendo el protocolo descrito en la sección 10.6.
7. Monitorea diariamente las listas negras durante las primeras dos semanas.
8. No abandones la IP anterior inmediatamente; mantenla activa por si necesitas retroceder.

## 10.6 Warm-up de IP: El Protocolo Completo para Construir Reputación

El warm-up (calentamiento) de una dirección IP es el proceso gradual de aumentar el volumen de correo enviado desde una IP nueva o recién deslistada, permitiendo que los proveedores de correo (Gmail, Outlook, Yahoo, etc.) construyan una reputación positiva basada en tu comportamiento de envío. Saltarse este proceso —es decir, comenzar a enviar el volumen completo desde el primer día— es una de las causas más comunes de listados en listas negras y bloqueos por parte de los grandes proveedores.

**Por qué es necesario el warm-up:**

Los proveedores de correo como Google y Microsoft mantienen bases de datos de reputación para cada dirección IP. Cuando ven una IP nueva que nunca ha enviado correo, no tienen información sobre su comportamiento. Si de repente esa IP comienza a enviar miles de correos al día, el algoritmo de detección de spam lo interpreta como un comportamiento sospechoso típico de spammers que rotan IPs para evadir bloqueos. Al aumentar el volumen gradualmente, le das tiempo al proveedor para observar que tus correos tienen bajas tasas de queja, buena tasa de apertura, y que sigues las reglas de autenticación y buenas prácticas.

**El protocolo de warm-up recomendado:**

| Período | Volumen máximo diario | Tipo de destinatarios |
|---------|-----------------------|-----------------------|
| Días 1-3 | 50 correos | Solo tus contactos más leales y activos |
| Días 4-7 | 200 correos | Usuarios que abrieron en los últimos 7 días |
| Días 8-14 | 1.000 correos | Segmento de alta apertura (últimos 30 días) |
| Días 15-21 | 5.000 correos | Segmento medio (abrieron en 30-60 días) |
| Días 22-30 | 10.000 correos | Base completa de activos |
| Día 31+ | Incremento gradual | Hasta alcanzar el volumen objetivo |

**Reglas fundamentales del warm-up:**

1. **Nunca aumentes más del 30% del volumen del día anterior.** Un incremento del 50% o 100% puede activar las alarmas de los proveedores.
2. **Envía primero a tus mejores suscriptores:** aquellos que han abierto correos recientemente, han hecho clic en enlaces, y nunca han marcado tus correos como spam. Estos usuarios generan señales positivas para tu reputación.
3. **Monitorea cada envío:** después de cada lote, revisa las tasas de apertura, clics, rebotes y quejas. Si la tasa de quejas supera el 0.1%, reduce el volumen y espera unos días antes de incrementar nuevamente.
4. **Distribuye los envíos a lo largo del día.** No envíes todo el volumen en una sola ráfaga; espácialo en 2-4 lotes para simular un comportamiento natural.
5. **Configura correctamente la autenticación (SPF, DKIM, DMARC) antes de comenzar el warm-up.** Si tus correos no pasan la autenticación, el warm-up no servirá de nada porque los proveedores no podrán asociar la reputación positiva a tu dominio.

**Qué hacer si el warm-up encuentra problemas:**

Si durante el warm-up notas que las tasas de apertura caen repentinamente, los rebotes aumentan, o comienzas a recibir quejas de spam, detén el incremento de volumen inmediatamente. Mantén el volumen actual o incluso redúcelo durante 3-5 días mientras investigas la causa. Las causas más comunes son: haber incluido un segmento de usuarios de baja calidad, problemas técnicos (enlaces rotos, imágenes que no cargan), o contenido que no es relevante para ese segmento. Una vez identificada y corregida la causa, reanuda el incremento gradual.

## 10.7 Servicios de Email Transaccional: Una Alternativa Estratégica

Si los problemas con listas negras se vuelven recurrentes o si tu infraestructura de correo no es lo suficientemente robusta para mantener una buena reputación por sí sola, considera delegar el envío de correos a servicios especializados de email transaccional. Estos servicios gestionan la reputación de sus IPs a nivel profesional, tienen equipos dedicados a mantener relaciones con los grandes proveedores, y ofrecen herramientas de monitoreo y analytics que facilitan la detección temprana de problemas.

**Comparativa de servicios:**

| Servicio | Precio | Volumen gratuito | Ideal para |
|----------|--------|------------------|------------|
| **SendGrid** | Desde $19.95/mes | 100 correos/día gratis | Marketing y transaccional |
| **Mailgun** | Desde $35/mes | 5.000/mes gratis | Desarrolladores, API |
| **Amazon SES** | $0.10 por 1.000 | 62.000/mes gratis (desde EC2) | Alto volumen, bajo costo |
| **Postmark** | Desde $10/mes | 100/mes gratis | Solo transaccional, máxima entregabilidad |
| **Mailjet** | Desde $9.65/mes | 6.000/mes gratis | Marketing, plantillas |
| **MailerSend** | Desde $11.50/mes | 3.000/mes gratis | Moderno, API |

**Cómo integrar un servicio transaccional con Postfix:**

```bash
# /etc/postfix/main.cf — Usar SendGrid como relay
relayhost = [smtp.sendgrid.net]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_security_level = encrypt

# /etc/postfix/sasl_passwd
[smtp.sendgrid.net]:587 apikey:TU_SENDGRID_API_KEY
```

La principal ventaja de este enfoque es que tu servidor local solo maneja la recepción de correo y el filtrado antispam, mientras que la entrega sale desde IPs gestionadas profesionalmente que mantienen relaciones directas con Gmail, Microsoft y Yahoo. La desventaja es el costo mensual y la dependencia de un tercero para una función crítica del negocio.

## 10.8 Checklist Completo de Desliste

Utiliza esta lista de verificación paso a paso para asegurarte de que no omitas ningún paso crítico en el proceso de desliste. Marca cada elemento con una `x` a medida que lo completes.

```
FASE 1: DIAGNÓSTICO
[ ] Identifiqué todas las listas negras donde mi IP está presente
[ ] Anoté los códigos de retorno de cada lista
[ ] Revisé los logs de mi servidor de correo buscando rechazos
[ ] Determiné la causa raíz del listado
[ ] Documenté cada hallazgo con fechas y evidencias

FASE 2: CORRECCIÓN DE LA CAUSA RAÍZ
[ ] Verifiqué que mi servidor no esté comprometido (revisión de seguridad)
[ ] Actualicé todo el software del servidor a las versiones más recientes
[ ] Cambié todas las contraseñas críticas (root, SSH, FTP, bases de datos)
[ ] Configuré SPF correctamente (verificado con dig)
[ ] Configuré DKIM correctamente (verificado con dig)
[ ] Configuré DMARC al menos en modo monitoreo (p=none)
[ ] Verifiqué que el registro PTR (rDNS) coincida con mi nombre de host
[ ] Aseguré todos los formularios web contra explotación (captcha, rate limiting)
[ ] Implementé rate limiting SMTP para evitar abusos
[ ] Eliminé cualquier lista de correo comprada, alquilada o raspada
[ ] Implementé double opt-in para nuevas suscripciones
[ ] Verifiqué que el enlace de baja funcione correctamente en todos los correos
[ ] Revisé que mi HELO/EHLO sea un FQDN válido y resuelva correctamente

FASE 3: SOLICITUD DE DESLISTE
[ ] Spamhaus SBL: Solicité desliste en https://www.spamhaus.org/lookup/
[ ] Spamhaus PBL: Verifiqué que mi IP cumple con la política o usé relay
[ ] SpamCop: Esperé 24-48h para desliste automático o solicité manual
[ ] Barracuda: Verifiqué que el abuso cesó (desliste automático)
[ ] Otras listas: Solicité desliste en cada una siguiendo su proceso
[ ] Guardé los números de caso o referencias de cada solicitud

FASE 4: MONITOREO POST-DESLISTE
[ ] Verifiqué cada 12 horas que la IP no haya sido relistada (durante 72h)
[ ] Monitoreé las tasas de rebote, apertura y quejas en cada envío
[ ] Mantuve el volumen de envío reducido durante los primeros 3-5 días
[ ] Inicié el warm-up gradual si es una IP nueva
[ ] Configuré alertas automáticas para detectar futuros listados
[ ] Documenté todo el proceso para referencia futura
```
