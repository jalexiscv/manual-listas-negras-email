# Manual Completo de Listas Negras de Correo Electrónico

## El Libro Definitivo sobre DNSBLs, Reputación de Remitente y Entregabilidad

---

**Autor:** Jose Alexis Correa Valencia
**Versión:** 1.0
**Fecha de publicación:** 15 de Julio de 2014

---

## Estructura del Manual

Este manual está organizado en quince capítulos y tres apéndices.

| # | Archivo | Título |
|---|---------|--------|
| 00 | `README.md` | Índice general y estructura |
| 01 | `01-introduccion.md` | Introducción al mundo de las listas negras |
| 02 | `02-fundamentos.md` | Fundamentos técnicos del correo electrónico |
| 03 | `03-que-es-una-lista-negra.md` | ¿Qué es una lista negra? |
| 04 | `04-tipos-de-listas-negras.md` | Tipos de listas negras |
| 05 | `05-principales-dnsbl.md` | Las principales DNSBL del mundo |
| 06 | `06-como-funcionan.md` | Cómo funcionan las DNSBL por dentro |
| 07 | `07-motivos-de-inclusion.md` | Motivos comunes para ser incluido |
| 08 | `08-como-detectar.md` | Cómo detectar si estás en una lista negra |
| 09 | `09-spf-dkim-dmarc.md` | SPF, DKIM y DMARC — El santo grial de la autenticación |
| 10 | `10-estrategias-desliste.md` | Estrategias para salir de una lista negra |
| 11 | `11-prevencion.md` | Prevención: buenas prácticas de envío |
| 12 | `12-monitoreo.md` | Monitoreo continuo y herramientas |
| 13 | `13-listas-blancas.md` | Listas blancas y reputación positiva |
| 14 | `14-estudios-de-caso.md` | Estudios de caso reales |
| 15 | `15-glosario.md` | Glosario de términos |
| A | `apendice-a-dnsbl-comparativa.md` | Apéndice A: Tabla comparativa de DNSBL |
| B | `apendice-b-codigos-smtp.md` | Apéndice B: Códigos de error SMTP |
| C | `apendice-c-scripts.md` | Apéndice C: Scripts útiles para administradores |

---

## Cómo usar este manual

- **Principiantes:** Lean los capítulos 1 al 4 para los fundamentos.
- **Administradores:** Enfoquen en capítulos 5-8 (operativo) y 9-11 (preventivo).
- **Expertos:** Capítulos 12-14 y apéndices para referencia avanzada.
- **Consulta rápida:** Capítulo 15 (glosario) y apéndices.

> Cada capítulo es autocontenido. Puedes leerlos en cualquier orden según tu necesidad.

## Convenciones

| Símbolo | Significado |
|---------|-------------|
| `Código` | Comandos, fragmentos de configuración |
| **Negrita** | Términos clave en su primera aparición |
| > | Citas, notas importantes, advertencias |
| 📌 **Dato clave** | Información esencial para recordar |
| ⚠️ **Advertencia** | Peligros y errores comunes |
| 💡 **Consejo** | Trucos y recomendaciones de expertos |


## 🤝 Contribución

Este proyecto es **Open Source** y vive gracias a la comunidad. ¡Tus contribuciones son bienvenidas!

### Cómo Contribuir

1. **Fork** del repositorio
2. **Crea tu rama** de característica
   ```bash
   git checkout -b feature/nueva-funcionalidad
   ```
3. **Asegúrate de ejecutar los tests**
   ```bash
   composer test
   ```
4. **Haz commit de tus cambios**
   ```bash
   git commit -m 'Add: Nueva funcionalidad increíble'
   ```
5. **Push a tu rama**
   ```bash
   git push origin feature/nueva-funcionalidad
   ```
6. **Abre un Pull Request**

### Directrices de Contribución

- ✅ Sigue los estándares PSR-12
- ✅ Mantén el tipado estricto (`declare(strict_types=1)`)
- ✅ Documenta todas las funciones públicas
- ✅ Agrega tests para nuevas funcionalidades
- ✅ Actualiza la documentación relevante

### Áreas que Necesitan Ayuda

- 📝 Mejoras en documentación
- 🧪 Tests unitarios y de integración
- 🎨 Nuevos componentes de Bootstrap
- 🔧 Implementación de nuevos frameworks (Tailwind, Material)
- 🌍 Traducciones de documentación
- 🐛 Reportes de bugs

---

## 🤝 Soporte y Comunidad

### ¿Necesitas Ayuda?

- 📖 **Documentación**: Lee el [README completo](README.md) y [ARCHITECTURE.md](ARCHITECTURE.md)
- 🐛 **Reportar bugs**: Abre un [issue en GitHub](https://github.com/jalexiscv/Html/issues)
- 💡 **Solicitar funcionalidades**: Usa las [GitHub Discussions](https://github.com/jalexiscv/Html/discussions)
- 📧 **Contacto directo**: jalexiscv@gmail.com

### Comunidad

- **Discusiones**: Únete a las conversaciones en GitHub Discussions
- **Contribuciones**: Revisa los [issues etiquetados como "good first issue"](https://github.com/jalexiscv/Html/labels/good%20first%20issue)

---

## 📜 Licencia

Distribuido bajo la Licencia **MIT**. Ver [LICENSE](LICENSE) para más información.

> La licencia MIT te permite usar, copiar, modificar, fusionar, publicar, distribuir, sublicenciar y/o vender copias del software sin restricciones, siempre que se incluya el aviso de copyright.

---

## 👨‍💻 Autor

**Jose Alexis Correa Valencia**
*Full Stack Developer & Software Architect*

Con más de 25 años de experiencia en desarrollo de software empresarial, especializado en arquitecturas escalables y soluciones PHP modernas.

- **GitHub**: [@jalexiscv](https://github.com/jalexiscv)
- **LinkedIn**: [Jose Alexis Correa Valencia](https://www.linkedin.com/in/jalexiscv/)
- **Email**: jalexiscv@gmail.com
- **Ubicación**: Colombia 🇨🇴

---

## ❤️ Donaciones

Si Frontend Framework te ha ayudado a ti o a tu negocio, considera apoyar su desarrollo y mantenimiento continuo.

| Método | Detalles |
|--------|----------|
| **PayPal** | [jalexiscv@gmail.com](https://www.paypal.com/paypalme/anssible) |
| **Nequi (Colombia)** | `3117977281` |

### Beneficios de tu Soporte

Tu donación ayuda a:
- ⚡ Acelerar el desarrollo de nuevas funcionalidades
- 📚 Crear más documentación y ejemplos
- 🧪 Mejorar la cobertura de tests
- 🎨 Implementar soporte para más frameworks
- 🌍 Mantener el proyecto activo y actualizado

*¡Gracias por tu apoyo!* 🙏

---

<div align="center">

**Desarrollado con ❤️ para la comunidad PHP**

[⬆ Volver arriba](#frontend-framework)

</div>
