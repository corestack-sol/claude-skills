---
name: adsense-expert
description: Use when the user asks to optimize, configure, diagnose, or make decisions about Google AdSense — ad format choice, placement strategy, policy compliance, ads.txt, EU/UK/CH consent (CMP), performance metrics, earnings drops, or blocking controls. Built from official Google AdSense Help Center sources, not training-data guesswork.
---

# Google AdSense — Guía Experta

Conocimiento operativo de AdSense, construido a partir de documentación oficial de Google (no memoria/entrenamiento). Cubre estrategia de formato/colocación, políticas que pueden suspender una cuenta, ads.txt, consentimiento EEE/UK/Suiza, métricas de reportes, diagnóstico de caídas de ingresos, y controles de bloqueo — con la ruta exacta de menú en la consola para cada configuración.

## Limitación dura — no negociable

**No hay forma de loguearse en la consola de AdSense (adsense.google.com) desde acá.** Sin OAuth, sin sesión de navegador con la cuenta de Google del usuario, sin credenciales guardadas. Todo lo que en la consola se hace con clicks (activar Auto ads, bloquear categorías, crear un bloque de anuncios, ver reportes) requiere que el USUARIO lo haga — esta skill da la ruta de menú exacta cada vez, no lo ejecuta.

Lo que SÍ se puede hacer sin el usuario: todo lo que sea código/archivos de un repo (componente de anuncio, `ads.txt`, CSP, env vars) — eso se hace directo, sin pedir permiso de portal.

## Fuente de investigación ampliable

Notebook NotebookLM "Google AdSense — Guia Experta" (10 fuentes oficiales de support.google.com, ver `references/fuentes.md`) — para preguntas puntuales que esta skill no cubra, usar la skill `notebooklm`:
```
python scripts/run.py ask_question.py --notebook-id "google-adsense-—-guia-experta" --question "..."
```
Preguntas cortas y específicas responden rápido (segundos). Preguntas gigantes de síntesis (10+ subtemas en una sola pregunta) pueden tardar 10+ minutos en esa UI — partir en preguntas chicas, no mandar una sola pregunta enorme.

## 1. Auto ads vs colocación manual

**Auto ads**: un solo `<script>` en el sitio, Google escanea las páginas y decide dónde poner anuncios (puede insertar anchor/vignette/overlay ads que no existen como bloques manuales). Menos control de diseño, pero se adapta solo y Google optimiza la colocación con IA.

**Colocación manual** (bloques de anuncios, lo que usa Vive Villahermosa): control total de dónde, cuándo y con qué formato aparece cada anuncio — necesario cuando el sitio ya tiene un diseño cuidado y no se quiere que Google decida insertar anuncios en lugares que rompan la UX. Trade-off: hay que mantenerlo manualmente si el layout cambia.

**Recomendación para Vive Villahermosa**: seguir con manual — ya hay 5 espacios elegidos a propósito (guías, propiedades, Home, zonas, sidebar), evitando `/mapa`, `/dashboard/**`, mensajería y `/publicar`. Auto ads podría insertar anuncios en esas zonas protegidas sin que el código lo controle.

Menú: **Anuncios → Por sitio → [sitio] → Auto ads** (activar/desactivar, o "Auto optimize" para que Google experimente automáticamente con % de tráfico — configurable, incluye poder bloquear qué tipos de experimento corre).

## 2. Elegir formato de anuncio

| Formato | Cuándo usarlo | Atributos clave |
|---|---|---|
| **Display** (gráficos) | Espacios genéricos: sidebar, quiebres de sección, bloques standalone | `data-ad-format="auto"` + `data-full-width-responsive="true"` |
| **In-feed** | Insertado DENTRO de una lista/grid de tarjetas (imita el estilo de la tarjeta vecina) | `data-ad-format="fluid"` + `data-ad-layout-key` (valor que da el panel, único por bloque) |
| **In-article** | Insertado dentro/al final de contenido editorial tipo artículo, imita tipografía del texto | `data-ad-format="fluid"` + `data-ad-layout="in-article"` |

Ancho mínimo de un in-article: 250px. Los 3 formatos permiten mostrar anuncios display seleccionados por Google que "encajen" en el espacio disponible.

Mapeo ya aplicado en Vive Villahermosa: guías→in-article, propiedades (grid, cada 9 tarjetas)→in-feed, Home/zonas/sidebar→display. Ver `src/lib/ads.ts` y `src/components/ads/AdSlot.tsx`.

## 3. Colocación — maximizar RPM sin dañar UX/Core Web Vitals

- Anuncios en línea con contenido real, en áreas ricas en contenido — nunca en zonas vacías solo para "rellenar".
- Preferir bloques **responsive** sobre tamaño fijo — se adaptan a cada dispositivo (crítico mobile-first).
- No sobrecargar una sola zona con demasiados anuncios.
- Reservar `min-height` en el contenedor antes de que el anuncio real cargue — evita CLS (layout shift) cuando Google resuelve el tamaño real. `AdSlot.tsx` ya hace esto (prop `minHeight`, 120-250px según espacio).
- El script se carga con `strategy="afterInteractive"` (next/script) — no bloquea el render inicial ni el LCP.

## 4. Políticas que pueden suspender la cuenta

- **Prohibido** incentivar o inducir clics de cualquier forma — ni siquiera indirectamente (ej. "apoya este sitio haciendo clic en los anuncios").
- **Prohibido** que un anuncio se pueda confundir con contenido propio del sitio: no imitar formato de menú, navegación, o links de descarga.
- **Prohibido** alinear imágenes con anuncios o que el contenido cercano imite su formato — cualquier técnica que aumente clics accidentales.
- Solo se permite etiquetar bloques como **"Anuncios" o "Publicidad"** — nunca algo engañoso. (Ya implementado: `AdSlot.tsx` pone "Publicidad" arriba de cada bloque real.)
- Requisitos de contenido: el sitio necesita contenido original suficiente — un sitio recién lanzado con poco contenido puede no aprobar revisión.

Fuentes: [Ad placement policies](https://support.google.com/adsense/answer/1346295), [Best practices for ad placement](https://support.google.com/adsense/answer/1282097), [AdSense policies: a beginner's guide](https://support.google.com/adsense/answer/23921).

## 5. ads.txt

`ads.txt` (Authorized Digital Sellers, iniciativa de IAB Tech Lab) declara qué vendedores (ej. AdSense) están autorizados a vender el inventario publicitario del sitio — previene inventario falsificado vendido bajo el nombre del dominio sin permiso. **No es obligatorio pero Google lo recomienda fuertemente** — algunos compradores solo pujan por inventario clasificado como autorizado vía ads.txt.

Formato: un archivo de texto plano en la RAÍZ del dominio (`https://dominio.com/ads.txt`, nunca en subcarpeta), una línea por vendedor autorizado:
```
google.com, pub-XXXXXXXXXXXXXXXX, DIRECT, f08c47fec0942fa0
```
El último campo (`f08c47fec0942fa0`) es el Google Certification Authority ID — fijo, igual para todos los publishers de Google.

Estado en este proyecto (2026-09-08): agregado en `public/ads.txt` de ambos repos (Vive Villahermosa y Corestack) con `pub-9968456021033218` — pendiente deploy/verificación en vivo.

Fuente: [Ads.txt guide](https://support.google.com/adsense/answer/12171612).

## 6. Consentimiento EEE / Reino Unido / Suiza (CMP)

Desde el 16 de enero de 2024, para servir anuncios en EEE/UK, Google exige usar una **Consent Management Platform (CMP) certificada por Google** e integrada con el IAB Transparency and Consent Framework (TCF) — o construir la propia solución de consentimiento, o usar una CMP no certificada bajo términos específicos. Google audita sitios periódicamente (revisores visitan el sitio como usuario real).

**Para Vive Villahermosa**: audiencia 100% Tabasco/México, esto es prescindible en la práctica — no bloquea nada, solo evita perder el pequeño % de ingreso de tráfico orgánico incidental de esas regiones. Si se quiere cubrir igual, la opción sin código es activar el **mensaje de consentimiento nativo** que ofrece el propio panel de AdSense (no requiere tocar el repo).

Menú: **Privacidad y mensajería → Mensajes de la UE** (o el nombre vigente en el panel — Google renombra esta sección con frecuencia, buscar "consent" si no aparece igual).

Fuentes: [EU user consent policy](https://support.google.com/adsense/answer/7670013), [Consent management requirements EEA/UK/CH](https://support.google.com/adsense/answer/13554116).

## 7. Métricas clave

- **RPM** (Revenue per Mille — ingreso estimado por cada 1000 unidades): existen 3 variantes — **Page RPM** (por 1000 páginas vistas), **Impression RPM** (por 1000 impresiones de anuncio), **Ad request RPM** (por 1000 solicitudes de anuncio). Page RPM es la métrica más representativa de "cuánto genera el sitio en total".
- **CTR** (page click-through rate) = clics en anuncios ÷ páginas vistas.
- **CPC** = costo/ingreso por clic.
- No hay un "rango saludable" universal publicado por Google — varía muchísimo por nicho/geografía/formato. Lo útil es la TENDENCIA propia en el tiempo, no compararse contra un benchmark genérico.

Menú: **Informes** (panel izquierdo) → elegir métricas con "Pick the metrics for your report".

Fuentes: [Revenue per thousand impressions (RPM)](https://support.google.com/adsense/answer/190515), [Overview of AdSense Reports](https://support.google.com/adsense/answer/9831227).

## 8. Diagnosticar una caída de ingresos

Al investigar una caída: agregar **CTR, CPC e impresiones** al reporte y aislar cuál métrica específica causó la baja (no asumir "menos tráfico" sin verificar) — una caída de RPM con tráfico estable casi siempre es CPC o fill rate, no audiencia.

Menú: **Informes → Troubleshoot a drop in earnings** (Google tiene una herramienta guiada para esto en el panel).

Fuente: [Troubleshoot a drop in earnings](https://support.google.com/adsense/answer/6334002).

## 9. Blocking controls / brand safety

Dos tipos de categoría a bloquear:
- **General categories** (rubros: ej. citas, apuestas) — todas permitidas por defecto, se bloquean manualmente.
- **Sensitive categories** — dos subtipos: **Standard** (permitidas por defecto) y **Restricted** (bloqueadas por defecto).

**Recomendación para Vive Villahermosa** (audiencia familiar buscando casa): bloquear como mínimo *Citas/Dating*, *Apuestas/Gambling*, *Alcohol* en categorías sensibles — no encajan con el contexto de un sitio inmobiliario familiar.

Menú: **Anuncios → Brand safety → Blocking controls → Manage general categories / Manage sensitive categories**. Revisión ad-hoc de anuncios individuales: **Ad review center** (mismo submenú).

Fuentes: [Block sensitive categories](https://support.google.com/adsense/answer/164131), [Block general categories](https://support.google.com/adsense/answer/186376), [Brand safety overview](https://support.google.com/adsense/answer/1059482).

## 10. Estado de cuenta — Vive Villahermosa (2026-09-08)

- Publisher ID: `ca-pub-9968456021033218` — cuenta confirmada con producto **AdSense (web)** activo, no solo AdMob (confirmado al obtener un bloque de anuncios in-article real del panel).
- 5 bloques reales ya creados y cargados en `.env.local`:
  - Guías (in-article): `8178563798`
  - Propiedades in-feed: `3720444033`
  - Home/zonas/sidebar (display, mismo bloque reusado): `5232353752`
- Dominio raíz verificado para AdSense: `corestacksolutions.com.mx` (Google exige raíz, no el subdominio `vivevillahermosa.corestacksolutions.com.mx` donde corren los anuncios reales) — script de verificación ya en `C:\Projects\Websites\Corestack\index.html`.
- `ads.txt` agregado en `public/` de ambos repos, pendiente deploy/verificación.
- Sitio en revisión de Google al momento de escribir esto — todos los bloques muestran `data-ad-status="unfilled"` (esperado hasta aprobación, no es bug).

## Referencias

- `references/fuentes.md` — lista completa de las 10 fuentes con URL, para citar o profundizar.
