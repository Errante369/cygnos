# Configuración Polyglot + Chirpy para Cygnos

## Resumen de cambios necesarios

1. **Fork del tema Chirpy** — obligatorio. El tema como gem no permite modificar layouts/includes
2. **Instalar Polyglot** — plugin de Jekyll para i18n
3. **Modificar `_config.yml`** — añadir configuración de idiomas
4. **Reemplazar `site.lang` → `site.active_lang`** en todos los archivos del tema
5. **Añadir language switcher** en `_includes/sidebar.html`
6. **Crear archivos de locale** para español
7. **Estructura de posts bilingües**

---

## Paso 1: Fork de Chirpy

Repo original: https://github.com/cotes2020/jekyll-theme-chirpy

```bash
# Fork desde GitHub (manual en la web)
# Luego clonar tu fork:
cd /home/kali/
git clone https://github.com/Errante369/jekyll-theme-chirpy.git
```

Después reemplazar en el Gemfile de cygnos:
```ruby
# ANTES:
gem "jekyll-theme-chirpy"

# DESPUÉS:
gem "jekyll-theme-chirpy", path: "../jekyll-theme-chirpy"
```

O si prefieres copiar los archivos del tema directamente al proyecto cygnos
(en lugar de usar gem), hay que copiar:
- `_layouts/`
- `_includes/`
- `_sass/`
- `assets/`
- `_data/`

---

## Paso 2: Gemfile — añadir Polyglot

```ruby
group :jekyll_plugins do
  gem "jekyll-polyglot"
end
```

Luego: `bundle install`

---

## Paso 3: _config.yml completo

```yaml
lang: es                          # idioma default
timezone: America/Lima
title: Cygnos
tagline: Offensive Security · AI · Technology
description: >-
  Bilingual blog about offensive security, AI red teaming, and technology.
  Red Team consultant sharing practical knowledge.
url: "https://errante369.github.io"
baseurl: "/cygnos"
permalink: /posts/:title/

github:
  username: Errante369

social:
  name: Cygnos
  links:
    - https://github.com/Errante369

theme: jekyll-theme-chirpy
plugins:
  - jekyll-seo-tag
  - jekyll-feed
  - jekyll-polyglot

# ---- Polyglot config ----
languages: ["es", "en"]
default_lang: "es"
exclude_from_localization: ["javascript", "images", "css", "public", "sitemap"]
parallel_localization: true

defaults:
  - scope:
      path: ""
      type: posts
    values:
      layout: post
      comments: false
      toc: true

kramdown:
  syntax_highlighter: rouge
  syntax_highlighter_opts:
    css_class: highlight

compress_html:
  clippings: all
  comments: all
  endings: all
  profile: false
  blanklines: false
  ignore:
    envs: []

exclude:
  - "*.gem"
  - "*.gemspec"
  - docs/
  - README.md
  - LICENSE
```

---

## Paso 4: Reemplazar site.lang → site.active_lang

En TODOS los archivos del tema (layouts, includes), reemplazar:

```
site.lang          → site.active_lang
site.data.locales[site.lang]  → site.data.locales[site.active_lang]
```

Comando para hacerlo:
```bash
cd /home/kali/jekyll-theme-chirpy/
# Buscar todas las ocurrencias:
grep -rn "site\.lang" _layouts/ _includes/ _data/ | grep -v "active_lang"
# Reemplazar (verificar antes con grep):
find _layouts/ _includes/ -type f -name "*.html" -exec sed -i 's/site\.lang/site.active_lang/g' {} +
```

---

## Paso 5: Archivos de locale para español

El tema tiene `_data/locales/` con archivos por idioma. Hay que crear:

**`_data/locales/es-ES.yml`** (o copiar de `en.yml` y traducir):

El tema Chirpy ya incluye locale para español. Verificar que existe en:
`_data/locales/es.yml`

Si no existe, copiar `en.yml` y traducir las strings.

---

## Paso 6: Language switcher en sidebar

Añadir a `_includes/sidebar.html` (dentro del div `sidebar-bottom`):

```html
<div class="sidebar-bottom d-flex flex-wrap align-items-center w-100">
  <!-- Language switcher -->
  <div class="lang-div d-flex flex-wrap w-100">
    {% for tongue in site.languages %}
      {% if tongue == site.active_lang %}
        <a class="lang-name" style="font-weight: bold; text-decoration: none;">
          {{ tongue | upcase }}
        </a>
      {% else %}
        <a class="lang-name" href="{% static_href /{{ tongue }}{{ page.url }} %}">
          {{ tongue | upcase }}
        </a>
      {% endif %}
    {% endfor %}
  </div>
</div>
```

---

## Paso 7: Estructura de posts bilingües

Cada post tiene DOS archivos con el mismo nombre en diferentes carpetas:

```
_posts/
  2026-04-15-bienvenido-a-cygnos.md     # lang: es (default, raíz)
  en/
    2026-04-15-bienvenido-a-cygnos.md   # lang: en (subdirectorio)
```

**Post español** (`_posts/2026-04-15-bienvenido-a-cygnos.md`):
```markdown
---
title: "Bienvenido a Cygnos"
lang: es
page_id: bienvenido-cygnos
date: 2026-04-15
categories: [blog]
tags: [intro]
layout: post
toc: false
---

## Bienvenido

Este es **Cygnos** — un espacio bilingüe sobre seguridad ofensiva, AI red teaming...
```

**Post inglés** (`_posts/en/2026-04-15-bienvenido-a-cygnos.md`):
```markdown
---
title: "Welcome to Cygnos"
lang: en
page_id: bienvenido-cygnos
date: 2026-04-15
categories: [blog]
tags: [intro]
layout: post
toc: false
---

## Welcome

This is **Cygnos** — a bilingual space for offensive security research...
```

**URLs resultantes:**
- Español: `https://errante369.github.io/cygnos/posts/bienvenido-a-cygnos/`
- Inglés: `https://errante369.github.io/cygnos/en/posts/bienvenido-a-cygnos/`

---

## Resultado final

- Toggle de idioma en el sidebar (siempre visible)
- URLs separadas por idioma (SEO correcto)
- `hreflang` automático (Polyglot lo maneja)
- Fallback: si no hay traducción, muestra el español
- Para posts nuevos: crear 2 archivos con mismo nombre, diferente `lang`

---

## Referencias

- Polyglot repo: https://github.com/untra/polyglot
- Guía Chirpy + Polyglot: https://aturret.space/posts/Customize-Bilingual-Blog-with-Chirpy-Theme/
- Chirpy getting started: https://chirpy.cotes.page/posts/getting-started/
- Polyglot v1.12 (ene 2026): compatible con Chirpy, fix de glob patterns
