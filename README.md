# Guía Genérica: Proyecto Vue 3 + PrimeVue + DDD + vue-i18n + API Real

> **Para qué sirve esta guía:** Puedes aplicarla a cualquier proyecto nuevo que use
> Vue 3, PrimeVue, vue-i18n, Pinia y una API real (no fake). Solo cambia los nombres
> del dominio y los campos de tu API. Todo lo demás es igual.

---

## PASO 1 — Ubicarse en la carpeta de trabajo

```bash
cd IdeaProjects
ls -l
cd [NRC]
```

> Reemplaza `[NRC]` con tu número de NRC (ej: `11990`).  
> El `ls -l` es solo para confirmar que estás en el lugar correcto.

---

## PASO 2 — Crear el proyecto Vue

```bash
sudo npm create vue@latest [nombre-del-proyecto]
```

> Reemplaza `[nombre-del-proyecto]` con el nombre que indique el enunciado.  
> Ejemplo: `pc111990u202418655`

Durante la creación responde **exactamente** así:

| Pregunta | Respuesta |
|---|---|
| Add TypeScript? | **No** (salvo que el enunciado lo exija) |
| Add JSX Support? | **No** |
| Add Vue Router? | **Yes** |
| Add Pinia? | **Yes** |
| Add Vitest? | **No** |
| Add ESLint? | **Yes** |

**¿Por qué estas opciones?**
- **Vue Router → Yes:** para tener soporte de rutas aunque no las uses en este proyecto.
- **Pinia → Yes:** es el gestor de estado oficial de Vue 3. Lo usarás en la capa `application`.
- **Vitest y JSX → No:** no son necesarios para proyectos de este tipo.

---

## PASO 3 — Entrar al proyecto e instalar PrimeVue

```bash
cd [nombre-del-proyecto]
sudo npm install primevue @primeuix/themes primeicons primeflex axios
```

> Esto instala:
> - **primevue:** librería de componentes UI (cards, buttons, menubar, etc.)
> - **@primeuix/themes:** temas visuales para PrimeVue (ej: Material)
> - **primeicons:** íconos `pi pi-*` que usan los componentes
> - **primeflex:** utilidades CSS (grid, flex, spacing) estilo Bootstrap
> - **axios:** para hacer llamadas HTTP a tu API real

---

## PASO 4 — Instalar vue-i18n

```bash
sudo npm install vue-i18n --save
```

**¿Por qué?** El enunciado pide que la interfaz soporte inglés y español.
`vue-i18n` es la librería estándar de internacionalización para Vue 3.

---

## PASO 5 — Cambiar propietario (solo MAC de los laboratorios)

```bash
cd ..
sudo chown -R alumnos ./[nombre-del-proyecto]
ls -l
```

**¿Por qué?** Al usar `sudo` en el paso 2, los archivos quedan con propietario `root`.
Este comando los regresa al usuario `alumnos` para poder editarlos en IntelliJ sin
problemas de permisos.

---

## PASO 6 — Instalar dependencias y verificar que corre

```bash
cd [nombre-del-proyecto]
npm install
npm run dev
```

Abre el navegador en `http://localhost:5173` y verifica que aparece la página por defecto de Vue.
Si funciona, puedes cerrar ese servidor por ahora (`Ctrl + C`).

---

## PASO 7 — Abrir el proyecto en IntelliJ IDEA

Abre IntelliJ IDEA → `Open` → selecciona la carpeta del proyecto.
Espera que indexe los archivos. Luego usa la terminal integrada del IDE para los siguientes pasos.

---

## PASO 8 — Crear la estructura de carpetas DDD

Dentro de `src/` crear **manualmente** la siguiente estructura.
Cambia `[subdominio]` por el nombre de tu contexto de negocio
(ejemplos: `news`, `catalog`, `sales`, `recipes`).

```
src/
├── locales/                        ← traducciones EN y ES
│   ├── en.json
│   └── es.json
├── [subdominio]/                   ← ej: catalog, news, sales
│   ├── application/                ← store con estado reactivo
│   ├── domain/
│   │   └── model/                  ← entidades (.entity.js)
│   ├── infrastructure/             ← llamadas HTTP y assemblers
│   └── presentation/
│       └── components/             ← componentes .vue del subdominio
└── shared/                         ← reutilizable entre subdominios
    ├── infrastructure/             ← servicios compartidos (ej: LogoApi)
    └── presentation/
        └── components/             ← layout, footer, language-switcher
```

**Regla de oro de la arquitectura:**

```
domain     → solo datos y lógica pura. Sin Vue, sin axios, sin HTTP.
infrastructure → todo lo que toca el exterior: HTTP, assemblers.
application    → coordina domain + infrastructure. Expone estado reactivo.
presentation   → solo componentes .vue. No llama axios directamente.
shared         → lo que se repite en más de un subdominio.
```

---

## PASO 9 — Crear los archivos de traducción

En `src/locales/` crear dos archivos JSON con **exactamente las mismas claves**.
Solo cambia los valores al español.

**`src/locales/en.json`**

```json
{
  "some-label": "Your text here",
  "another-label": "Another text",
  "authoring-phrase": {
    "intro": "Made with",
    "use": "using",
    "author": "by {brand} Developer Team"
  },
  "footer": {
    "powered-by": "Powered by",
    "and": "and"
  }
}
```

**`src/locales/es.json`**

```json
{
  "some-label": "Tu texto aquí",
  "another-label": "Otro texto",
  "authoring-phrase": {
    "intro": "Hecho con",
    "use": "utilizando",
    "author": "por el Equipo de Desarrollo de {brand}"
  },
  "footer": {
    "powered-by": "Contenido proporcionado por",
    "and": "y"
  }
}
```

> **Qué cambiar:**
> - Agrega o quita claves según los textos fijos que tenga tu interfaz.
> - `{brand}` es un placeholder dinámico. En el componente lo pasas así:
>   `t('authoring-phrase.author', { brand: 'MiMarca' })`.
> - Nunca pongas datos que vienen de la API aquí. Solo textos fijos de la UI.

---

## PASO 10 — Crear `src/i18n.js`

```js
import en from "./locales/en.json";
import es from "./locales/es.json";
import { createI18n } from "vue-i18n";

/**
 * Shared internationalization service used across presentation modules.
 */
const i18n = createI18n({
  legacy: false,        // OBLIGATORIO para Vue 3 Composition API
  locale: "en",         // idioma por defecto al cargar la app
  fallbackLocale: "en", // idioma de respaldo si falta una clave
  messages: { en, es }
});

export default i18n;
```

> **`legacy: false` es obligatorio.** Sin esto, `useI18n()` no funciona en
> los componentes con `<script setup>`.

---

## PASO 11 — Crear los archivos de variables de entorno

Crear **dos archivos** en la raíz del proyecto (mismo nivel que `package.json`):

**`.env.development`**

```env
# ─── Tu API Key ────────────────────────────────────────────────────────────────
# Copia aquí la API Key que te da el proveedor al registrarte.
# Ejemplo: si usas NewsAPI, te la dan en https://newsapi.org/account
VITE_[NOMBRE]_API_KEY="pega-tu-api-key-aqui"

# ─── URL base del API ──────────────────────────────────────────────────────────
# Es la parte del URL que NO cambia entre endpoints.
# Ejemplo NewsAPI:  https://newsapi.org/v2
# Ejemplo otro API: https://api.spoonacular.com
# No pongas / al final.
VITE_[NOMBRE]_API_URL="https://dominio-del-api.com/version"

# ─── Rutas de los endpoints ────────────────────────────────────────────────────
# Es la parte del URL que sí cambia según el recurso que pides.
# Míralo en la documentación del API bajo "Endpoint" o "Path".
# Ejemplo: /top-headlines/sources  o  /recipes  o  /products
VITE_[RECURSO]_ENDPOINT_PATH="/ruta-del-endpoint"

# Si tu API tiene más de un endpoint, agrega una línea por cada uno:
# VITE_[OTRO_RECURSO]_ENDPOINT_PATH="/otra-ruta"
```

**`.env.production`**

```env
# Mismas variables pero apuntando al ambiente de producción.
# Si usas la misma API en prod y dev, los valores son iguales.
VITE_[NOMBRE]_API_KEY="pega-tu-api-key-aqui"
VITE_[NOMBRE]_API_URL="https://dominio-del-api.com/version"
VITE_[RECURSO]_ENDPOINT_PATH="/ruta-del-endpoint"
```

> **Reglas importantes:**
> - Todas las variables **deben empezar con `VITE_`**. Sin ese prefijo, Vite no las
>   expone al código del frontend.
> - Se acceden en el código con: `import.meta.env.VITE_NOMBRE_VARIABLE`
> - Nunca subas estos archivos a GitHub si contienen API keys reales.
>   Agrégalos al `.gitignore`.

---

## PASO 12 — Crear las entidades del dominio

En `src/[subdominio]/domain/model/` crear un archivo `.entity.js` por cada
objeto de tu dominio.

**¿Cuántas entidades necesito?**  
Una por cada objeto distinto que devuelve tu API. Mira el JSON de respuesta:
- Si tiene un objeto anidado con varios campos → entidad separada.
- Si tiene un array de objetos → entidad para esos objetos.

**Ejemplo base (`mi-entidad.entity.js`):**

```js
/**
 * Domain entity representing [describe qué es este objeto].
 *
 * @remarks
 * This model belongs to the domain layer and stays independent from
 * transport or framework details.
 */
export class MiEntidad {
  /**
   * @param {Object} props
   * @param {string} [props.campo1]   ← un campo por cada atributo de tu JSON
   * @param {number} [props.campo2]
   * @param {Object} [props.objetoAnidado]
   * @param {Array}  [props.listaDeItems]
   */
  constructor({
    campo1          = '',
    campo2          = 0,
    objetoAnidado   = {},
    listaDeItems    = []
  } = {}) {
    this.campo1        = campo1;
    this.campo2        = campo2;
    this.objetoAnidado = objetoAnidado;
    this.listaDeItems  = listaDeItems;
  }

  // Si necesitas formatear algún dato (ej: fecha), agrégalo aquí como método:
  // getFormattedDate() { return this.fecha.toLocaleDateString(...); }
}
```

> **Qué cambiar:**
> - Renombra la clase según tu dominio (`Recipe`, `Article`, `Product`, etc.)
> - Los campos del `constructor` deben coincidir con los campos de tu JSON,
>   **pero usando camelCase**. Si el API devuelve `dish_name`, aquí va `dishName`.
>   El Assembler (paso 14) se encarga de esa traducción.
> - Si un campo es un objeto anidado complejo, crea una entidad aparte para él
>   y en el constructor instanciala: `this.objetoAnidado = new OtraEntidad(props.objetoAnidado)`.

---

## PASO 13 — Crear el servicio de infraestructura (API Service)

En `src/[subdominio]/infrastructure/` crear `[nombre]-api.js`:

```js
import axios from "axios";

// Lee las variables del archivo .env.development o .env.production
// según el ambiente en que corra la app (Vite lo detecta automáticamente).
const baseUrl        = import.meta.env.VITE_[NOMBRE]_API_URL;
const apiKey         = import.meta.env.VITE_[NOMBRE]_API_KEY;
const recursoPath    = import.meta.env.VITE_[RECURSO]_ENDPOINT_PATH;
// Si tienes más endpoints, agrega más constantes aquí:
// const otroPath    = import.meta.env.VITE_[OTRO]_ENDPOINT_PATH;

// Configuración base de axios: todas las peticiones de esta clase
// usarán esta baseURL y enviarán el apiKey automáticamente.
const http = axios.create({
  baseURL: baseUrl,
  params: {
    apiKey: apiKey,   // elimina esta línea si tu API no usa apiKey como param
  }
});

/**
 * Infrastructure adapter for [nombre del proveedor] HTTP endpoints.
 *
 * @remarks
 * Isolates external transport concerns from application and domain layers.
 */
export class [Nombre]Api {

  /**
   * Fetches [describe qué devuelve].
   * @returns {Promise<import('axios').AxiosResponse>}
   */
  get[Recursos]() {
    return http.get(recursoPath);
  }

  // Si tu API tiene más endpoints, agrega un método por cada uno:
  // get[OtroRecurso](param) {
  //   return http.get(otroPath, { params: { key: param } });
  // }
}
```

> **Qué cambiar:**
> - Renombra la clase (`NewsApi`, `SpoonacularApi`, `ProductsApi`, etc.)
> - Renombra las constantes con los nombres de tus variables de entorno.
> - El método `params: { apiKey }` varía según cómo tu API reciba la autenticación:
>   - **Query param:** `params: { apiKey: apiKey }` ← como está arriba
>   - **Header Bearer:** `headers: { Authorization: \`Bearer ${apiKey}\` }`
>   - **Sin auth:** elimina `params` del `axios.create`
> - Mira la documentación de tu API para saber cuál aplica.

---

## PASO 14 — Crear el Assembler

En `src/[subdominio]/infrastructure/` crear `[nombre].assembler.js`:

```js
import { MiEntidad } from "../domain/model/mi-entidad.entity.js";

/**
 * Maps raw API resources into domain entities.
 *
 * @remarks
 * Responsible for translating infrastructure data (snake_case, nested objects,
 * different field names) into clean domain entities.
 */
export class [Nombre]Assembler {

  /**
   * Maps the full API response to an array of entities.
   *
   * @param {import('axios').AxiosResponse} response
   * @returns {MiEntidad[]}
   */
  static toEntitiesFromResponse(response) {
    // Ajusta esta línea según la estructura real de tu JSON de respuesta:
    //
    // Si el API devuelve un array directamente: [ {...}, {...} ]
    //   → const list = response.data;
    //
    // Si el API devuelve un objeto con status y un array adentro:
    //   { "status": "ok", "recipes": [ {...}, {...} ] }
    //   → const list = response.data.recipes;
    //
    // Si el API devuelve un objeto con status "ok" que debes verificar:
    //   → if (response.data.status !== "ok") return [];
    //      const list = response.data.[nombreDelArray];

    const list = response.data; // ← cambia esto según tu caso

    return list.map(item => this.toEntityFromResource(item));
  }

  /**
   * Maps a single resource object to a domain entity.
   *
   * @param {Object} resource
   * @returns {MiEntidad}
   */
  static toEntityFromResource(resource) {
    return new MiEntidad({
      // Aquí mapeas cada campo del JSON al atributo de la entidad.
      // Izquierda: nombre del atributo en tu entidad (camelCase).
      // Derecha:   nombre del campo en el JSON del API (como venga).
      campo1:        resource.campo1,         // si tienen el mismo nombre
      campo2:        resource.campo_2,        // si el API usa snake_case
      objetoAnidado: resource.nested_object,  // objeto anidado
      listaDeItems:  resource.items ?? [],    // array, con fallback a []
    });
  }
}
```

> **Qué cambiar:**
> - Renombra la clase y los imports.
> - En `toEntitiesFromResponse`, ajusta cómo se extrae la lista del JSON.
>   Abre `http://tu-api.com/tu-endpoint` en el navegador y mira la estructura.
> - En `toEntityFromResource`, mapea campo por campo.
>   Si el API devuelve `dish_name` y tu entidad tiene `dishName`, aquí va:
>   `dishName: resource.dish_name`.
> - Si hay objetos anidados complejos, crea un segundo Assembler para ellos
>   y llámalo aquí.

---

## PASO 15 — Crear el Store (capa Application)

En `src/[subdominio]/application/` crear `[nombre].store.js`:

```js
import { reactive } from "vue";
import { [Nombre]Api } from "../infrastructure/[nombre]-api.js";
import { [Nombre]Assembler } from "../infrastructure/[nombre].assembler.js";

const api = new [Nombre]Api();

/**
 * Application-layer state container for [describe el dominio].
 *
 * @remarks
 * Coordinates use-case behavior and delegates data acquisition
 * and mapping to infrastructure services.
 */
export const [nombre]Store = reactive({
  [recursos]: [],    // array principal de entidades (ej: recipes, articles)
  errors: [],        // errores de red o del API

  /**
   * Loads [recursos] from the API and maps them to domain entities.
   * @returns {void}
   */
  load[Recursos]() {
    this.errors = [];

    // Evita llamadas duplicadas si ya hay datos cargados:
    if (this.[recursos].length > 0) return;

    api.get[Recursos]()
      .then(response => {
        this.[recursos] = [Nombre]Assembler.toEntitiesFromResponse(response);
      })
      .catch(error => {
        this.errors.push(error);
        this.[recursos] = [];
      });
  },

  // Si tu dominio necesita más acciones, agrégalas aquí como métodos:
  // setCurrentItem(item) { ... },
  // loadDetailFor(id) { ... }
});
```

> **Qué cambiar:**
> - Renombra todo lo que está entre corchetes `[ ]`.
> - El objeto `reactive({})` puede tener tantas propiedades y métodos como
>   necesite tu dominio.
> - Si la selección de un ítem dispara otra llamada al API (como en el proyecto
>   de referencia con `setCurrentSource`), agrégalo como método adicional.

---

## PASO 16 — Crear los componentes de presentación del subdominio

En `src/[subdominio]/presentation/components/` crear un `.vue` por ítem
y otro por lista.

### `[nombre]-item.vue` — card de un elemento

```vue
<script setup lang="js">
import { toRefs } from "vue";
import { useI18n } from "vue-i18n";
import { MiEntidad } from "../../domain/model/mi-entidad.entity.js";

/**
 * Presentation component for a single [nombre] card.
 */

const { t } = useI18n();

// defineProps declara qué datos recibe este componente desde el padre.
const props = defineProps({
  [nombreEntidad]: { type: MiEntidad, required: true }
});
const { [nombreEntidad] } = toRefs(props);
</script>

<template>
  <!--
    Aquí va el HTML del card.
    Accedes a los datos con: [nombreEntidad].campo1
    Accedes a traducciones con: t('clave.del.json')
    Accedes a ARIA con: :aria-label="[nombreEntidad].campo1"
  -->
  <pv-card :aria-label="[nombreEntidad].campo1">
    <template #title>{{ [nombreEntidad].campo1 }}</template>
    <template #subtitle>{{ [nombreEntidad].campo2 }}</template>
    <template #content>
      <p>
        <strong>{{ t('alguna-etiqueta') }}:</strong>
        {{ [nombreEntidad].otrocampo }}
      </p>
      <!--
        Para secciones expandibles usa pv-panel o el componente
        de Accordion de PrimeVue.
        Para listas usa <ul>/<li> o el componente Listbox de PrimeVue.
      -->
    </template>
  </pv-card>
</template>

<style scoped>
/* Estilos propios de este componente. No afectan al resto de la app. */
</style>
```

### `[nombre]-list.vue` — lista de elementos

```vue
<script setup lang="js">
import { toRefs } from "vue";
import { useI18n } from "vue-i18n";
import [Nombre]Item from "./[nombre]-item.vue";

/**
 * Presentation component that renders a collection of [nombre] cards.
 */

const { t } = useI18n();

const props = defineProps({
  [recursos]: { type: Array, required: true }
});
const { [recursos] } = toRefs(props);
</script>

<template>
  <section :aria-label="t('catalog.title')">
    <h2>{{ t('catalog.title') }}</h2>

    <!--
      primeflex grid: cols="X" controla cuántos cards por fila.
      col-12 = 1 por fila | col-6 = 2 | col-4 = 3 | col-3 = 4
      Ajusta según lo que pide el enunciado.
    -->
    <div class="grid">
      <div
        v-for="item in [recursos]"
        :key="item.[campoUnico]"
        class="col-4">
        <[nombre]-item :[nombreEntidad]="item"/>
      </div>
    </div>
  </section>
</template>

<style scoped>
h2 { text-align: center; }
</style>
```

> **Qué cambiar:**
> - Renombra todos los `[nombre]`, `[recursos]`, `[nombreEntidad]`, `[campoUnico]`.
> - `[campoUnico]` es el campo que usas en `:key`, debe ser único por ítem
>   (ej: `id`, `dishName`, `url`).
> - Agrega o quita campos en el template según los atributos de tu entidad.
> - El número de columnas (`col-4` = 3 por fila) cámbialo según el enunciado.

---

## PASO 17 — Crear los componentes shared

### `src/shared/presentation/components/language-switcher.vue`

```vue
<template>
  <!--
    $i18n.locale: variable reactiva con el idioma actual.
    $i18n.availableLocales: array con los idiomas definidos en i18n.js (["en", "es"]).
    Al hacer click en un botón, $i18n.locale cambia y toda la app se retraduce.
  -->
  <pv-select-button
    v-model="$i18n.locale"
    :options="$i18n.availableLocales">
    <template #option="slotProps">
      <span>{{ slotProps.option.toUpperCase() }}</span>
    </template>
  </pv-select-button>
</template>
```

### `src/shared/presentation/components/footer-content.vue`

```vue
<script setup lang="js">
import { useI18n } from "vue-i18n";

/**
 * Shared footer with attribution and localization support.
 */
const { t } = useI18n();
</script>

<template>
  <footer class="footer" role="contentinfo">
    <!--
      Cambia el contenido según lo que pida el enunciado.
      Usa t('clave') para textos que deben traducirse.
      Usa texto fijo solo si el enunciado dice que va igual en todos los idiomas.
    -->
    <div>
      <p>Copyright &copy; 2026. [Nombre de la empresa]</p>
    </div>
    <div>
      <p>
        {{ t('authoring-phrase.intro') }}
        {{ t('authoring-phrase.use') }} <a href="[url-framework]" target="_blank">[Framework]</a>
        {{ t('authoring-phrase.author', { brand: '[TuMarca]' }) }}
      </p>
      <p>
        {{ t('footer.powered-by') }}
        <a href="[url-api]" target="_blank">[Nombre API]</a>
      </p>
    </div>
  </footer>
</template>

<style scoped>
.footer {
  background-color: #1976d2; /* cambia el color si el enunciado lo indica */
  color: white;
  text-align: center;
  padding: 16px;
  margin-top: 24px;
}
</style>
```

### `src/shared/presentation/components/layout.vue`

```vue
<script setup lang="js">
import { computed, onMounted } from "vue";

// Importa el store de tu subdominio
import { [nombre]Store } from "../../../[subdominio]/application/[nombre].store.js";

// Importa los componentes de presentación
import [Nombre]List from "../../../[subdominio]/presentation/components/[nombre]-list.vue";
import LanguageSwitcher from "./language-switcher.vue";
import FooterContent from "./footer-content.vue";

/**
 * Main layout component. Coordinates UI structure and delegates
 * business logic to the application store.
 */

// Computed properties reactivas que apuntan al estado del store
const [recursos] = computed(() => [nombre]Store.[recursos]);
const errors     = computed(() => [nombre]Store.errors);

// onMounted: se ejecuta cuando el componente se monta en el DOM.
// Aquí es donde llamas al store para cargar los datos por primera vez.
onMounted(() => {
  [nombre]Store.load[Recursos]();
});
</script>

<template>
  <div>
    <!-- Barra de navegación superior -->
    <pv-menubar>
      <template #start>
        <!--
          Aquí va el logo o título de la app.
          Puedes usar un texto fijo o una imagen.
        -->
        <span>[Título de la App]</span>
      </template>
      <template #end>
        <!-- Selector de idioma siempre en la esquina superior derecha -->
        <language-switcher/>
      </template>
    </pv-menubar>
  </div>

  <!-- Contenido principal -->
  <main>
    <!--
      v-if / v-else: muestra la lista si hay datos,
      un mensaje de error si algo falló,
      o "Loading..." mientras carga.
    -->
    <[nombre]-list
      v-if="[recursos].length > 0"
      :[recursos]="[recursos]"/>
    <p v-else-if="errors.length > 0">
      Service is currently unavailable.
    </p>
    <p v-else>Loading...</p>
  </main>

  <footer-content/>
</template>
```

---

## PASO 18 — Configurar `src/main.js`

```js
import { createApp } from 'vue'
import './style.css'
import App from './app.vue'
import i18n from "./i18n.js";
import PrimeVue from 'primevue/config';
import Material from '@primeuix/themes/material';
import 'primeicons/primeicons.css';
import 'primeflex/primeflex.css';

// Importa aquí SOLO los componentes de PrimeVue que uses en tu app.
// Si agregas un componente nuevo al template, también debes importarlo aquí.
import {
  Avatar,
  Button,
  Card,
  Drawer,
  Menubar,
  SelectButton,
  Toolbar,
  Tooltip
  // Agrega o quita según los componentes que uses
} from "primevue";

/**
 * Application composition root.
 * Wires cross-cutting services and UI components before mounting.
 */
createApp(App)
  .use(i18n)
  .use(PrimeVue, { ripple: true, theme: { preset: Material } })
  // Registra cada componente con su alias pv-* para usarlo en los templates
  .component('pv-button',        Button)
  .component('pv-select-button', SelectButton)
  .component('pv-avatar',        Avatar)
  .component('pv-drawer',        Drawer)
  .component('pv-card',          Card)
  .component('pv-toolbar',       Toolbar)
  .component('pv-menubar',       Menubar)
  .directive('tooltip',          Tooltip)
  .mount('#app');
```

> **Qué cambiar:**
> - Agrega o quita componentes de PrimeVue según los que uses en tus templates.
> - Si usas un `pv-algo` en algún `.vue` y no está registrado aquí, verás un
>   warning en consola y el componente no renderizará.
> - El nombre del alias (`pv-button`) puede ser cualquier cosa, pero por
>   convención se usa el prefijo `pv-` para distinguirlos de HTML nativo.

---

## PASO 19 — Configurar `src/app.vue`

```vue
<script setup>
import { useI18n } from 'vue-i18n'
import Layout from "./shared/presentation/components/layout.vue";

/**
 * Presentation shell component.
 * Hosts the application layout and keeps bootstrapping concerns
 * out of feature components.
 */
const { t } = useI18n()
</script>

<template>
  <!-- App.vue solo renderiza el Layout. No pongas lógica de negocio aquí. -->
  <layout/>
</template>
```

---

## PASO 20 — Verificar `index.html`

En la raíz del proyecto, `index.html` debe verse así (sin cambios respecto al default de Vue):

```html
<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="UTF-8">
    <link rel="icon" href="/favicon.ico">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>[Nombre de tu App]</title>  <!-- cambia solo el título si quieres -->
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

---

## PASO 21 — Iniciar la aplicación

Abre **dos terminales separadas** en el IDE:

### Terminal 1 — Iniciar la app Vue
```bash
npm run dev
```

Verifica en: `http://localhost:5173`

### Terminal 2 — (Opcional) Si en algún proyecto necesitas json-server
```bash
json-server --watch server/db.json --routes server/routes.json
```

> En proyectos con API real como este, **no necesitas json-server**.
> Solo úsalo si el enunciado lo indica como fallback o si el API real falla.

---

## PASO 22 — Empaquetar para entrega

```bash
# Asegúrate de estar en la raíz del proyecto
rm -rf node_modules

# Comprimir (en MAC/Linux)
zip -r pc1[NRC][codigo].zip ./[nombre-del-proyecto]
```

El archivo resultante debe llamarse exactamente como indique el enunciado,


Para restaurar después de descomprimir:
```bash
npm install
npm run dev
```

---

## Referencia rápida: Flujo de datos

```
Usuario interactúa con un componente .vue
           ↓
  [presentation] → emite evento o llama método del store
           ↓
  [application/store] → coordina la lógica, llama al API service
           ↓
  [infrastructure/api] → hace la petición HTTP con axios
           ↓
  API Real responde con JSON
           ↓
  [infrastructure/assembler] → traduce JSON → entidad del dominio
           ↓
  [application/store] → guarda entidades en reactive()
           ↓
  [presentation] → computed() detecta el cambio y re-renderiza
```

---

## Errores comunes y soluciones

| Error | Causa | Solución |
|---|---|---|
| Pantalla en blanco | Error JS en consola | F12 → Console → leer el error |
| `Cannot find module` | Ruta de import incorrecta | Verificar ruta relativa exacta |
| Componente no renderiza | No registrado en main.js | Agregar `.component('pv-X', X)` |
| Textos no se traducen | `legacy: false` falta | Agregarlo en `createI18n({...})` |
| API devuelve 401 | API Key incorrecta o falta | Verificar `.env.development` |
| API devuelve 403 | Plan gratuito no permite CORS | Usar proxy o cambiar de API |
| Datos no aparecen | Error en el Assembler | Console.log de `response.data` para ver estructura real |
| Puerto 5173 en uso | Otra instancia corriendo | `npm run dev -- --port 5174` |

---

*Guía elaborada para el curso Desarrollo de Aplicaciones Web*
