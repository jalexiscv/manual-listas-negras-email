# Manual Completo de Listas Negras de Correo Electrónico

## El Libro Definitivo sobre DNSBLs, Reputación de Remitente y Entregabilidad

---

**Autor:** Jose Alexis Correa Valencia  
**Versión:** 1.0  
**Fecha de publicación:** 15 de Julio de 2014

---

## Estructura del Manual

Este manual está organizado en quince capítulos y tres apéndices. Haz clic en cualquier capítulo para leerlo.

| # | Capítulo | Enlace |
|---|----------|--------|
| 01 | Introducción al mundo de las listas negras | [Leer →](01-introduccion.md) |
| 02 | Fundamentos técnicos del correo electrónico | [Leer →](02-fundamentos.md) |
| 03 | ¿Qué es una lista negra? | [Leer →](03-que-es-una-lista-negra.md) |
| 04 | Tipos de listas negras | [Leer →](04-tipos-de-listas-negras.md) |
| 05 | Las principales DNSBL del mundo | [Leer →](05-principales-dnsbl.md) |
| 06 | Cómo funcionan las DNSBL por dentro | [Leer →](06-como-funcionan.md) |
| 07 | Motivos comunes para ser incluido | [Leer →](07-motivos-de-inclusion.md) |
| 08 | Cómo detectar si estás en una lista negra | [Leer →](08-como-detectar.md) |
| 09 | SPF, DKIM y DMARC — El santo grial de la autenticación | [Leer →](09-spf-dkim-dmarc.md) |
| 10 | Estrategias para salir de una lista negra | [Leer →](10-estrategias-desliste.md) |
| 11 | Prevención: buenas prácticas de envío | [Leer →](11-prevencion.md) |
| 12 | Monitoreo continuo y herramientas | [Leer →](12-monitoreo.md) |
| 13 | Listas blancas y reputación positiva | [Leer →](13-listas-blancas.md) |
| 14 | Estudios de caso reales | [Leer →](14-estudios-de-caso.md) |
| 15 | Glosario de términos | [Leer →](15-glosario.md) |
| A | Apéndice A: Tabla comparativa de DNSBL | [Leer →](apendice-a-dnsbl-comparativa.md) |
| B | Apéndice B: Códigos de error SMTP | [Leer →](apendice-b-codigos-smtp.md) |
| C | Apéndice C: Scripts útiles para administradores | [Leer →](apendice-c-scripts.md) |

---

## Cómo usar este manual

Este manual ha sido diseñado para adaptarse a diferentes perfiles de lectores:

- **🔰 Principiantes:** Si estás empezando en la administración de servidores de correo, lee los **capítulos 1 al 4** en orden. Ellos establecen los fundamentos conceptuales y técnicos necesarios para entender el resto del libro.

- **🛠️ Administradores de sistemas:** Enfócate en los **capítulos 5 al 8** (cómo funcionan las listas negras) y **9 al 11** (autenticación, desliste y prevención). También te serán de gran utilidad los scripts del Apéndice C.

- **🎯 Especialistas en marketing:** Los **capítulos 11 (prevención)** y **14 (casos reales)** son lectura obligada. También te recomendamos el capítulo 9 sobre autenticación.

- **📖 Consulta rápida:** El **capítulo 15 (glosario)** y los **apéndices A y B** están diseñados para consulta sobre la marcha cuando necesitas recordar un término o interpretar un código de error.

> Cada capítulo es autocontenido. Puedes leerlos en el orden que prefieras según tu necesidad del momento.

---

## Vista General del Contenido

### Bloque 1: Fundamentos (Capítulos 1-4)
Los primeros cuatro capítulos sientan las bases: qué es una lista negra, cómo funciona el correo electrónico, quién opera las listas y qué tipos existen.

### Bloque 2: Operativo (Capítulos 5-8)
Aquí se explica en detalle cómo funcionan las listas negras en la práctica: las principales DNSBL del mundo, los motivos por los que una IP termina listada, y las herramientas para detectar listados.

### Bloque 3: Preventivo (Capítulos 9-11)
SPF, DKIM y DMARC son los pilares de la autenticación. Este bloque cubre cómo configurarlos, cómo salir de una lista negra cuando ya estás en ella, y las mejores prácticas para evitar futuros listados.

### Bloque 4: Avanzado (Capítulos 12-14)
Herramientas de monitoreo continuo, listas blancas y reputación positiva, y seis estudios de caso reales documentados con causas, soluciones y lecciones aprendidas.

### Bloque 5: Referencia (Capítulo 15 y Apéndices)
Glosario completo de términos técnicos, tabla comparativa de DNSBL, guía de códigos de error SMTP, y scripts listos para usar.

---

## Convenciones

A lo largo de todo el manual se utilizan las siguientes convenciones visuales:

| Símbolo | Significado |
|---------|-------------|
| `Código` | Comandos de terminal, configuraciones, nombres de archivo |
| **Negrita** | Términos clave en su primera aparición |
| > | Citas, notas importantes, advertencias |
| 📌 **Dato clave** | Información esencial para recordar |
| ⚠️ **Advertencia** | Peligros y errores comunes |
| 💡 **Consejo** | Trucos y recomendaciones de expertos |
| ```bloques``` | Fragmentos de código listos para copiar y ejecutar |

---

## Enlaces Rápidos

- 📘 **Repositorio en GitHub:** [jalexiscv/manual-listas-negras-email](https://github.com/jalexiscv/manual-listas-negras-email)
- 📧 **Contacto:** jalexiscv@gmail.com
- 🌐 **Perfil GitHub:** [@jalexiscv](https://github.com/jalexiscv)

---

## Licencia

Distribuido bajo la Licencia **MIT**. Eres libre de usar, copiar, modificar y distribuir este contenido, siempre que se incluya el crédito al autor original.

---

*"El correo electrónico no está muerto. Está en una guerra constante contra el abuso. Las listas negras son los centinelas en las murallas. Respétalos, entiéndelos y trabajarán contigo, no contra ti."*

---

**Jose Alexis Correa Valencia** — Colombia 🇨🇴
