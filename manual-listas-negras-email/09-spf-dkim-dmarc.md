# Capítulo 9: SPF, DKIM y DMARC — El Santo Grial de la Autenticación

[← Anterior](08-como-detectar.md) | [Índice](00-indice.md) | [Siguiente →](10-estrategias-desliste.md)

---

## 9.1 SPF (Sender Policy Framework)

Especifica **qué servidores están autorizados** a enviar correo para tu dominio.

```txt
ejemplo.com.  IN  TXT  "v=spf1 ip4:203.0.113.0/24 include:_spf.google.com -all"
```

| Mecanismo | Significado |
|-----------|-------------|
| `ip4:` | Autoriza IPs en este rango |
| `include:` | Incluye servidores externos |
| `-all` | Rechazar todo lo demás (estricto) |
| `~all` | Marcar como sospechoso (softfail) |

> ⚠️ Límite de 10 consultas DNS por SPF. No incluyas demasiados `include:`.

```bash
dig +short TXT ejemplo.com | grep "v=spf1"
```

## 9.2 DKIM (DomainKeys Identified Mail)

Firma digital de correos. Clave privada en el servidor, pública en DNS.

### Configuración

```bash
# Generar par de claves
openssl genrsa -out dkim-private.pem 2048
openssl rsa -in dkim-private.pem -pubout -out dkim-public.pem

# Publicar en DNS
# mail._domainkey.ejemplo.com.  IN  TXT  "v=DKIM1; h=sha256; k=rsa; p=..."
```

## 9.3 DMARC (Domain-based Message Authentication, Reporting & Conformance)

Unifica SPF y DKIM. Le dice al receptor **qué hacer** cuando ambos fallan.

```txt
_dmarc.ejemplo.com.  IN  TXT  "v=DMARC1; p=quarantine; rua=mailto:dmarc@ejemplo.com"
```

| Etiqueta | Valores | Descripción |
|----------|---------|-------------|
| `p` | none / quarantine / reject | Política |
| `rua` | mailto: | Reportes agregados |
| `ruf` | mailto: | Reportes forenses |

### Estrategia de Implementación

1. **Fase 1 (p=none):** Solo monitorear — 2-4 semanas
2. **Fase 2 (p=quarantine):** Spam para no autenticados — 2-4 semanas
3. **Fase 3 (p=reject):** Protección máxima

## 9.4 BIMI (Brand Indicators for Message Identification)

Muestra el logo junto al correo. Requiere DMARC en p=quarantine o p=reject.

```txt
default._bimi.ejemplo.com.  IN  TXT  "v=BIMI1; l=https://ejemplo.com/logo.svg"
```

## 9.5 ARC (Authenticated Received Chain)

Permite que servicios de reenvío (listas de correo, Google Groups) no rompan SPF/DKIM/DMARC.
