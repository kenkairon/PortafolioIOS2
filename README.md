# CV estilo iOS 26 (Liquid Glass) — Carlos Vásquez

Portafolio/CV interactivo hecho con **Next.js 14 (App Router)**, **TypeScript** y **Tailwind CSS**, con una interfaz que imita el **Home Screen de iOS 26**: barra de estado con Dynamic Island, cuadrícula de apps tipo squircle, Dock inferior, "sheets" (hojas) que se deslizan desde abajo al abrir cada sección con el material **Liquid Glass**, y un **gato asistente** flotante que ayuda a navegar el portafolio.

## 📁 Estructura del proyecto

```
cv-win11/
├── app/
│   ├── layout.tsx        → layout raíz (metadata, fuente)
│   ├── page.tsx           → home screen: cuadrícula + dock + sheets + gato
│   └── globals.css        → material Liquid Glass, squircle, animaciones
├── components/
│   ├── StatusBar.tsx        → hora, Dynamic Island, señal/batería
│   ├── AppIcon.tsx            → ícono squircle con gradiente tipo iOS
│   ├── Dock.tsx                → dock inferior + home indicator
│   ├── AppSheet.tsx             → hoja modal que se desliza al abrir una app
│   ├── CatGuide.tsx              → gato asistente flotante + buscador
│   └── SkillBar.tsx               → barra de progreso de habilidades
├── lib/
│   ├── data.ts                     → TODA tu info del CV (edítala aquí)
│   └── storage.ts                    → persistencia en localStorage
├── tailwind.config.ts               → paleta de colores iOS
└── package.json
```

## 🚀 Paso a paso para correrlo

### 1. Requisitos
Node.js 18 o superior (https://nodejs.org).

### 2. Instalar y correr
```bash
npm install
npm run dev
```
Abre **http://localhost:3000**. Toca cualquier ícono de la cuadrícula o del dock inferior para abrir su "app". Toca al **gato** 🐱 (abajo a la izquierda) para buscar habilidades/proyectos o recibir consejos.

### 3. Editar tu información
Todo el contenido vive en `lib/data.ts` (perfil, habilidades, proyectos, certificaciones, contacto, wallpapers y colores de cada ícono).

### 4. Producción
```bash
npm run build
npm start
```

### 5. Publicarlo gratis (Vercel)
1. Sube el proyecto a GitHub.
2. Entra a https://vercel.com → conecta GitHub → importa el repo.
3. Vercel detecta Next.js automáticamente → "Deploy". En minutos tendrás una URL pública.

## ✨ Funciones incluidas

- **Home screen tipo iOS**: cuadrícula de 6 apps (Sobre mí, Habilidades, Proyectos, Certificados, Contacto, Ajustes) con íconos squircle degradados, más una barra de estado con Dynamic Island simulada.
- **Dock inferior** con las 4 apps principales, estilo "glass" translúcido con blur real, más el home indicator.
- **Sheets estilo iOS**: al tocar un ícono, la sección se desliza desde abajo con el material Liquid Glass (blur + saturación), barra de navegación con ícono/título y botón de cerrar (flecha hacia abajo).
- **Gato asistente ("Miu")** 🐱: personaje flotante con animación de rebote suave. Al tocarlo despliega una burbuja con:
  - Consejos rotativos sobre cómo usar el portafolio.
  - Un **buscador funcional** de habilidades y proyectos: escribe "Python" o "Django" y te muestra resultados; al elegir uno, abre la app correspondiente y hace scroll con un resaltado animado hasta el ítem.
- **App de Ajustes**: cambia el **fondo de pantalla** (5 wallpapers) y el **modo oscuro/claro**, con un botón para "Restablecer valores predeterminados".
- **Persistencia real con localStorage**: el fondo de pantalla y el modo oscuro elegidos se guardan en el navegador del visitante y se recuerdan en su próxima visita — esto funciona porque es una app real que corre en su navegador (no un artifact embebido), a diferencia de la vista previa dentro del chat de Claude.

## 🤖 Chatbot con Gemini (Miu responde sobre tus proyectos de GitHub)

Agregué una app nueva, **"Pregúntale a Miu"**, que:
1. Trae tus repositorios públicos desde la API de GitHub (sin necesidad de token).
2. Le pasa esa lista como contexto a **Gemini** y deja que el visitante haga preguntas ("¿qué tecnología usa tu proyecto X?", "¿cuál es tu repo más reciente?", etc.).

### Por qué se hace así (y su límite de seguridad)

Este sitio es **100% estático** (se exporta con `output: "export"` para GitHub Pages), así que no hay backend propio donde esconder una API key. Por eso el chat llama a Gemini **directamente desde el navegador**. Esto significa que la key queda visible en el código fuente del sitio — la forma de mitigarlo es **restringir la key para que solo funcione desde tu dominio**, como se explica abajo. No es 100% inviolable (nada que corra en el navegador lo es), pero sí evita que cualquiera la copie y la use en otro sitio.

Si en algún momento quieres una solución más robusta (key completamente oculta), la alternativa es mover esta llamada a una función serverless (ej. Vercel/Cloudflare Workers) en vez de GitHub Pages — pero para un portafolio personal con la key restringida, esto es suficiente.

### Paso a paso

**1. Configura tu usuario de GitHub**
En `lib/data.ts`, cambia:
```ts
export const githubConfig = {
  username: "TU_USUARIO_GITHUB", // ← pon tu usuario real
};
```

**2. Crea la API key de Gemini**
1. Entra a [Google AI Studio](https://aistudio.google.com/apikey) con tu cuenta de Google.
2. Clic en **"Create API key"** (puedes usar un proyecto nuevo de Google Cloud o uno existente).
3. Copia la key generada.

**3. Restringe la key a tu dominio (importante)**
1. Ve a [Google Cloud Console → APIs y servicios → Credenciales](https://console.cloud.google.com/apis/credentials) (mismo proyecto donde se creó la key).
2. Abre la key que acabas de crear.
3. En **"Restricciones de la aplicación"** elige **"Referentes HTTP (sitios web)"**.
4. Agrega tu dominio de GitHub Pages, por ejemplo:
   ```
  https://kenkairon.github.io/PortafolioIOS2/*
   ```
5. En **"Restricciones de API"**, limita la key solo a **"Generative Language API"**.
6. Guarda.

**4. Prueba localmente (opcional)**
Crea un archivo `.env.local` en la raíz del proyecto (no lo subas a git, ya está en `.gitignore`):
```
NEXT_PUBLIC_GEMINI_API_KEY=tu_api_key_aqui
```
Como la restricción por referrer de Google a veces bloquea `localhost`, durante pruebas locales puedes usar temporalmente una key sin restricción, y solo restringir la que subes a producción.

**5. Agrega la key como secreto en GitHub (para el deploy automático)**
1. En tu repositorio de GitHub → **Settings → Secrets and variables → Actions**.
2. **New repository secret**.
3. Nombre: `GEMINI_API_KEY`. Valor: tu API key.
4. Guarda. El workflow (`.github/workflows/deploy.yml`) ya está configurado para inyectarla como `NEXT_PUBLIC_GEMINI_API_KEY` durante el build:
   ```yaml
   - run: npm run build
     env:
       NEXT_PUBLIC_GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
   ```

**6. Haz commit y push** — el siguiente deploy ya incluirá el chatbot funcionando.

### Archivos involucrados
- `lib/github.ts` — trae tus repos públicos desde la API de GitHub.
- `lib/gemini.ts` — llama a la API REST de Gemini (`gemini-2.0-flash`) desde el navegador.
- `components/ChatBot.tsx` — la interfaz de conversación.
- `lib/data.ts` → `githubConfig.username` y `appColors.chat`.

## 🧩 Cómo agregar una nueva "app" (sección)

1. Agrega los datos en `lib/data.ts` (y un color en `appColors` si quieres un ícono nuevo).
2. Agrega la key al tipo `AppKey` en `app/page.tsx`.
3. Copia un bloque `<AppSheet>...</AppSheet>` existente, cámbiale el ícono/gradiente y el contenido.
4. Agrégalo al arreglo `grid` (cuadrícula) y, si quieres, a `dockApps` (dock inferior).

## 🐾 Ideas para seguir extendiendo
- Reemplazar el emoji del gato por una ilustración SVG personalizada o animada (Lottie).
- Agregar más "personajes" de presentación (ej. un compañero que muestre certificaciones).
- Simular "swipe" entre páginas del home screen si agregas más apps.
- Sonidos sutiles tipo iOS al abrir/cerrar sheets.

## 📦 Dependencias principales
- `next` 14.2.35 · `react` 18 · `tailwindcss` 3 · `lucide-react` (íconos)




