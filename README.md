# root@j0su · Blog de Red Team

Blog personal sobre **Red Teaming, seguridad ofensiva y write-ups técnicos**, escrito en español e inglés.

### 🔗 Web: **https://josupalacios99.github.io/blog/**

---

## Sobre el blog

Apuntes, guías y casos prácticos de seguridad ofensiva: desde fundamentos de Red Teaming
hasta write-ups técnicos (evasión de AV/EDR, abuso de credenciales, Active Directory…).
Contenido divulgativo y orientado a demostrar impacto real, no a recetas de ataque.

## Stack

- [Hugo](https://gohugo.io/) (extended) + tema [PaperMod](https://github.com/adityatelange/hugo-PaperMod)
- Despliegue automático en **GitHub Pages** vía GitHub Actions
- Bilingüe **ES / EN**
- Estética propia: terminal, acento rojo y viñetas tipo cómic

## Desarrollo local

```bash
# servidor de desarrollo (incluye borradores)
hugo server -D

# build de producción (lo que se publica)
hugo
```

## Estructura

```
content/posts/   Artículos (page bundles: index.md + index.en.md + imágenes)
layouts/         Plantillas y parciales personalizados
assets/css/      Estilos propios (extended/custom.css)
i18n/            Cadenas ES/EN
static/          Recursos estáticos (favicon, etc.)
```

## Notas

- Los artículos marcados como `draft: true` no se publican; se previsualizan en local con `hugo server -D`.
- Cada post vive como *page bundle*, con su versión ES (`index.md`) e inglesa (`index.en.md`) compartiendo imágenes.

---

Hecho por [j0su](https://github.com/JosuPalacios99) · [LinkedIn](https://www.linkedin.com/in/josu-palacios-lartain-454702214/)
