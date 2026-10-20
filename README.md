# Atenuación en Fibra

App de práctica sobre las **causas de la atenuación en fibra óptica** y los
**cálculos de pérdidas**, organizada en temas (sesiones) con preguntas de
**opción múltiple** y **problemas numéricos** (el estudiante escribe el
resultado y recibe la solución paso a paso). Cliente para celular (APK) y
servidor desplegable en una intranet **o** en Vercel (plan Hobby).

Es una **plantilla editable**: el contenido vive en archivos JSON, uno por
tema, que puedes modificar sin tocar el código — ver
[`server/data/README.md`](server/data/README.md).

> **Importante sobre el contenido incluido.** Las 40 preguntas de ejemplo
> están redactadas a partir de la teoría estándar de atenuación en fibras
> (unidades, absorción, esparcimiento, curvaturas, curva espectral,
> presupuesto de pérdidas). **No se redactaron viendo las páginas del libro
> de Keiser** (cap. 3, secs. 3.1 y 3.3), así que antes de usarlas en clase
> conviene contrastar con el libro: notación, valores típicos, fórmulas y
> el orden de los temas. Cada pregunta admite un campo opcional
> `reference` (p. ej. «Keiser, sec. 3.1, p. 105») que se muestra al
> estudiante como «Para repasar»; complétalo tras verificar la página.

```
fiber-attenuation-qa/
├── public/         HTML/CSS/JS del cliente — fuente única, sirve tanto
│   │               para Vercel (sitio estático) como para el APK
│   └── img/        Imágenes para las preguntas (incluye el esquema espectral)
├── client/         Configuración de Capacitor que empaqueta public/ en un APK
├── server/
│   ├── app.js      La app Express (rutas de la API) — sin escuchar puerto
│   ├── server.js   Arranque para intranet: usa app.js y escucha en 0.0.0.0
│   ├── validate.js Revisa los archivos de temas (npm run validate)
│   ├── lib/        Carga y validación del contenido
│   └── data/
│       ├── app.json            Textos del encabezado de la app
│       └── sessions/           sesion-01.json ... sesion-10.json (+ plantilla)
├── api/index.js    Punto de entrada para Vercel: exporta la misma app.js
├── vercel.json     Reenvía /api/* a la función; public/ se sirve en la raíz
├── package.json    Dependencias y scripts (npm run validate)
└── .devcontainer/  Entorno Theia/Docker con JDK + Android SDK preinstalados
```

`server/app.js` es la única definición de la API: `server/server.js`
(intranet) y `api/index.js` (Vercel) la reutilizan tal cual.

## Cómo funciona

1. El estudiante elige un tema, responde sus preguntas (cada una aparece
   una sola vez, en orden aleatorio por defecto) y recibe corrección y
   explicación inmediatas. Al terminar ve su resultado y puede repetir el
   tema o pasar al siguiente.
2. **Opción múltiple**: toca la opción; se marca la correcta y se explica.
3. **Cálculo**: escribe un número (acepta coma o punto decimal) y pulsa
   «Comprobar». Se acepta dentro de una tolerancia configurable (por
   defecto ±2 %), puede pedir una pista antes de responder y, después,
   ve la solución paso a paso. Si escribe algo que no es un número, la app
   se lo indica sin contarlo como intento.
4. Algunas preguntas llevan imagen (se puede ampliar tocándola).
5. El backend corrige en el servidor: las respuestas correctas no viajan
   al celular hasta que el estudiante responde.

## Puesta en marcha rápida — intranet

1. **Servidor**: ver [`server/README.md`](server/README.md) — instalar
   dependencias, arrancar, anotar la IP de la máquina y abrir el puerto
   3000 en el firewall.
2. **Cliente**: ver [`client/README.md`](client/README.md) — apuntar
   `public/js/config.js` a esa IP, probar en el navegador, y luego
   compilar el APK con Android Studio (o con Theia IDE / línea de
   comandos usando el entorno preconfigurado en
   [`.devcontainer/`](.devcontainer/README.md)).

## Puesta en marcha rápida — Vercel (plan Hobby)

Desplegar en Vercel cambia el modelo: en vez de un servidor solo
alcanzable dentro de la red del campus, obtienes una URL pública
(`https://tu-proyecto.vercel.app`) con HTTPS automático, accesible desde
cualquier red (Wi-Fi de casa, datos móviles, etc.) — no solo la intranet.
A cambio, el plan Hobby está pensado para uso personal/no comercial y el
sistema de archivos del servidor pasa a ser de **solo lectura**: no hay
forma de editar los archivos de sesiones en caliente como con `/api/admin/reload`
en intranet — actualizar el contenido significa editar los archivos, hacer
commit y volver a desplegar (Vercel lo hace automático si conectas un
repo de Git).

### 1. Sube el proyecto a un repositorio Git

Vercel despliega desde GitHub, GitLab o Bitbucket. Si aún no es un repo:

```bash
cd fiber-attenuation-qa
git init
git add .
git commit -m "Atenuación en Fibra"
```

y súbelo a GitHub (crea el repo vacío en GitHub primero, luego
`git remote add origin <url>` y `git push -u origin main`).

### 2. Importa el proyecto en Vercel

1. Entra a [vercel.com](https://vercel.com) e inicia sesión (puedes usar
   tu cuenta de GitHub directamente).
2. **Add New → Project**, selecciona el repositorio.
3. `vercel.json` ya fija el *Framework Preset* en «Other» y la carpeta
   de salida en `public/`: no hace falta cambiar nada en «Build & Output
   Settings». Importa el repositorio desde su **raíz** (ver «404» más abajo).
4. **Deploy**. En un par de minutos tendrás la URL pública.

Alternativa por línea de comandos, si lo prefieres:

```bash
npm install -g vercel
cd fiber-attenuation-qa
vercel        # sigue las preguntas; crea un preview
vercel --prod # despliega a producción
```

### 3. Verifica que la API responde

```bash
curl https://tu-proyecto.vercel.app/api/health
```

Debería devolver `{"status":"ok","sessionCount":6,"warnings":0}`. Abrir
`https://tu-proyecto.vercel.app` directamente en el navegador debería
mostrar la app funcionando igual que en intranet.

### 4. Apunta el cliente (y el APK) a esa URL

Edita `public/js/config.js` y cambia `FALLBACK_API_BASE_URL` por tu URL
de Vercel — esto afecta al APK, que no tiene forma de detectar la URL
automáticamente (ver [`client/README.md`](client/README.md)):

```js
const FALLBACK_API_BASE_URL = "https://tu-proyecto.vercel.app/api";
```

Como Vercel sirve todo por HTTPS, **puedes saltarte** el paso del
`AndroidManifest.xml` (`usesCleartextTraffic`) que sí hace falta para un
backend de intranet en HTTP plano.

### 5. Actualizar sesiones después de desplegar

Edita los archivos de `server/data/sessions/` (y, si agregaste imágenes
locales, súbelas a `public/img/`). Antes de publicar, revísalos:

```bash
npm install          # solo la primera vez
npm run validate
```

Luego:

```bash
git add server/data public/img
git commit -m "Actualiza sesiones"
git push
```

Vercel vuelve a desplegar automáticamente al detectar el push (o corre
`vercel --prod` de nuevo si desplegaste por CLI). No necesitas recompilar
el APK — solo el backend cambia.

### Si ves «404: NOT_FOUND» al abrir la URL

Revisa, en este orden:

1. **Carpeta raíz del repositorio.** `vercel.json`, `api/`, `public/` y
   `server/` deben estar en la **raíz del repositorio**. Si al subir el zip
   quedó una carpeta extra (`fiber-attenuation-qa/` dentro del repo), o importas
   solo una subcarpeta, Vercel no encuentra nada y responde 404. Solución: mueve
   el contenido a la raíz, o en Vercel entra a **Settings → General → Root
   Directory** y escribe el nombre de esa carpeta.
2. **Configuración antigua.** Las primeras versiones de este proyecto
   usaban `builds` y `routes` en `vercel.json`; con esa forma, la página
   principal (`/`) devolvía 404. Asegúrate de que tu `vercel.json` sea
   exactamente el de este proyecto (sin `builds` ni `routes`) y vuelve a
   desplegar.
3. **Ajustes del proyecto en Vercel.** En **Settings → Build &
   Development Settings**, el *Framework Preset* debe ser **Other** (este
   `vercel.json` ya lo fuerza) y no debe haber un *Output Directory*
   distinto de `public`.
4. **Comprobar qué se desplegó.** En el despliegue, pestaña **Source** (o
   *Build Logs* → *Deployment Summary*): debe verse `index.html` dentro de
   *Static Assets* y una función `api/index.js` en *Functions*.
5. **Probar la API aparte.** `https://TU-PROYECTO.vercel.app/api/health`
   debe responder JSON. Si la API responde pero la página no, el problema
   es de archivos estáticos (puntos 1 a 4); si la página carga pero la API
   no, revisa *Runtime Logs* del despliegue.

### Límites a tener presentes en el plan Hobby

- Pensado para uso personal/no comercial (revisa los
  [términos de Vercel](https://vercel.com/docs/limits/overview) si el
  uso institucional te genera dudas).
- Los proyectos inactivos por un tiempo prolongado pueden "dormir" el
  primer request (cold start) — el siguiente ya responde rápido.
- Suficiente ancho de banda y ejecuciones de función para el tráfico de
  un curso; si el uso crece mucho (miles de estudiantes concurrentes),
  revisa los límites exactos en la documentación de Vercel.

## Alcance de esta versión

No incluye (a propósito):

- Panel de administración: el contenido se edita en los archivos JSON de
  `server/data/sessions/`.
- Cuentas de estudiante ni seguimiento de calificaciones: el resultado se
  muestra solo mientras se usa la app y no se guarda en el servidor.
- Números aleatorios por intento: cada problema numérico tiene valores
  fijos. Generar variantes para evitar que se memoricen las respuestas es
  una extensión posible.

## Contenido incluido

6 temas, 40 preguntas (27 de opción múltiple y 13 de cálculo):

1. Definición y unidades de la atenuación (dB/km, nepers, dBm)
2. Pérdidas por absorción (intrínseca y extrínseca, pico de OH⁻)
3. Esparcimiento (scattering) de la luz (Rayleigh y Mie, dependencia 1/λ⁴)
4. Curvaturas e imperfecciones de la guía (macro y microcurvatura)
5. Atenuación espectral y ventanas de transmisión (con esquema ilustrado)
6. Cálculo de pérdidas en un enlace (presupuesto, potencia recibida, margen)

La imagen del tema 5 (`public/img/atenuacion-espectral.svg`) es un esquema
**cualitativo** de elaboración propia, no una gráfica medida.
