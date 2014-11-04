# Capítulo 5: Las Principales DNSBL del Mundo

[← Anterior](04-tipos-de-listas-negras.md) | [Índice](README.md) | [Siguiente →](06-como-funcionan.md)

---

## 5.1 Spamhaus Project — El Estándar de Oro del Bloqueo de Spam

**URL oficial:** https://www.spamhaus.org  
**Zona principal de consulta:** `zen.spamhaus.org`  
**Tipo de organización:** Sin fines de lucro, registrada en el Reino Unido  
**Año de fundación:** 1998

Spamhaus es, sin discusión, la lista negra más respetada, más consultada y más influyente del mundo. Según estimaciones de la propia organización y de analistas independientes, es consultada por más del 80% de los servidores de correo que utilizan DNSBL, procesando varios miles de millones de consultas diarias. Su autoridad se basa en una combinación de factores: la rigurosidad de sus criterios de inclusión, la verificación humana de cada caso, la transparencia de sus procesos, y la negativa rotunda a aceptar pagos por desliste.

Spamhaus no es una sola lista, sino un conjunto de listas especializadas que cubren diferentes tipos de amenazas. Desde 2006, la mayoría de los administradores consultan `zen.spamhaus.org`, que es una zona compuesta que integra las listas más importantes en una sola consulta DNS.

### Las Listas que Componen Spamhaus

**SBL (Spamhaus Block List) — Zona: `sbl.spamhaus.org`**
Es la lista principal y la más selectiva. Incluye direcciones IP que han sido identificadas como fuentes de spam directo, confirmado por analistas humanos de Spamhaus. No se lista a la ligera: cada IP en SBL ha sido investigada y hay evidencia concreta de que desde ella se está enviando correo no solicitado. Si tu IP aparece en SBL, el problema es grave y requiere atención inmediata. El código de retorno típico en Zen es `127.0.0.2`. El desliste requiere demostrar que la causa del spam ha sido eliminada.

**XBL (Exploits Block List) — Zona: `xbl.spamhaus.org`**
Esta lista contiene IPs que muestran evidencia de estar comprometidas por malware, virus, gusanos, troyanos, o que operan proxies abiertos, servidores SOCKS, o redes de bots (botnets). A diferencia de la SBL, la XBL no lista por envío de spam directo, sino por comportamiento técnico comprometido. Si tu IP aparece en XBL, es muy probable que tu servidor esté infectado o mal configurado. El código de retorno en Zen es `127.0.0.4` o `127.0.0.5`.

**PBL (Policy Block List) — Zona: `pbl.spamhaus.org`**
La PBL es diferente de las anteriores: no lista porque hayas hecho algo malo, sino porque tu dirección IP pertenece a un rango que, según la política de Spamhaus, no debería estar enviando correo SMTP directamente. Esto incluye la gran mayoría de IPs de ISP residenciales (conexiones de fibra óptica doméstica, ADSL, cable módem, LTE/4G/5G), IPs dinámicas, e IPs asignadas a redes que no son de servidores. La PBL es la lista más voluminosa de Spamhaus, con millones de IPs incluidas. El código de retorno en Zen es `127.0.0.6` o `127.0.0.7`. La solución no es solicitar desliste (aunque es posible en algunos casos), sino usar un relay SMTP autenticado.

**DBL (Domain Block List) — Zona: `dbl.spamhaus.org`**
A diferencia de las anteriores, la DBL no lista IPs sino nombres de dominio. Un dominio aparece en la DBL cuando ha sido encontrado en el cuerpo de correos no solicitados, ya sea como enlace, como dirección de respuesta, o como dominio en la cabecera `From:`. La DBL es particularmente efectiva contra phishing y malware distribuido por correo.

**SWL (Spamhaus Whitelist) — Zona: `swl.spamhaus.org`**
Es la lista blanca de Spamhaus. Las IPs incluidas en la SWL son consideradas remitentes de confianza y no serán bloqueadas por las otras listas de Spamhaus. Estar en la SWL es el "nirvana" de la entregabilidad, pero los requisitos son estrictos y el proceso de admisión es manual.

### Códigos de Respuesta de Zen

Cuando consultas `zen.spamhaus.org` y obtienes una respuesta, la dirección IP devuelta (siempre en el rango 127.0.0.0/8) codifica la razón del listado:

| IP devuelta | Lista de origen | Significado |
|-------------|-----------------|-------------|
| `127.0.0.2` | SBL | Spam directo confirmado por analistas humanos |
| `127.0.0.3` | SBL | Snowshoe spam (muchas IPs, poco volumen cada una) |
| `127.0.0.4` | XBL | Exploits, malware, proxies abiertos, redes de bots |
| `127.0.0.5` | XBL | Redes de bots C&C (Command & Control) |
| `127.0.0.6` | PBL | IP residencial o dinámica que no debe enviar correo directo |
| `127.0.0.7` | PBL | IP asignada a URL de proveedor (no debe enviar correo) |
| `127.0.0.8` | SBL | Dominios .XYZ y otros TLDs considerados abusivos |
| `127.0.0.9` | SBL | Recién observado como problemático (nuevas detecciones) |
| `127.0.0.10` | SBL | Recién observado por contenido (nuevas detecciones) |
| `127.0.0.11` | SBL | Spamhaus sugiere ignorar esta IP (casos especiales) |

### Proceso de Desliste en Spamhaus

1. **Identificar el motivo:** Ve a https://check.spamhaus.org/ e ingresa tu IP. La página te mostrará el código de retorno y una explicación detallada.
2. **Corregir la causa raíz:** Si es PBL, debes configurar tu servidor para usar un relay. Si es SBL o XBL, debes detener el abuso (limpiar el servidor, cambiar contraseñas, eliminar listas compradas).
3. **Solicitar el desliste:** Usa el formulario en https://www.spamhaus.org/lookup/. Explica clara y detalladamente las medidas correctivas que tomaste.
4. **Esperar:** Para SBL, el tiempo de respuesta típico es de 24 a 48 horas hábiles. Para PBL, si la IP cumple con los requisitos, el desliste puede ser inmediato o en pocas horas.

> 📌 **Dato clave:** Spamhaus **nunca cobra** por deslistar. Si alguien te contacta ofreciéndote quitarte de Spamhaus a cambio de dinero, es una estafa. El proceso de desliste es completamente gratuito.

## 5.2 SpamCop — El Poder de los Reportes Ciudadanos

**URL oficial:** https://www.spamcop.net  
**Zona de consulta:** `bl.spamcop.net`  
**Propietario:** Cisco Systems (adquirido en 2005)

SpamCop es fundamentalmente diferente de Spamhaus: mientras que Spamhaus depende de analistas humanos, SpamCop automatiza completamente el proceso de listado basándose en los reportes de los usuarios. Cualquier persona con una dirección de correo puede registrarse en SpamCop y comenzar a reportar los correos no deseados que recibe.

**Cómo funciona el listado automático:** Cuando un usuario reporta un correo como spam, SpamCop analiza las cabeceras completas del mensaje para extraer la dirección IP real del servidor que lo envió. Si esa IP acumula un número suficiente de reportes en un período de tiempo determinado (el umbral exacto no es público, pero se sabe que es relativamente bajo), la IP es añadida automáticamente a `bl.spamcop.net`.

**Características distintivas:**
- **Alta sensibilidad:** Se necesita relativamente pocos reportes para que una IP sea listada. Esto la hace muy reactiva, pero también propensa a falsos positivos.
- **Listado temporal:** A diferencia de Spamhaus SBL, donde el listado puede durar semanas si no se solicita desliste, SpamCop expira automáticamente la inclusión si los reportes cesan. Generalmente, si una IP deja de recibir nuevos reportes, el listado expira en 24 a 48 horas.
- **Propenso a falsos positivos:** Una campaña de email marketing legítima puede ser listada si unos pocos destinatarios confundidos marcan el correo como spam en lugar de darse de baja.

**Proceso de desliste:**
- **Automático:** Si los reportes cesan completamente, la IP sale automáticamente en 24-48 horas. Esta es la opción recomendada en la mayoría de los casos.
- **Manual:** Si el automático no funciona o necesitas acelerar el proceso, puedes usar el formulario en https://www.spamcop.net/fom-serve/cache/329.html. Explica la situación y proporciona evidencias de que el abuso ha cesado. El proceso manual puede tardar de 24 a 72 horas.

> 💡 **Consejo:** SpamCop es la lista en la que más probablemente terminarás si envías newsletters y algunos usuarios marcan "es spam" en lugar de usar el enlace de baja. Educa a tus suscriptores a usar la baja y no el botón de reporte de spam. Incluye siempre la cabecera List-Unsubscribe para que los proveedores de correo muestren un botón de baja automática.

## 5.3 Barracuda Reputation Block List (BRBL)

**URL oficial:** https://www.barracudacentral.org/lookup  
**Zona de consulta:** `b.barracudacentral.org`  
**Propietario:** Barracuda Networks

Barracuda Networks es una empresa de seguridad que fabrica appliances de protección de correo electrónico. Con más de 200.000 dispositivos desplegados globalmente, Barracuda tiene una visibilidad única del tráfico de correo mundial. La BRBL se alimenta de los datos recogidos por estos dispositivos.

**Características:**
- **Menos sensible que Spamhaus:** Barracuda solo lista IPs con comportamiento claramente abusivo y sostenido. No lista por PBL ni por sospechas leves.
- **Efecto "network effect":** A más dispositivos Barracuda desplegados, mejor es la detección, porque cada dispositivo aporta datos de su tráfico local.
- **Desliste automático:** No hay formulario manual. Cuando el comportamiento abusivo cesa, la IP es removida automáticamente.

## 5.4 SURBL — Listas de URI (Enlaces en el Cuerpo)

**URL oficial:** https://surbl.org  
**Zona de consulta:** `multi.surbl.org`

SURBL es única en su categoría: no lista la IP del remitente, sino los **dominios que aparecen en los enlaces (URLs) dentro del cuerpo del correo**. Esto significa que puede detectar spam incluso si el remitente cambia constantemente de servidor.

**Cómo funciona:**
1. El servidor receptor recibe un correo
2. Extrae todos los enlaces (URLs) del cuerpo del mensaje
3. Para cada dominio en esos enlaces, consulta SURBL
4. Si algún dominio está listado, el correo es marcado como spam

**Zonas disponibles:**
| Zona | Contenido |
|------|-----------|
| `multi.surbl.org` | Combinación de todas las fuentes |
| `abuse.surbl.org` | Dominios de abuso verificados |
| `phish.surbl.org` | Dominios de phishing |
| `malware.surbl.org` | Dominios que alojan o distribuyen malware |
| `spam.surbl.org` | Dominios encontrados en correos spam |

## 5.5 Otras Listas Notables

| Lista | Zona DNS | Tipo | Fiabilidad | Nota |
|-------|----------|------|:-----------:|------|
| **PSBL** | `psbl.surriel.com` | IP | ⭐⭐⭐ | Comunitaria, abierta y gratuita |
| **UCEPROTECT L1** | `dnsbl-1.uceprotect.net` | IP (CIDR) | ⭐⭐ | Level 1 lista IPs, niveles superiores listan ASNs |
| **UCEPROTECT L2/L3** | `dnsbl-2.uceprotect.net` | ASN | ⭐ | Extorsiva. Lista redes completas. No pagar. |
| **Invaluement** | `dnsbl.invaluement.com` | IP | ⭐⭐⭐ | Comercial, usada por proveedores de hosting |
| **SORBS** | `dnsbl.sorbs.net` | IP | ⭐⭐ | Históricamente relevante, declinó en fiabilidad |
| **SpamEatingMonkey** | `uribl.spameatingmonkey.net` | URI | ⭐⭐⭐ | Lista de URI mantenida por comunidad |

> ⚠️ **Advertencia:** No intentes deslistarte de UCEPROTECT. Su modelo de negocio se basa en listar IPs masivamente y cobrar por el desliste. La mayoría de los servidores de correo serios no consultan UCEPROTECT. Si tu IP aparece allí, ignórala y concéntrate en las listas que realmente importan.
