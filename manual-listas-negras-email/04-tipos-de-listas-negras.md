# Capítulo 4: Tipos de Listas Negras

[← Anterior](03-que-es-una-lista-negra.md) | [Índice](00-indice.md) | [Siguiente →](05-principales-dnsbl.md)

---

## 4.1 Por lo que Listan

### 4.1.1 Listas de IPs (IP-based DNSBL)
Las más comunes. Listan direcciones IP de servidores SMTP.

**Ejemplos:** `zen.spamhaus.org`, `bl.spamcop.net`, `b.barracudacentral.org`

```bash
dig +short 50.113.0.203.zen.spamhaus.org
```

### 4.1.2 Listas de Dominios (Domain-based DNSBL)
Listan dominios presentes en correos spam (enlaces, desde, etc.).

**Ejemplos:** `dbl.spamhaus.org`, `multi.surbl.org`

```bash
dig +short ejemplo.com.dbl.spamhaus.org
```

### 4.1.3 Listas de Redes (CIDR-based DNSBL)
Listan rangos completos de IP.

**Ejemplo:** `dul.dnsbl.sorbs.net`

### 4.1.4 Listas de Países (Geo-based DNSBL)
Listan IPs de países considerados problemáticos. Controvertidas y cada vez más raras.

## 4.2 Por Método de Recolección

| Tipo | Cómo funciona | Falsos positivos |
|------|---------------|:----------------:|
| **Spamtrap / Honeypot** | Direcciones falsas que nunca se usaron. Cualquier correo que llegue es spam | Muy bajo |
| **Reporte de usuarios** | Usuarios marcan correos como spam | Alto |
| **Automático por comportamiento** | Analiza tráfico SMTP: rebotes, quejas, picos | Medio |
| **Curado por expertos** | Analistas humanos revisan evidencias | Muy bajo |

## 4.3 Consultadas vs. No Consultadas

### Públicas (Query-based)
Cualquier servidor puede consultarlas voluntariamente.

### Privadas / Internas
Gmail, Outlook, Yahoo, ProtonMail tienen sus propias listas internas. **No son consultables externamente.** Son cajas negras.

> ⚠️ **Advertencia:** Puedes estar limpio en todas las listas públicas y aún así tener problemas en Gmail u Outlook. Las listas internas de los grandes proveedores son las más determinantes.
