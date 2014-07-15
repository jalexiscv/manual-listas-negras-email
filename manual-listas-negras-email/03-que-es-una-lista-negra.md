# Capítulo 3: ¿Qué es una Lista Negra?

[← Anterior](02-fundamentos.md) | [Índice](00-indice.md) | [Siguiente →](04-tipos-de-listas-negras.md)

---

## 3.1 Definición

Una **lista negra de correo electrónico** (DNSBL o RBL) es una base de datos pública de IPs o dominios identificados como fuentes de spam. Cuando un servidor recibe un correo, consulta una o más listas negras. Si la IP del remitente aparece, el servidor puede:

- Rechazar el mensaje (lo más común)
- Marcarlo como spam (cuarentena)
- Aceptarlo con menor prioridad (throttling)
- Aceptarlo con cabecera de advertencia

## 3.2 Breve Historia

| Año | Hito |
|-----|------|
| 1997 | MAPS RBL — Primera lista negra (Paul Vixie) |
| 1998 | Spamhaus — Fundada por Steve Linford |
| 1999 | SpamCop — Julian Haight |
| 2003 | Barracuda — Basada en sus dispositivos de seguridad |
| 2006 | Spamhaus Zen — Consolidación de listas |
| 2010 | Microsoft SmartScreen y Gmail desarrollan sistemas propios |
| 2013 | Spamhaus DBL — Listas de dominios |
| 2020+ | Grandes proveedores confían más en ML propio que en DNSBL públicas |

## 3.3 ¿Quién Opera las Listas Negras?

1. **Organizaciones sin fines de lucro:** Spamhaus, SpamCop
2. **Empresas de seguridad:** Barracuda, Trend Micro, Proofpoint
3. **Comunidades de voluntarios:** SURBL
4. **Proveedores:** Microsoft (SmartScreen), Google (Postmaster Tools)
5. **Proyectos open-source:** PSBL

> ⚠️ No todas las listas son iguales. Spamhaus es extremadamente fiable; otras como UCEPROTECT son consideradas extorsivas.

## 3.4 Listas Legítimas vs. Extorsivas

### Legítimas
- Criterios claros y públicos
- Desliste gratuito y documentado
- Responden a apelaciones
- Usadas por múltiples proveedores grandes

### Extorsivas (evitar)
- Cobran por deslistar (pay-to-delist)
- No publican criterios
- Listan IPs masivamente sin verificación
- Ejemplo: UCEPROTECT Level 3

> 📌 **Dato clave:** Si una lista te exige dinero para deslistarte sin alternativa gratuita, es extorsiva. La mayoría de servidores serios no la consultan.
