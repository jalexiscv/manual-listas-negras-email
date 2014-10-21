# Capítulo 4: Tipos de Listas Negras

[← Anterior](03-que-es-una-lista-negra.md) | [Índice](README.md) | [Siguiente →](05-principales-dnsbl.md)

---

## 4.1 Clasificación por el Tipo de Dato que Listan

Las listas negras no son todas iguales: unas listan direcciones IP, otras listan nombres de dominio, otras listan rangos completos de red, e incluso existen listas que listan países enteros. Comprender esta clasificación es esencial para saber qué tipo de lista te está afectando y cómo resolverlo.

### 4.1.1 Listas de IPs (IP-based DNSBL)

Son las más comunes y las más antiguas. Listan direcciones IP específicas de servidores SMTP que han mostrado comportamiento abusivo. Cuando un servidor de correo recibe una conexión, extrae la IP del socket TCP (la dirección del servidor que está estableciendo la conexión) y la consulta contra una o varias de estas listas. Si la IP aparece listada, el servidor receptor aplica la acción configurada.

La consulta se realiza invirtiendo los octetos de la IP y concatenando con el dominio de la lista, como se explicó en el Capítulo 2. Por ejemplo, para la IP `203.0.113.50` contra Spamhaus Zen, la consulta sería `50.113.0.203.zen.spamhaus.org`.

**Ejemplos de listas de IPs:**
- `zen.spamhaus.org` — Spamhaus Zen, la lista compuesta más utilizada globalmente
- `bl.spamcop.net` — SpamCop, basada en reportes de usuarios
- `b.barracudacentral.org` — Barracuda BRBL
- `psbl.surriel.com` — Passive Spam Block List

**Cómo consultar manualmente:**
```bash
$ dig +short 50.113.0.203.bl.spamcop.net
$ dig +short 50.113.0.203.b.barracudacentral.org
```

Si cualquiera de estos comandos devuelve una IP (del rango 127.0.0.0/8), la IP consultada está listada en esa lista. Si devuelve NXDOMAIN o una cadena vacía, está limpia.

### 4.1.2 Listas de Dominios (Domain-based DNSBL)

A diferencia de las listas de IPs, que bloquean servidores, las listas de dominios bloquean nombres de dominio específicos que aparecen en el contenido de los correos no deseados. Esto puede ser el dominio en la dirección `From:`, el dominio en los enlaces dentro del cuerpo del mensaje, o el dominio en la cabecera `Return-Path:`.

Estas listas son particularmente efectivas contra phishing y spam que utiliza enlaces a sitios maliciosos. Incluso si un spammer cambia de servidor (y por tanto de IP), si sigue utilizando el mismo dominio en sus enlaces, seguirá siendo bloqueado.

**Ejemplos de listas de dominios:**
- `dbl.spamhaus.org` — Spamhaus DBL (Domain Block List)
- `multi.surbl.org` — SURBL
- `uribl.spameatingmonkey.net` — URIBL

**Cómo consultar manualmente:**
```bash
$ dig +short ejemplo-maligno.com.dbl.spamhaus.org
$ dig +short ejemplo-maligno.com.multi.surbl.org
```

> 📌 **Dato clave:** Si tu dominio está listado en una DBL, el problema es más grave que si solo tu IP está listada. Cambiar de IP no resolverá el problema porque el dominio viaja con cada correo. Debes resolver por qué tu dominio aparece en correos no solicitados.

### 4.1.3 Listas de Redes (CIDR-based DNSBL)

Estas listas son más agresivas: en lugar de listar IPs individuales, listan rangos completos de direcciones IP (en notación CIDR) que pertenecen a proveedores de servicios o regiones conocidas por albergar spammers. La lógica es que si un cierto porcentaje de IPs en un rango genera spam, todo el rango es problemático.

La controversia de este enfoque es evidente: IPs legítimas que nunca han enviado spam pueden ser bloqueadas simplemente por estar en el mismo rango que spammers. Esto ha llevado a que muchas de estas listas sean consideradas demasiado agresivas y no sean consultadas por servidores reputados.

**Ejemplos:**
- `dul.dnsbl.sorbs.net` — SORBS DUHL (Dial-up User Host List), que lista rangos de ISP de acceso telefónico y doméstico
- Los niveles 2 y 3 de UCEPROTECT, que listan ASNs (Autonomous System Numbers) completos

### 4.1.4 Listas de Países (Geo-based DNSBL)

Estas listas van un paso más allá y listan todas las direcciones IP asignadas a países enteros que tienen una alta proporción de spam en relación con el correo legítimo. Históricamente, países como Rusia, China, Ucrania y varias naciones de Europa del Este y África han sido incluidos en este tipo de listas.

Son extremadamente controvertidas por varias razones: son geopolíticamente insensibles (castigan a usuarios legítimos de todo un país por el comportamiento de una minoría), son fáciles de evadir (los spammers simplemente usan servidores en otros países), y violan el principio de neutralidad de la red.

**Ejemplo histórico:** `countries.nerd.dk` (descontinuado). Esta lista fue popular a principios de los 2000 pero cayó en desuso precisamente por su enfoque demasiado generalista.

> 📌 **Dato clave:** Las listas basadas en país son cada vez más raras y la mayoría de los administradores experimentados las consideran demasiado agresivas e injustas. Ningún proveedor importante de correo (Gmail, Outlook, Yahoo) utiliza este tipo de listas en sus filtros principales.

## 4.2 Clasificación por Método de Recolección de Datos

La forma en que una lista negra recopila las IPs y dominios que incluye determina en gran medida su fiabilidad, su velocidad de reacción y su propensión a falsos positivos.

### 4.2.1 Listas por Spamtrap (Honeypot)

Este es el método más fiable y el que produce menos falsos positivos. Los operadores de listas negras crean direcciones de correo electrónico falsas que nunca se han utilizado para ninguna comunicación legítima. Estas direcciones se "siembran" en lugares donde los spammers suelen recolectar direcciones: páginas web, formularios públicos, listas de correo, o incluso se publican en foros. Cualquier correo que llegue a una de estas direcciones es, por definición, no solicitado, es decir, es spam.

Cuando un servidor envía correo a una spamtrap, su IP queda registrada y, si acumula suficientes envíos a traps, es listada. La ventaja de este método es que los falsos positivos son prácticamente imposibles: una dirección que nunca se ha usado para comunicación legítima no debería recibir correo legítimo. La desventaja es que puede ser lento: pasa tiempo hasta que los spammers encuentran y comienzan a usar las direcciones trap.

### 4.2.2 Listas por Reporte de Usuarios (Feedback Loops)

Este método es más rápido pero más propenso a errores. Permite que los usuarios finales reporten los correos que consideran spam, ya sea reenviándolos a una dirección especial (como hace SpamCop) o haciendo clic en el botón "Reportar spam" en su cliente de correo (que activa el feedback loop del proveedor).

Cuando suficientes usuarios reportan correos provenientes de una misma IP en un período corto, la IP es listada automáticamente o marcada para revisión. La velocidad de detección es alta: si un spammer comienza a enviar desde una IP nueva, los primeros usuarios que reciben sus correos pueden reportarlo y la IP queda listada en cuestión de horas.

El problema principal es la alta tasa de falsos positivos. Un usuario puede marcar como spam un correo legítimo porque no recuerda haberse suscrito, porque el remitente no se identifica claramente, o simplemente porque está de mal humor. Una campaña de marketing perfectamente legítima puede terminar en SpamCop porque unos pocos usuarios confundieron el boletín con spam.

### 4.2.3 Listas Automáticas por Análisis de Comportamiento

Estos sistemas no dependen de traps ni de reportes de usuarios, sino que analizan el tráfico SMTP de cada IP en tiempo real y determinan si su comportamiento es consistente con el de un servidor legítimo o con el de un spammer. Los indicadores que analizan incluyen:

- **Tasa de rebotes:** Si una IP envía a muchas direcciones que no existen (ratio de hard bounces superior al 10%), es un comportamiento típico de spammer que utiliza listas compradas o raspadas.
- **Volumen de envío:** Picos repentinos de volumen (por ejemplo, pasar de 100 a 100.000 correos por hora) son característicos de campañas de spam masivo.
- **Ratio de quejas:** Si una IP recibe muchas quejas de spam a través de feedback loops, su reputación se degrada automáticamente.
- **Patrones de conexión:** Los spammers suelen conectar desde IPs que no tienen registro PTR, o cuyo PTR no coincide con el nombre declarado en EHLO.
- **Protocolo SMTP no estándar:** Secuencias de comandos SMTP incorrectas, tiempos de espera anómalos, o intentos de enviar sin autenticar son señales de alerta.

### 4.2.4 Listas Curadas por Expertos (Manuales)

Este es el método más caro pero también el más preciso. Analistas humanos con experiencia revisan evidencias (correos completos con cabeceras, logs de servidores, reportes de usuarios) y deciden caso por caso si una IP o dominio merece ser listado. Solo Spamhaus tiene la escala y los recursos para mantener un equipo de analistas trabajando 24/7.

La ventaja es innegable: los falsos positivos son prácticamente inexistentes porque un humano puede distinguir entre un spammer y un remitente legítimo que cometió un error. La desventaja es que el proceso es más lento y costoso, por lo que estas listas suelen tener menos cobertura que las automáticas.

> 💡 **Consejo:** Las listas curadas son las que más peso deben tener en tu configuración. Si estás listado en Spamhaus SBL (curada), el problema es real y debes actuar con seriedad. Si solo estás en SpamCop (reportes de usuarios), puede ser un falso positivo y el desliste suele ser más sencillo.

## 4.3 Listas Consultables vs. Listas Privadas

### Listas Públicas Consultables

Son las listas negras tradicionales que cualquier servidor de correo puede consultar mediante consultas DNS estándar. Su funcionamiento es transparente: sabes qué listas consulta tu servidor y puedes verificar tu IP contra ellas manualmente. Spamhaus, SpamCop y Barracuda son ejemplos de listas públicas.

### Listas Privadas o Internas

Los grandes proveedores de correo como Gmail, Outlook.com, Yahoo Mail y ProtonMail mantienen sus propias listas negras internas que **no son consultables externamente**. No existe un comando `dig` ni una herramienta web que te diga si estás en la lista interna de Gmail. Solo puedes inferirlo indirectamente: si tus correos a usuarios de Gmail no llegan o van a spam consistentemente, es probable que estés en su lista interna.

Estas listas internas son mucho más determinantes para la entregabilidad real que cualquier DNSBL pública, porque son los proveedores con mayor cantidad de usuarios. Google solo, con más de 1.800 millones de cuentas activas, es el árbitro final de la entregabilidad para la mayoría de los remitentes.

> ⚠️ **Advertencia:** Puedes estar perfectamente limpio en todas las listas públicas (Spamhaus, SpamCop, Barracuda) y aún así tener graves problemas de entregabilidad en Gmail u Outlook si tu reputación con ellos es mala. Las listas internas de los grandes proveedores son cajas negras: no sabes con certeza si estás en ellas, ni exactamente por qué, ni cuándo saldrás. La única forma de mejorar tu situación es seguir rigurosamente las mejores prácticas descritas en los capítulos siguientes.
