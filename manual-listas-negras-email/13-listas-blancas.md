# Capítulo 13: Listas Blancas y Reputación Positiva

[← Anterior](12-monitoreo.md) | [Índice](00-indice.md) | [Siguiente →](14-estudios-de-caso.md)

---

## 13.1 ¿Qué es una Lista Blanca?

Base de datos de remitentes de confianza. Los correos en listas blancas:
- Saltan filtros antispam
- Tienen entrega garantizada en bandeja de entrada
- No tienen rate limiting

## 13.2 Principales Listas Blancas

### Spamhaus Whitelist (SWL)

```bash
dig +short 50.113.0.203.swl.spamhaus.org
# 127.0.0.1 = estas en whitelist
```

**Requisitos:** IP dedicada, sin historial de abuso, configuración impecable, proceso manual.

### Otras
- **Barracuda:** Automática para remitentes con buena reputación
- **Microsoft:** Remitentes con buena reputación en SNDS
- **Google:** No hay whitelist pública, pero remitentes verificados obtienen trato preferencial

## 13.3 Factores de Reputación

### De IP
| Factor | Peso |
|--------|:----:|
| Historial de spam | Alto |
| Tasa de rebotes | Alto |
| Tasa de quejas | Alto |
| Autenticación (SPF/DKIM/DMARC) | Alto |
| PTR coincidente con HELO | Medio |
| Edad de la IP | Medio |
| Volumen consistente | Medio |

### De Dominio
| Factor | Peso |
|--------|:----:|
| Edad del dominio | Alto |
| Historial DMARC | Alto |
| Contenido del sitio web | Medio |
| Enlaces salientes | Medio |

## 13.4 Recuperación de Reputación

| Semana | Acción |
|--------|--------|
| 1-2 | Detener envíos, corregir técnico, deslistarse |
| 3-4 | Warm-up al 10% del volumen normal |
| 5-8 | Crecimiento +20% por semana |
| 9+ | Volumen normal, monitoreo continuo |

> 📌 No hay atajos. La confianza se gana con semanas de comportamiento consistente.
