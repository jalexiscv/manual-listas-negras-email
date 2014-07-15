# Apéndice A: Tabla Comparativa de DNSBL

[← Glosario](15-glosario.md) | [Índice](00-indice.md) | [Siguiente →](apendice-b-codigos-smtp.md)

---

| DNSBL | Zona | Tipo | Fiabilidad | Desliste | Volumen |
|-------|------|------|:----------:|----------|:-------:|
| **Spamhaus SBL** | `sbl.spamhaus.org` | IP | ⭐⭐⭐⭐⭐ | Manual, gratuito | Medio |
| **Spamhaus XBL** | `xbl.spamhaus.org` | IP | ⭐⭐⭐⭐⭐ | Automático | Medio |
| **Spamhaus PBL** | `pbl.spamhaus.org` | IP | ⭐⭐⭐⭐ | Automático | Muy alto |
| **Spamhaus Zen** | `zen.spamhaus.org` | Compuesta | ⭐⭐⭐⭐⭐ | Varía | Muy alto |
| **Spamhaus DBL** | `dbl.spamhaus.org` | Dominio | ⭐⭐⭐⭐⭐ | Manual | Alto |
| **SpamCop** | `bl.spamcop.net` | IP | ⭐⭐⭐ | Automático (48h) | Alto |
| **Barracuda** | `b.barracudacentral.org` | IP | ⭐⭐⭐⭐ | Automático | Medio |
| **PSBL** | `psbl.surriel.com` | IP | ⭐⭐⭐ | Manual | Bajo |
| **UCEPROTECT L1** | `dnsbl-1.uceprotect.net` | IP | ⭐⭐ | Pago | Alto |
| **UCEPROTECT L2** | `dnsbl-2.uceprotect.net` | IP | ⭐ | Casi imposible | Muy alto |
| **SURBL** | `multi.surbl.org` | URI | ⭐⭐⭐⭐ | N/A (dominio) | Alto |
| **SORBS** | `dnsbl.sorbs.net` | IP | ⭐⭐ | Variable | Medio |
| **Invaluement** | `dnsbl.invaluement.com` | IP | ⭐⭐⭐ | Manual | Bajo |

## Listas a Evitar

| Lista | Razón |
|-------|-------|
| **UCEPROTECT Level 3** | Lista rangos completos de ISP. Extorsiva. |
| **SORBS (reciente)** | Proceso de desliste inconsistente. |
| **AHBL** | Descontinuada |
| **NJABL** | Descontinuada |

## Configuración Recomendada

```bash
# Imprescindibles
zen.spamhaus.org
bl.spamcop.net
b.barracudacentral.org

# Opcionales
psbl.surriel.com
multi.surbl.org
```
