# Capítulo 10: Estrategias para Salir de una Lista Negra

[← Anterior](09-spf-dkim-dmarc.md) | [Índice](00-indice.md) | [Siguiente →](11-prevencion.md)

---

## 10.1 Diagnóstico Inicial

1. Identificar qué lista te listó
2. Entender el motivo (código de retorno)
3. Corregir la causa raíz
4. Documentar las medidas tomadas

## 10.2 Spamhaus

### SBL (Spamhaus Block List)
```bash
dig +short <IP>.zen.spamhaus.org
# 127.0.0.2 = spam directo
```
- IP comprometida: limpiar servidor, cambiar contraseñas
- Lista comprada: eliminarla, implementar double opt-in
- Desliste: https://www.spamhaus.org/lookup/ (gratuito, 24-48h)

### PBL (Policy Block List)
- Usar relay SMTP autenticado (recomendado)
- Contratar IP dedicada

## 10.3 SpamCop
- **Automático:** Si cesan reportes, IP sale en 24-48h
- **Manual:** Formulario en su web

## 10.4 Barracuda BRBL
- Desliste automático al cesar el abuso

## 10.5 Cambio de IP

**Cuándo sí:** IP con historial largo de abuso, listas extorsivas.
**Cuándo no:** Problema subyacente no resuelto, reputación de dominio intacta.

## 10.6 Warm-up de IP

| Días | Volumen |
|------|---------|
| 1-3 | 50/día |
| 4-7 | 200/día |
| 8-14 | 1.000/día |
| 15-21 | 5.000/día |
| 22-30 | 10.000/día |

Enviar primero a usuarios más activos. Aumentar max 20-30% por día.

## 10.7 Checklist de Desliste

```
[ ] Identifique todas las listas donde estoy
[ ] Entendi la causa raiz de cada listado
[ ] Corregi la causa raiz
[ ] Verifique que el servidor no este comprometido
[ ] Configure SPF correctamente
[ ] Configure DKIM correctamente
[ ] Configure DMARC (al menos p=none)
[ ] Verifique el PTR (rDNS)
[ ] Asegure formularios web
[ ] Implemente rate limiting SMTP
[ ] Elimine listas compradas o raspadas
[ ] Implemente double opt-in
[ ] Solicite desliste en cada lista
[ ] Monitoree durante 48-72 horas posteriores
```
