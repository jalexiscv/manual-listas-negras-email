# Apéndice A: Tabla Comparativa Completa de DNSBL

[← Glosario](15-glosario.md) | [Índice](README.md) | [Siguiente →](apendice-b-codigos-smtp.md)

---

## Comparativa Detallada de las Principales DNSBL

Esta tabla comparativa incluye todas las listas negras importantes que un administrador de sistemas debe conocer, ordenadas por relevancia y fiabilidad. Los niveles de fiabilidad se basan en la experiencia documentada de la comunidad de administradores y en la frecuencia con que estas listas son consultadas por servidores de correo reputados.

| DNSBL | Zona DNS | Tipo | Fiabilidad | Desliste | Volumen |
|-------|----------|:----:|:----------:|----------|:-------:|
| **Spamhaus SBL** | `sbl.spamhaus.org` | IP | ⭐⭐⭐⭐⭐ | Manual, gratuito, 24-48h | Medio |
| **Spamhaus XBL** | `xbl.spamhaus.org` | IP | ⭐⭐⭐⭐⭐ | Automático al limpiar el servidor | Medio |
| **Spamhaus PBL** | `pbl.spamhaus.org` | IP | ⭐⭐⭐⭐ | Automático al cumplir política | Muy alto |
| **Spamhaus Zen** | `zen.spamhaus.org` | Compuesta | ⭐⭐⭐⭐⭐ | Varía según sub-lista | Muy alto |
| **Spamhaus DBL** | `dbl.spamhaus.org` | Dominio | ⭐⭐⭐⭐⭐ | Manual, gratuito | Alto |
| **SpamCop** | `bl.spamcop.net` | IP | ⭐⭐⭐ | Automático (48h) o manual | Alto |
| **Barracuda BRBL** | `b.barracudacentral.org` | IP | ⭐⭐⭐⭐ | Automático al cesar abuso | Medio |
| **PSBL** | `psbl.surriel.com` | IP | ⭐⭐⭐ | Manual, gratuito | Bajo |
| **UCEPROTECT L1** | `dnsbl-1.uceprotect.net` | IP | ⭐⭐ | Pago recomendado | Alto |
| **UCEPROTECT L2** | `dnsbl-2.uceprotect.net` | IP (CIDR) | ⭐ | Casi imposible | Muy alto |
| **SURBL** | `multi.surbl.org` | URI (dominio) | ⭐⭐⭐⭐ | N/A (depende del dominio) | Alto |
| **SORBS** | `dnsbl.sorbs.net` | IP | ⭐⭐ | Variable, inconsistente | Medio |
| **Invaluement** | `dnsbl.invaluement.com` | IP | ⭐⭐⭐ | Manual | Bajo |
| **MXToolbox** | `misc.dnsbl.sorbs.net` | Mixta | ⭐⭐ | Variable | Bajo |
| **SpamEatingMonkey** | `uribl.spameatingmonkey.net` | URI | ⭐⭐⭐ | Automático | Bajo |

## Listas que NO Deberías Consultar ni Pagar

| Lista | Razón para evitarla |
|-------|---------------------|
| **UCEPROTECT Level 3** | Lista ASNs completos (redes enteras). Modelo de negocio extorsivo. Pagar solo valida su modelo. |
| **SORBS (gestión reciente)** | Proceso de desliste inconsistente, arbitrario y lento. Su reputación ha declinado significativamente. |
| **AHBL** | Descontinuada desde 2012. Ya no se mantiene. |
| **NJABL** | Descontinuada. Cualquier consulta a esta zona es inútil. |
| **listas.dnsbl.com.au** | Baja cobertura, mantenimiento irregular. |

## Configuración Recomendada para tu Servidor

Para la mayoría de los servidores de correo, esta configuración de DNSBL ofrece el mejor equilibrio entre protección y falsos positivos:

```bash
# Imprescindibles (prioridad alta)
zen.spamhaus.org       # Spamhaus compuesto: SBL + XBL + PBL
bl.spamcop.net         # SpamCop: reportes de usuarios
b.barracudacentral.org # Barracuda: comportamiento abusivo

# Complementos recomendados (prioridad media)
psbl.surriel.com       # PSBL: lista comunitaria gratuita
multi.surbl.org        # SURBL: verifica dominios en enlaces del cuerpo

# Opcionales (bajo impacto en falsos positivos)
dnsbl.invaluement.com  # Invaluement: lista comercial selectiva
uribl.spameatingmonkey.net # SpamEatingMonkey: URI comunitaria
```
