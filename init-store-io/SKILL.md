---
name: init-store-io
description: "Inicializa una tienda VTEX IO basada en Fast Template LATAM. Tiene 3 fases: Fase 1 levanta sitio funcional (solo VENDOR + PROYECTO), Fase 2 aplica estilos y contenido, Fase 3 customs. Usar cuando el usuario dice /init-store-io, 'inicializar tienda vtex io', 'levantar tienda io', 'setup store io'."
---

# Skill: Init Store — VTEX IO (Fast Template LATAM)

Inicializa una tienda VTEX IO basada en Fast Template LATAM en 3 fases independientes.

**Repo de scripts**: `git@bitbucket.org:vtex-infralatam/init-ftlat.git`
Se clona al inicio de Fase 1 y se elimina al final de Fase 2.

---

## Las 3 Fases

```
FASE 1 — LEVANTAR (sitio funcional)
  Datos: VENDOR, PROYECTO, tipo tienda, pais
  Resultado: tienda corriendo en workspace fasttemplate, linkeada

FASE 2 — ESTILOS Y CONTENIDO
  Datos: MARCA, colores, contacto, fuentes, imagenes
  Resultado: branding aplicado, mails subidos, init-ftlat eliminado

FASE 3 — CUSTOMS
  Datos: integraciones, envio gratis, newsletter
  Resultado: tienda personalizada lista para QA
```

Cada fase se puede ejecutar por separado. Fase 2 requiere Fase 1 completada. Fase 3 requiere Fase 2 completada.

---

## Prerrequisitos del Sistema

Verificar antes de arrancar:

```bash
node --version && yarn --version && vtex --version && ssh -T git@bitbucket.org
```

- **Node.js** >= 14
- **Yarn** global
- **VTEX CLI** instalado
- **Acceso SSH a Bitbucket**

Si algo falta → informar y no continuar.

---

# FASE 1 — LEVANTAR

Objetivo: tienda VTEX IO funcional corriendo en workspace `fasttemplate`.

## F1 - Paso 0: Recoger datos

Solo 4 campos. Preguntar asi:

```
Vamos a levantar la tienda VTEX IO.

1. VENDOR — Account VTEX (ej: "mitienda")
2. PROYECTO — Acronimo Bitbucket (ej: "ATKAR", "VANUY")
```

Usar `AskUserQuestion` para:

**Tipo de tienda** (para Easy Setup):
- Opciones: "General (Recomendado)", "Moda", "Tecnologia", "Alimentacion"

**Pais**:
- Opciones: "ARG", "BRA", "COL", "CHL"

**Validar**:
- VENDOR: lowercase, sin espacios
- PROYECTO: uppercase, sin espacios

WORKSPACE se fija en `fasttemplate` (no preguntar).
NOMBRE_PROYECTO = VENDOR (no preguntar).

## F1 - Paso 0b: Confirmar

```
=== Resumen Fase 1 ===
Vendor: {VENDOR}
Proyecto: {PROYECTO}
Workspace: fasttemplate
Tipo tienda: {TIPO}
Pais: {PAIS}

Se va a clonar repos, instalar deps, poblar catalogo y linkear.
Confirmas?
```

## F1 - Paso 1: Crear carpeta y clonar init-ftlat

```bash
mkdir -p {VENDOR}
cd {VENDOR}
git clone git@bitbucket.org:vtex-infralatam/init-ftlat.git
cd init-ftlat
```

**Verificar**: init-ftlat/ existe con scripts.
**Este paso es invisible para el usuario** — solo mostrar si falla.

## F1 - Paso 2: VTEX Login

```bash
vtex login {VENDOR} -w master
```

Avisar al usuario que complete login en browser.
Verificar con `vtex whoami`.

## F1 - Paso 3: Instalar y ejecutar Easy Setup

```bash
vtex install infralatam.easy-setup@0.x
```

Ejecutar populate en background:

```bash
curl -s -X POST \
  "https://{VENDOR}.myvtex.com/_v/api/easy-setup/populate" \
  -H "Content-Type: application/json" \
  -H "VtexIdclientAutCookie: $(vtex local token)" \
  -d '{
    "storeType": "{STORE_TYPE}",
    "resources": ["brands","categories","products","prices","payments","logistics"],
    "country": "{COUNTRY}"
  }' > /tmp/easy-setup-result.json 2>&1 &
```

Mapeo storeType:
- General → `"General (recomendado)"`
- Moda → `"Moda"`
- Tecnologia → `"Tecnología"`
- Alimentacion → `"Alimentación"`

Continuar sin esperar. Verificar resultado al final de Fase 1.

## F1 - Paso 4: Indexar Intelligent Search

Informar al usuario:

```
Abrir: https://{VENDOR}.myvtex.com/admin/search/integration
→ Click boton para iniciar indexacion
IS tarda varios minutos. Continuamos mientras indexa.
```

Si Chrome DevTools MCP disponible → automatizar click.

## F1 - Paso 5: Clonar repos del proyecto

```bash
echo -e "{PROYECTO}\ns" | node clone-repos.js
```

**Verificar**: listar directorio padre, confirmar 10 repos.

Si algun repo falla:
```bash
git clone git@bitbucket.org:vtex-infralatam/{REPO}-{PROYECTO}.git
```

## F1 - Paso 6: Instalar dependencias

```bash
# Yarn en todos los repos
node configure-repo.js

# Peer dependencies
echo "s" | node install-peer-deps.js

# Wishlist
vtex install vtex.wish-list@1.x
```

## F1 - Paso 7: Cambiar a workspace y linkear

```bash
vtex use fasttemplate

node vtex-link.js
```

**Long-running**: Multiples procesos vtex link en background. Informar cuales completaron OK.

## F1 - Paso 8: Verificar Easy Setup

Leer `/tmp/easy-setup-result.json` e informar resultado.

Si fallo algun recurso → avisar. La app es idempotente, se puede reintentar.

## F1 - Resultado

```
═══════════════════════════════════════
Fase 1 completada ✓
Tienda: https://{VENDOR}.myvtex.com
Workspace: fasttemplate

Para branding → ejecutar Fase 2
═══════════════════════════════════════
```

---

# FASE 2 — ESTILOS Y CONTENIDO

Objetivo: aplicar branding (colores, fuentes, imagenes, marca) y subir mails.

**Requiere**: Fase 1 completada. CWD debe ser `{VENDOR}/init-ftlat/`.

## F2 - Paso 0: Recoger datos

### Bloque A — Marca y contacto

```
1. MARCA — Nombre visible (ej: "Mi Tienda")
2. PHONE_MARCA — Telefono de contacto
3. MAIL_TO — Email de contacto
```

### Bloque B — Colores

```
4. PRIMARY_COLOR   — Color principal (hex, ej: #2a2a2a)
5. SECONDARY_COLOR — Color secundario (hex, ej: #717171)
```

Usar `AskUserQuestion`:
- "Personalizar todos los colores o usar defaults basados en primary/secondary?"
- Opciones: "Usar defaults (Recomendado)", "Personalizar todos"

**Defaults (generados de primary/secondary):**
```
BG_COLOR               = #ffffff
PRE_HEADER_BG_COLOR    = {PRIMARY_COLOR}
PRE_HEADER_TEXT_COLOR  = #ffffff
PRE_FOOTER_BG_COLOR    = #ffffff
PRE_FOOTER_TEXT_COLOR  = {PRIMARY_COLOR}
PRE_FOOTER_BTN_BG_COLOR = {PRIMARY_COLOR}
PRE_FOOTER_BTN_TEXT_COLOR = #ffffff
FOOTER_BG_COLOR        = #ffffff
FOOTER_TEXT_COLOR       = {PRIMARY_COLOR}
CHECKOUT_HEADER_BG_COLOR = {PRIMARY_COLOR}
CHECKOUT_HEADER_TEXT_COLOR = #ffffff
CHECKOUT_FOOTER_BG_COLOR = #eeeeee
CHECKOUT_FOOTER_TEXT_COLOR = {SECONDARY_COLOR}
MAIL_HEADER_BG_COLOR   = {PRIMARY_COLOR}
MAIL_FOOTER_BG_COLOR   = #eeeeee
```

Si personaliza → pedir los 15 colores restantes. Validar todos hex `/^#[0-9a-fA-F]{6}$/`.

### Bloque C — Botones

Usar `AskUserQuestion` para BORDER_RADIUS_BTN:
- Opciones: "0px (sin redondeo)", "4px", "8px", "Otro"

### Bloque D — Assets

Fuentes:
- "Tenes las fuentes tipograficas listas?"
- Opciones: "Si, te doy la ruta", "Las agrego manualmente a init-ftlat/fonts/"

Si no hay fonts en `init-ftlat/fonts/`:

```
Necesito fuentes (.otf/.ttf/.woff/.woff2) organizadas por familia:
  fonts/
  ├── futura-std/
  │   ├── FuturaStd-Bold.otf
  │   └── FuturaStd-Regular.otf
  └── neue-montreal/
      └── NeueMontreal-Regular.woff2

Opciones:
1. Indicar ruta (las copio)
2. Agregarlas a init-ftlat/fonts/ y avisarme
```

**NO continuar sin fuentes.**

Imagenes:
- "Tenes las imagenes listas? (logos, heroes, iconos)"
- Opciones: "Si, te doy la ruta", "Las agrego manualmente", "Continuar sin imagenes"

Si usuario da ruta → copiar:
```bash
rm -rf init-ftlat/fonts/* && cp -r {RUTA_FUENTES}/* init-ftlat/fonts/
rm -rf init-ftlat/images/* && cp -r {RUTA_IMAGENES}/* init-ftlat/images/
```

Si agrega manual → esperar confirmacion.
Si continua sin imagenes → marcar en checklist final.

## F2 - Paso 0b: Confirmar

Mostrar resumen de todos los valores + colores + assets detectados. Pedir confirmacion.

## F2 - Paso 1: Subir fuentes

```bash
node upload-fonts.js
```

**Post-verificacion**: Leer config.json, confirmar PRIMARY_FONT y SECONDARY_FONT. Si orden incorrecto, preguntar al usuario.

## F2 - Paso 2: Llenar config.json

Leer config.json actual (preservar PRIMARY_FONT / SECONDARY_FONT de paso anterior).
Escribir todos los valores del usuario. URLs de imagenes quedan vacias.

**NO sobreescribir PRIMARY_FONT ni SECONDARY_FONT.**

## F2 - Paso 3: Subir imagenes

```bash
echo "s" | node upload-images-vtex.js
```

**Post-verificacion**: Leer config.json, verificar URLs de imagenes. Si LOGO_IMAGE o LOGO_BLANCO vacios → avisar.

## F2 - Paso 4: Ejecutar setup

```bash
node setup.js config.json
```

Reemplaza todas las variables en TODOS los archivos de TODOS los repos.

## F2 - Paso 5: Re-linkear

```bash
node kill-vtex-links.js
node vtex-link.js
```

Matar linkeos anteriores, re-linkear con cambios aplicados.

## F2 - Paso 6: Subir templates de email

```bash
cd ../mails-backup-{PROYECTO}
node upload-templates.js
```

## F2 - Paso 7: Limpiar init-ftlat

```bash
cd ..
rm -rf init-ftlat
```

**SOLO si todos los pasos completaron OK.**

## F2 - Resultado

```
═══════════════════════════════════════
Fase 2 completada ✓
Marca: {MARCA}
Fuentes: {PRIMARY_FONT} / {SECONDARY_FONT}
Mails: subidos
init-ftlat: eliminado

Para integraciones → ejecutar Fase 3
═══════════════════════════════════════
```

---

# FASE 3 — CUSTOMS

Objetivo: integraciones, envio gratis, newsletter, configuraciones finales.

## F3 - Paso 0: Recoger datos

### Bloque A — Integraciones

```
1. TOKEN_TYPEFORM   (vacio si no aplica)
2. MARCA_TYPEFORM   (vacio si no aplica)
3. FRESHDESK_MARCA  (vacio si no aplica)
```

### Bloque B — Envio gratis

```
4. precioEnvio — Monto minimo (ej: 82000)
```

Usar `AskUserQuestion`:
- "Personalizar mensajes de barra de envio gratis o usar defaults?"
- Opciones: "Usar defaults (Recomendado)", "Personalizar"

Defaults:
```json
{
  "precioEnvio": "{valor}",
  "showProgressBar": true,
  "messageBarTitle": "Completa tu pedido y obten <b>envio gratis</b>",
  "messageBarPending": "Solo te quedan",
  "messageBarSuccess": "Preparate para estrenar! Disfruta tu envio gratis.",
  "minicartTextPending": "Te faltan solo",
  "minicartTextPendingSuffix": "para envio gratis",
  "minicartTextSuccess": "Ganaste envio gratis!"
}
```

## F3 - Paso 1: Aplicar integraciones

Buscar y reemplazar tokens de integracion en archivos relevantes de los repos del proyecto.

## F3 - Paso 2: Generar checklist manual

```markdown
## Checklist Post-Instalacion — {MARCA}

### 1. Favicon
- URL: https://{VENDOR}.myvtex.com/admin/cms/layout
- Configuracion > General > Favicon > Agregar

### 2. HTML Header/Footer Checkout
- URL: https://{VENDOR}.myvtex.com/admin/portal#/sites/default/code
- Pestana Codigo → Header / Footer
- Pegar de checkout-io-ftlat/header.html y footer.html

### 3. Archivo monto-envio.json
- URL: https://{VENDOR}.myvtex.com/admin/portal#/sites/default/code
- Crear archivo con contenido:
{JSON generado}

### 4. Entidad Newsletter en MasterData
- URL: https://{VENDOR}.ds.vtexcrm.com.br
- Sigla: NL, Nombre: Newsletter, Campo: email

### 5. Trigger Newsletter
- URL: https://{VENDOR}.ds.vtexcrm.com.br → Trigger
- Entidad NL, Rules: "A record is created", ASAP
- Send email a {!email}, HTML, pegar template mails-backup

### 6. Mascara Simple
- URL: https://{VENDOR}.myvtex.com/admin/checkout#/configurations
- Tipo mascara conversacion → Simple
```

## F3 - Resultado

```
═══════════════════════════════════════
Fase 3 completada ✓
Integraciones: [aplicadas/no aplica]
Envio gratis: configurado
Checklist manual: generada

Tienda lista para QA.
═══════════════════════════════════════
```

---

## Manejo de Errores

### Script falla
1. Leer output del error
2. Diagnosticar:
   - `ENOENT` → archivo/carpeta no existe
   - `401/403` → token VTEX expirado, `vtex login`
   - `yarn` falla → `yarn cache clean` + reintentar
   - `vtex link` falla → `vtex whoami`
3. Reintentar paso fallido
4. Si falla 2 veces → reportar con error exacto

### Repo no se clona
```bash
git clone git@bitbucket.org:vtex-infralatam/{repo}-{PROYECTO}.git
```
Si SSH falla → verificar acceso Bitbucket.

---

## Reglas

1. **NUNCA ejecutar paso sin completar anterior** (excepto IS + Easy Setup que corren en paralelo)
2. **SIEMPRE verificar** exito antes de avanzar
3. **PRESERVAR** PRIMARY_FONT/SECONDARY_FONT en Fase 2 paso 2
4. **INFORMAR** progreso despues de cada paso (usar TaskCreate/TaskUpdate)
5. **NO mezclar fases** — Fase 1 no pregunta colores, Fase 2 no pregunta integraciones
6. **CWD**: Scripts siempre desde init-ftlat/
7. **ELIMINAR** init-ftlat solo al final de Fase 2 si todo OK
8. **WORKSPACE**: Fase 1 siempre usa `fasttemplate`

---

## Output al Usuario

### Formato de progreso

Usar formato compacto de una linea por paso:

```
Fase {N} — {titulo}
─────────────────────────────────────

[1/N] Descripcion corta              ✓
[2/N] Descripcion corta              ⚠️ Accion requerida
[3/N] Descripcion corta              ⏳ background
[4/N] Descripcion corta              ✗ Error: {mensaje}
```

### Iconos de estado

- `✓` — completado OK
- `⚠️` — requiere accion del usuario (login, click en browser, etc.)
- `⏳` — corriendo en background
- `✗` — fallo (mostrar error corto)

### Reglas de output

1. **Una linea por paso**. No bloques de codigo con output del comando.
2. **Pasos internos son invisibles**. Clone de init-ftlat, mkdir, cd — no se muestran. Solo aparecen si fallan.
3. **Solo detalle cuando**: hay error, o se necesita accion del usuario.
4. **Resumen final** compacto al terminar la fase.

### Ejemplo Fase 1

```
Fase 1 — Levantando tienda VTEX IO
─────────────────────────────────────

[1/7] VTEX login {VENDOR}               ⚠️ Completa login en browser
[1/7] VTEX login {VENDOR}               ✓
[2/7] Easy Setup ({TIPO}, {PAIS})        ⏳ background
[3/7] Intelligent Search               ⚠️ Abrir {URL} → click boton
[4/7] Clonar repos (10)                 ✓ 10/10
[5/7] Dependencias + peer deps          ✓
[6/7] Linkear en workspace fasttemplate  ✓ 9/9 apps
[7/7] Easy Setup                        ✓ catalogo + logistica + pagos

═══════════════════════════════════════
Fase 1 completada ✓
Tienda: https://{VENDOR}.myvtex.com
Workspace: fasttemplate

Para branding → ejecutar Fase 2
═══════════════════════════════════════
```
