# Capítulo 5: Las Principales DNSBL del Mundo

[← Anterior](04-tipos-de-listas-negras.md) | [Índice](README.md) | [Siguiente →](06-como-funcionan.md)

---

## 5.1 Spamhaus Project — El Estándar de Oro

**URL:** https://www.spamhaus.org | **Zona:** `zen.spamhaus.org`
Consultada por >80% de los servidores que usan DNSBL.

| Zona | Tipo | Descripción |
|------|------|-------------|
| `zen.spamhaus.org` | Compuesta | SBL + XBL + PBL + DBL |
| `sbl.spamhaus.org` | IP | Spammers confirmados manualmente |
| `xbl.spamhaus.org` | IP | Exploits, malware, proxies |
| `pbl.spamhaus.org` | IP | IPs que no deben enviar correo directo |
| `dbl.spamhaus.org` | Dominio | Dominios en correos spam |

### Códigos Zen

| IP | Significado |
|----|-------------|
| 127.0.0.2 | SBL — spam directo |
| 127.0.0.4 | XBL — exploits/proxies |
| 127.0.0.6 | PBL — IP residencial/dinámica |
| 127.0.0.8 | SBL — TLDs abusivos |

### Desliste Spamhaus
1. Verificar en https://check.spamhaus.org/
2. Corregir la causa
3. Solicitar desliste (gratuito)
4. 24-48h para SBL, inmediato para PBL

> 📌 **Spamhaus nunca cobra por deslistar.** Si alguien te pide dinero, es estafa.

## 5.2 SpamCop

**Zona:** `bl.spamcop.net` | Basado en reportes de usuarios.

- Alta sensibilidad (pocos reportes listan)
- Listado temporal (sale automático en 24-48h si cesan reportes)
- Propenso a falsos positivos

## 5.3 Barracuda BRBL

**Zona:** `b.barracudacentral.org`
- Menos sensible que Spamhaus
- Desliste automático al cesar el abuso

## 5.4 SURBL

**Zona:** `multi.surbl.org` | Lista **dominios en enlaces** del cuerpo del correo.

| Zona | Contenido |
|------|-----------|
| `abuse.surbl.org` | Abuso verificado |
| `phish.surbl.org` | Phishing |
| `malware.surbl.org` | Malware |

## 5.5 Otras Listas

| Lista | Zona | Nota |
|-------|------|------|
| UCEPROTECT | `dnsbl-1.uceprotect.net` | Extorsiva. Evitar. |
| PSBL | `psbl.surriel.com` | Comunitaria, gratuita |
| Invaluement | `dnsbl.invaluement.com` | Comercial |

> ⚠️ No intentes deslistarte de UCEPROTECT. Listan IPs masivamente para cobrar por el desliste.
