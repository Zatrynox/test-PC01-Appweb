# Guía Paso a Paso: Proyecto Vue con Arquitectura DDD

> **Basado en:** el proyecto de referencia con Vue 3 + PrimeVue + Pinia + vue-i18n + json-server  
> **Objetivo:** Entender cada paso para poder replicarlo en cualquier proyecto similar

---

## PASO 1 — Ubicarse en la carpeta correcta

```bash
cd IdeaProjects
ls -l
cd 11990
```

**¿Por qué?** Es buena práctica tener todos los proyectos organizados por carpeta de curso/NRC para no mezclarlos.

---

## PASO 2 — Crear el proyecto Vue

```bash
sudo npm create vue@latest nombre-del-proyecto
```

> Reemplaza `nombre-del-proyecto` con el nombre real, por ejemplo: `pc111990u202418655`

Durante la creación, responde las preguntas así:

| Pregunta | Respuesta |
|---|---|
| Add TypeScript? | No (o Yes si el enunciado lo pide) |
| Add JSX Support? | No |
| Add Vue Router? | **Yes** |
| Add Pinia? | **Yes** |
| Add Vitest? | **Yes** |
| Add ESLint? | Yes |

**¿Por qué Pinia?** Es el gestor de estado oficial de Vue 3. En esta arquitectura lo usaremos como capa `application` (state management).  
**¿Por qué Vue Router?** Permite navegar entre vistas aunque en el enunciado diga "fuera de alcance", se instala por si acaso.

---

## PASO 3 — Entrar al proyecto e instalar dependencias de UI

```bash
cd nombre-del-proyecto
sudo npm install vuetify @mdi/font
```

> **Nota:** En este proyecto de referencia se usa **PrimeVue** en lugar de Vuetify. Si tu enunciado pide PrimeVue, instala esto en su lugar:

```bash
sudo npm install primevue @primeuix/themes primeicons primeflex axios
```

**¿Cuándo usar cada uno?**
- **Vuetify:** cuando el enunciado dice "Material Design" o "Vuetify"
- **PrimeVue:** cuando el enunciado menciona PrimeVue o la referencia usa componentes `pv-*`

---

## PASO 4 — Instalar vue-i18n (internacionalización)

```bash
sudo npm install vue-i18n --save
```

**¿Por qué?** El enunciado siempre pide que la interfaz soporte múltiples idiomas (EN/ES). `vue-i18n` es la librería estándar para esto en Vue.

---

## PASO 5 — Instalar json-server (fake REST API)

```bash
sudo npm install -g json-server@0.17.4
```

**¿Por qué la versión 0.17.4?** Las versiones posteriores cambian la API y el comportamiento de las rutas. El curso usa específicamente esta versión para que `routes.json` funcione correctamente.

---

## PASO 6 — Cambiar propietario (solo MAC)

```bash
cd ..
sudo chown -R alumnos ./nombre-del-proyecto
ls -l
```

**¿Por qué?** Al usar `sudo` para crear el proyecto, los archivos quedan con propietario `root`. Este comando los devuelve al usuario `alumnos` para poder editarlos sin problemas en IntelliJ.

---

## PASO 7 — Instalar dependencias y ejecutar

```bash
cd nombre-del-proyecto
npm install
npm run dev
```

Verifica que corra en `http://localhost:5173` (puerto por defecto de Vite).

---

## PASO 8 — Configurar la estructura de carpetas DDD

Dentro de `src/` crear la siguiente estructura manualmente:

```
src/
├── locales/              ← archivos de traducción EN y ES
├── [subdominio]/         ← nombre del contexto (ej: news, catalog, sales)
│   ├── application/      ← store / estado global / casos de uso
│   ├── domain/
│   │   └── model/        ← entidades del dominio (.entity.js)
│   ├── infrastructure/   ← llamadas HTTP, assemblers, adaptadores
│   └── presentation/
│       └── components/   ← componentes .vue de este subdominio
└── shared/               ← elementos reutilizables entre subdominios
    ├── infrastructure/   ← servicios compartidos (ej: LogoDevApi)
    └── presentation/
        └── components/   ← header, footer, layout, language-switcher
```

**Regla clave de DDD:**
- `domain/model/` → solo lógica pura, sin imports de Vue ni de HTTP
- `infrastructure/` → todo lo que toca el mundo exterior (API, HTTP, assemblers)
- `application/` → coordina domain + infrastructure, expone estado reactivo
- `presentation/` → solo componentes `.vue`, no llama HTTP directamente

---

## PASO 9 — Crear los archivos de traducción

En `src/locales/` crear dos archivos:

**`en.json`** (ejemplo base, adaptar al proyecto):
```json
{
  "toolbar": {
    "title": "Mi Aplicación"
  },
  "catalog": {
    "title": "Catálogo",
    "someLabel": "Some Label"
  },
  "footer": {
    "rights": "Copyright © 2026 MiEmpresa. All rights reserved",
    "author": "Developed by [Código], [Nombre Apellido]"
  }
}
```

**`es.json`** (misma estructura, textos en español):
```json
{
  "toolbar": {
    "title": "Mi Aplicación"
  },
  "catalog": {
    "title": "Catálogo",
    "someLabel": "Alguna Etiqueta"
  },
  "footer": {
    "rights": "Derechos de autor © 2026 MiEmpresa. Todos los derechos reservados",
    "author": "Desarrollado por [Código], [Nombre Apellido]"
  }
}
```

**Regla:** Los keys deben ser exactamente iguales en ambos archivos. Solo cambia el valor, nunca la clave.

---

## PASO 10 — Crear el archivo `i18n.js`

En `src/` crear `i18n.js`:

```js
import en from "./locales/en.json";
import es from "./locales/es.json";
import { createI18n } from "vue-i18n";

/**
 * Shared internationalization service used across presentation modules.
 */
const i18n = createI18n({
  legacy: false,       // obligatorio para Vue 3 Composition API
  locale: "en",        // idioma por defecto
  fallbackLocale: "en",
  messages: { en, es }
});

export default i18n;
```

**¿Por qué `legacy: false`?** Habilita el modo Composition API de vue-i18n, necesario para usar `useI18n()` en los componentes.

---

## PASO 11 — Configurar variables de entorno

Crear `.env.development` en la raíz del proyecto:

```env
# URL base del API que usarás en desarrollo
VITE_API_BASE_URL="http://localhost:3000/api/v1"

# Endpoint del recurso principal
VITE_RECIPES_ENDPOINT_PATH="/recipes"
```

Crear `.env.production` en la raíz:

```env
# URL base del API en producción
VITE_API_BASE_URL="https://miapi.com"
VITE_RECIPES_ENDPOINT_PATH="/recipes"
```

**Regla importante:** Todas las variables deben empezar con `VITE_` para que Vite las exponga al frontend. Se acceden con `import.meta.env.VITE_NOMBRE_VARIABLE`.

---

## PASO 12 — Configurar el json-server

Crear carpeta `server/` en la raíz del proyecto con dos archivos:

**`server/db.json`** — pegar el JSON del enunciado o del GitHub indicado:
```json
{
  "recipes": [
    { "dishName": "Lomo Saltado", "..." : "..." }
  ]
}
```

**`server/routes.json`** — mapeo de rutas:
```json
{
  "/api/v1/*": "/$1"
}
```

Iniciar el servidor en una terminal separada:
```bash
json-server --watch server/db.json --routes server/routes.json
```

Verificar en el navegador: `http://localhost:3000/api/v1/recipes`

---

## PASO 13 — Crear las entidades del dominio

En `src/[subdominio]/domain/model/` crear un archivo `.entity.js` por cada entidad:

**Ejemplo: `recipe.entity.js`**
```js
/**
 * Domain entity representing a Peruvian recipe.
 * @remarks Stays independent from HTTP or Vue concerns.
 */
export class Recipe {
  /**
   * @param {Object} props
   * @param {string} [props.dishName]
   * @param {string} [props.originCity]
   * @param {string} [props.mainProtein]
   * @param {Object} [props.nutritionalValue]
   * @param {Array}  [props.ingredients]
   */
  constructor({
    dishName = '',
    originCity = '',
    mainProtein = '',
    cookingTechnique = '',
    preparationTime = '',
    nutritionalValue = { calories: 0, fatContent: '', proteinContent: '' },
    ingredients = []
  } = {}) {
    this.dishName = dishName;
    this.originCity = originCity;
    this.mainProtein = mainProtein;
    this.cookingTechnique = cookingTechnique;
    this.preparationTime = preparationTime;
    this.nutritionalValue = nutritionalValue;
    this.ingredients = ingredients;
  }
}
```

**Regla:** Las entidades solo tienen datos y lógica de negocio pura (como formatear fechas). Nunca llaman `axios`, `fetch` ni usan `ref`/`reactive` de Vue.

---

## PASO 14 — Crear el servicio de infraestructura (API)

En `src/[subdominio]/infrastructure/` crear `[nombre]-api.js`:

```js
import axios from "axios";

const baseUrl      = import.meta.env.VITE_API_BASE_URL;
const recipesPath  = import.meta.env.VITE_RECIPES_ENDPOINT_PATH;

const http = axios.create({ baseURL: baseUrl });

/**
 * Infrastructure adapter for the Spoonacular HTTP endpoint.
 */
export class SpoonacularApi {
  /**
   * Fetches all recipes from the API.
   * @returns {Promise<import('axios').AxiosResponse>}
   */
  getRecipes() {
    return http.get(recipesPath);
  }
}
```

**¿Por qué una clase separada?** Si el día de mañana cambias de axios a fetch, solo modificas esta clase. El resto del código no se entera.

---

## PASO 15 — Crear el Assembler

En `src/[subdominio]/infrastructure/` crear `[nombre].assembler.js`:

```js
import { Recipe } from "../domain/model/recipe.entity.js";

/**
 * Maps raw API resources into domain entities.
 */
export class RecipeAssembler {
  /**
   * @param {import('axios').AxiosResponse} response
   * @returns {Recipe[]}
   */
  static toEntitiesFromResponse(response) {
    // Ajusta según la estructura real del JSON
    const data = response.data;
    const list = Array.isArray(data) ? data : data.recipes ?? [];
    return list.map(item => this.toEntityFromResource(item));
  }

  /**
   * @param {Object} resource
   * @returns {Recipe}
   */
  static toEntityFromResource(resource) {
    return new Recipe({
      dishName:         resource.dishName,
      originCity:       resource.originCity,
      mainProtein:      resource.mainProtein,
      cookingTechnique: resource.cookingTechnique,
      preparationTime:  resource.preparationTime,
      nutritionalValue: resource.nutritionalValue,
      ingredients:      resource.ingredients
    });
  }
}
```

**¿Por qué el Assembler?** Si el API devuelve `dish_name` (snake_case) pero tu entidad usa `dishName` (camelCase), el Assembler hace esa traducción. La entidad siempre queda limpia.

---

## PASO 16 — Crear el Store (capa Application)

En `src/[subdominio]/application/` crear `[nombre].store.js`:

```js
import { reactive } from "vue";
import { SpoonacularApi } from "../infrastructure/spoonacular-api.js";
import { RecipeAssembler } from "../infrastructure/recipe.assembler.js";

const api = new SpoonacularApi();

/**
 * Application-layer state container for recipes.
 * @remarks Coordinates use-case behavior and delegates to infrastructure.
 */
export const recipeStore = reactive({
  recipes: [],
  errors: [],

  /**
   * Loads recipes from the API and maps them to domain entities.
   * @returns {void}
   */
  loadRecipes() {
    this.errors = [];
    if (this.recipes.length > 0) return; // evitar llamadas duplicadas

    api.getRecipes()
      .then(response => {
        this.recipes = RecipeAssembler.toEntitiesFromResponse(response);
      })
      .catch(error => {
        this.errors.push(error);
        this.recipes = [];
      });
  }
});
```

**¿Por qué `reactive()` y no `ref()`?** Un objeto con múltiples propiedades relacionadas es más legible con `reactive`. Para valores simples se usa `ref`.

---

## PASO 17 — Crear los componentes de presentación

### `recipe-item.vue` (card de un ítem)

```vue
<script setup lang="js">
import { toRefs } from "vue";
import { useI18n } from "vue-i18n";
import { Recipe } from "../../domain/model/recipe.entity.js";

const { t } = useI18n();
const props = defineProps({ recipe: { type: Recipe, required: true } });
const { recipe } = toRefs(props);
</script>

<template>
  <pv-card :aria-label="recipe.dishName">
    <template #title>{{ recipe.dishName }}</template>
    <template #subtitle>{{ recipe.originCity }}</template>
    <template #content>
      <p><strong>{{ t('catalog.protein') }}:</strong> {{ recipe.mainProtein }}</p>
      <p><strong>{{ t('catalog.cookingStyle') }}:</strong> {{ recipe.cookingTechnique }}</p>
      <p><strong>{{ t('catalog.prepTime') }}:</strong> {{ recipe.preparationTime }}</p>
    </template>
  </pv-card>
</template>
```

### `recipe-list.vue` (lista de ítems)

```vue
<script setup lang="js">
import { toRefs } from "vue";
import RecipeItem from "./recipe-item.vue";
import { useI18n } from "vue-i18n";

const { t } = useI18n();
const props = defineProps({ recipes: { type: Array, required: true } });
const { recipes } = toRefs(props);
</script>

<template>
  <section aria-label="Peruvian recipes catalog">
    <h2>{{ t('catalog.title') }}</h2>
    <div class="grid">
      <div v-for="recipe in recipes" :key="recipe.dishName" class="col-4">
        <recipe-item :recipe="recipe"/>
      </div>
    </div>
  </section>
</template>
```

**Regla de presentación:**
- Los componentes reciben datos por `props` y emiten eventos con `emit`
- Nunca llaman `axios` ni modifican el store directamente
- Usan `useI18n()` para traducir textos fijos

---

## PASO 18 — Crear componentes shared

### `language-switcher.vue`

```vue
<template>
  <pv-select-button v-model="$i18n.locale" :options="$i18n.availableLocales">
    <template #option="slotProps">
      <span>{{ slotProps.option.toUpperCase() }}</span>
    </template>
  </pv-select-button>
</template>
```

### `footer-content.vue`

```vue
<script setup lang="js">
import { useI18n } from "vue-i18n";
const { t } = useI18n();
</script>

<template>
  <footer class="footer" role="contentinfo">
    <p>{{ t('footer.rights') }}</p>
    <p>{{ t('footer.author') }}</p>
  </footer>
</template>

<style scoped>
.footer {
  background-color: #1976d2;
  color: white;
  text-align: center;
  padding: 16px;
  margin-top: 24px;
}
</style>
```

### `layout.vue`

```vue
<script setup lang="js">
import { computed, onMounted } from "vue";
import { recipeStore } from "../../../catalog/application/recipe.store.js";
import RecipeList from "../../../catalog/presentation/components/recipe-list.vue";
import LanguageSwitcher from "./language-switcher.vue";
import FooterContent from "./footer-content.vue";

const recipes = computed(() => recipeStore.recipes);
const errors  = computed(() => recipeStore.errors);

onMounted(() => {
  recipeStore.loadRecipes();
});
</script>

<template>
  <div>
    <pv-menubar>
      <template #start>
        <span>Spoonacular - Peruvian Food</span>
      </template>
      <template #end>
        <language-switcher/>
      </template>
    </pv-menubar>
  </div>

  <main>
    <recipe-list v-if="recipes.length > 0" :recipes="recipes"/>
    <p v-else-if="errors.length > 0">Service unavailable.</p>
    <p v-else>Loading...</p>
  </main>

  <footer-content/>
</template>
```

---

## PASO 19 — Configurar `main.js`

```js
import { createApp } from 'vue'
import './style.css'
import App from './app.vue'
import i18n from "./i18n.js";
import PrimeVue from 'primevue/config';
import Material from '@primeuix/themes/material';
import 'primeicons/primeicons.css';
import 'primeflex/primeflex.css';
import {
  Avatar, Button, Card, Drawer,
  Menubar, SelectButton, Toolbar, Tooltip
} from "primevue";

createApp(App)
  .use(i18n)
  .use(PrimeVue, { ripple: true, theme: { preset: Material } })
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

**¿Por qué registrar componentes globalmente?** Así no necesitas importarlos en cada `.vue`. El prefijo `pv-` evita colisiones con elementos HTML nativos.

---

## PASO 20 — Configurar `app.vue`

```vue
<script setup>
import { useI18n } from 'vue-i18n'
import Layout from "./shared/presentation/components/layout.vue";

const { t } = useI18n()
</script>

<template>
  <layout/>
</template>
```

---

## PASO 21 — Iniciar todo

### Terminal 1 — json-server
```bash
json-server --watch server/db.json --routes server/routes.json
```

### Terminal 2 — Vue app
```bash
npm run dev
```

Abrir: `http://localhost:5173`

---

## PASO 22 — Empaquetar para entrega

Antes de comprimir, eliminar `node_modules/` para reducir el tamaño:

```bash
rm -rf node_modules
```

Comprimir el proyecto como `.zip`:
```
pc111990u202418655.zip
```

Para restaurar dependencias después de descomprimir:
```bash
npm install
```

---

## Resumen de la arquitectura

```
Petición del usuario
        ↓
  [Componente .vue]   ← solo presenta datos y emite eventos
        ↓
  [Store / application]  ← coordina casos de uso, gestiona estado reactivo
        ↓
  [Assembler]         ← traduce recursos del API a entidades del dominio
        ↓
  [API Service]       ← hace llamadas HTTP con axios
        ↓
  [json-server / API real]
```

---

## Errores comunes y cómo resolverlos

| Error | Causa probable | Solución |
|---|---|---|
| Puerto 3000 en uso | json-server ya corre | `kill -9 $(lsof -t -i:3000)` |
| Puerto 5173 en uso | Otra instancia de Vite | `npm run dev -- --port 5174` |
| `Cannot find module` | Ruta de import incorrecta | Verificar la ruta relativa exacta del archivo |
| Pantalla en blanco | Error en consola del navegador | Abrir F12 → Console y leer el error |
| Datos no aparecen | json-server no corre | Verificar `http://localhost:3000/api/v1/recipes` |
| Textos no se traducen | `legacy: false` falta en i18n.js | Agregar `legacy: false` al `createI18n` |

---
