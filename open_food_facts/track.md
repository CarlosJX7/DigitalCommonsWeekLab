# Open Food Facts — guía del track

El enunciado corto está en [`subject.md`](subject.md). Esta guía lo desarrolla: qué es cada reto, con qué está hecho, qué conviene aprender antes, qué hace falta tener instalado y cómo usar la IA. Los retos van de menor a mayor dificultad. Antes de elegir uno está la sección de conceptos fundamentales: es el vocabulario común del track.

La dificultad mide tres cosas a la vez: cuánta infraestructura local hay que levantar, cuánto dominio específico pide el reto (datos alimentarios, búsqueda, sincronización, visión) y cuánto cuidado exige tocar datos que ya consultan millones de personas. Un cambio de texto en un juego de anotación se prueba en una tarde. Un import de medio millón de productos, o un modelo que rellena alérgenos, pide otra preparación.

## Conceptos fundamentales

Esto no es un reto. Es lo que hace falta reconocer para poder empezar cualquiera de ellos. Si un apartado de aquí no te suena, párate antes de clonar un repo.

Van de lo más general a lo más cercano al código del track.

### La terminal y los comandos

La terminal es la ventana de texto donde se lanzan órdenes. En Linux y macOS se llama Terminal. En Windows sirve PowerShell o, mejor para estos proyectos, WSL (un Linux dentro de Windows).

Un comando es una orden seguida de argumentos. Estos seis cubren el día a día:

```bash
pwd            # carpeta en la que estás
ls             # qué hay dentro
cd nombre      # entrar en una carpeta
cd ..          # subir un nivel
mkdir prueba   # crear una carpeta
cat fichero    # ver el contenido de un fichero
```

En este track vas a teclear, una y otra vez:

- `git clone …` para bajarte un repositorio.
- `npm install`, `yarn`, `pnpm install` o `pip install` para instalar las librerías que el proyecto declara.
- `npm run dev`, `yarn dev` o `make up` para arrancarlo en tu máquina.
- `curl` para pedir una URL y ver la respuesta en crudo, sin navegador.

`http://localhost:3000` (el puerto cambia) es un servidor en tu propio ordenador. `https://world.openfoodfacts.org` es el servicio público. Confundirlos es la forma más rápida de mezclar pruebas con datos reales.

Una variable de entorno es un valor con nombre que el programa lee al arrancar: una URL, un `User-Agent`, una clave. Casi todos los repos traen un `.env.example`. Se copia a `.env` y se rellena. Ese fichero se queda en tu máquina: git no debe subirlo.

### Git y GitHub

Git guarda la historia del código. GitHub es donde Open Food Facts publica esa historia. Cada proyecto es un repositorio (repo).

El ciclo de un cambio es este:

```bash
git clone https://github.com/openfoodfacts/hunger-games.git
cd hunger-games
git checkout -b mi-cambio
# editas ficheros
git status
git diff
git add fichero
git commit -m "Explica el cambio en una frase"
git push -u origin mi-cambio
```

- **clone** descarga el repo y su historia.
- **rama** (`checkout -b`) aísla tu trabajo de la rama principal (`main` o `master`, según el repo).
- **status** y **diff** enseñan qué has tocado antes de guardarlo.
- **commit** es una foto del cambio, con un mensaje que dice para qué sirve.
- **push** sube la rama a GitHub.
- **pull request** (PR) pide incorporar tu rama. Ahí comentan los mentores.
- **issue** es una tarea o un fallo. La etiqueta `good first issue` marca un primer cambio de tamaño razonable.
- **pull** trae los commits nuevos de otras personas. Si la rama principal avanzó mientras trabajabas, hay que traerla antes de dar la PR por lista.

El mensaje del commit y el texto de la PR se leen meses después. Una frase concreta (“Muestra el alérgeno vacío en la ficha”) vale más que “cambios” o “fix”.

### La web

Una web es un cliente pidiendo recursos a un servidor. El cliente habitual es el navegador. La dirección es una URL:

`https://world.openfoodfacts.org/api/v3/product/3017620422003.json`

- El esquema (`https`) es el protocolo.
- El dominio (`world.openfoodfacts.org`) es el servidor.
- El camino (`/api/v3/product/3017620422003`) señala un recurso. Aquí, un producto identificado por su código de barras.
- Lo que va tras `?` son parámetros (`?fields=code,product_name`).

**Frontend** es lo que se ve y se toca. **Backend** es el programa que guarda datos y contesta. En este track el frontend cubre Explorer, Hunger Games, el frontend de Nutri-Patrol, el de Open Prices y los web components. El backend cubre Product Opener, Robotoff, Search-a-licious, Open Prices y Nutri-Patrol.

**HTML** marca la estructura de la página, **CSS** el aspecto y **JavaScript** el comportamiento. **TypeScript** es JavaScript con tipos: el compilador avisa cuando pasas un número donde tocaba un texto. React, Svelte y Vue son formas de construir esa interfaz a partir de componentes. Un **web component** es una etiqueta HTML propia, reutilizable entre webs distintas; aquí se escriben con Lit.

Las herramientas de desarrollador del navegador se abren con F12. La pestaña Red (Network) lista cada petición que hace la página: la URL, el código de respuesta y el JSON. Es la forma directa de ver a qué API llaman Explorer o Hunger Games.

### HTTP, API y JSON

**HTTP** es el protocolo de esas peticiones.

| Verbo | Para qué se usa aquí |
| --- | --- |
| `GET` | Leer un producto, una búsqueda, una pregunta. No modifica nada |
| `POST` | Crear o enviar: una foto, una anotación, un precio, un evento |
| `PUT` / `PATCH` | Sustituir o actualizar un recurso que ya existe |

La respuesta llega con un **código de estado**:

| Código | Significado práctico |
| --- | --- |
| 200 | La petición ha ido bien |
| 400 | El servidor no entiende lo que has mandado |
| 401 | Falta identificarte |
| 403 | Estás identificado y no tienes permiso |
| 404 | Esa URL no existe |
| 500 | El servidor ha fallado por dentro |

Las **cabeceras** viajan junto a la petición. Open Food Facts pide un `User-Agent` con el nombre de tu proyecto y una forma de contacto. Sin eso, un uso intensivo se confunde con un cliente anónimo y se puede limitar.

Una **API** es un contrato para que un programa hable con otro, con una respuesta pensada para ser leída por código. Un **endpoint** es una de esas URLs concretas. La API de Open Food Facts habla **JSON**: texto con objetos `{}`, listas `[]`, textos, números y `null`.

```bash
curl -H "User-Agent: DigitalCommonsWeekLab - tu-email@ejemplo.org" \
  "https://world.openfoodfacts.org/api/v3/product/3017620422003.json?fields=code,product_name"
```

Esa orden lee un producto. `null`, o la ausencia de un campo, significa que nadie lo ha rellenado. Una cadena vacía guardada a propósito y un error HTTP son otras dos situaciones.

Un **SDK** es una librería que llama a la API por ti y te devuelve objetos de un lenguaje (Dart, JavaScript, Python). Conviene usarlo en cuanto hagas algo más que un `curl`.

**REST**, en la práctica de estos repos, quiere decir: el recurso va en la URL, el verbo HTTP dice la acción y el cuerpo va en JSON.

La **autenticación** dice quién hace la llamada. En el track aparece así:

- Leer la API pública, en general, no pide cuenta.
- Escribir un producto, anotar con tu nombre o subir un precio pide una cuenta de Open Food Facts.
- La **cookie de sesión** es lo que el navegador guarda tras el login. Un frontend en `localhost` no recibe la cookie de `openfoodfacts.org`: por eso varios repos documentan un modo de desarrollo.
- **OAuth** (Keycloak, en Explorer) entrega un permiso al frontend. La contraseña se queda en el proveedor de identidad.

**CORS** es la regla del navegador que impide a una página en `localhost` leer la respuesta de otro dominio. Si Hunger Games falla en local con un error de CORS, el fallo es esa regla, no tu código de pintado.

### Datos, entornos y colas

Un **CSV** es una tabla en texto: una fila por línea y columnas separadas por coma o por tabulador. Los productores mandan así sus catálogos. La primera fila suele ser el nombre de cada columna. El encoding que pide OFF es UTF-8.

Una **base de datos** guarda el estado del servicio:

- **MongoDB** guarda el producto vigente de Product Opener.
- **PostgreSQL** lo usan Robotoff, Open Prices y Nutri-Patrol.
- **Elasticsearch** es un índice para buscar texto, facetas y parecidos. La fuente que manda sigue siendo la base del servicio, no el índice.

**Staging** (`openfoodfacts.net`) es el entorno de prueba. **Producción** (`openfoodfacts.org`) es el que consulta la gente. Las escrituras de ensayo van a staging.

Un **dump** es un volcado de esa base (a menudo en JSONL: un JSON por línea) para trabajar en local.

**Docker** empaqueta un programa con el sistema y los servicios que necesita, de forma que arranquen igual en otra máquina. `docker compose up`, o el `make up` que lo envuelve, es la orden típica. La primera vez descarga imágenes y tarda.

**Redis**, aquí, hace de cola y de canal de avisos: “este producto acaba de cambiar”. Search-a-licious y Robotoff se enteran por ahí, sin que cada uno interrogue a Product Opener a ciegas.

### Lenguajes que vas a encontrarte

No hace falta dominarlos todos. Hace falta saber cuál abre el repo que tienes delante, e instalar su versión (viene en el `README`, en `.nvmrc` o en `pyproject.toml`).

| Si abres… | Trabajas con… |
| --- | --- |
| Explorer, web components, Hunger Games, frontends de Nutri-Patrol y Open Prices, frontend de Search-a-licious | JavaScript o TypeScript. El gestor es `npm`, `yarn` o `pnpm`, el que diga el repo |
| Robotoff, la API de Search-a-licious, Nutri-Patrol, Open Prices, Events, Folksonomy | Python 3. El gestor es `pip`, Poetry o `uv` |
| smooth-app y el SDK de Dart | Dart y Flutter. La versión de Flutter la fija FVM |
| Product Opener y los imports masivos | Perl |

`npm`, `yarn` y `pnpm` resuelven el mismo problema (instalar paquetes de JavaScript) y no se intercambian dentro de un repo: si hay `pnpm-lock.yaml`, el comando es `pnpm`.

### Cómo se colabora

- El `README` se lee antes de instalar. Ahí están la versión del lenguaje, el comando de arranque y el canal de Slack.
- Una duda de producto se pregunta en el canal, con lo que ya has probado y el error literal.
- La PR toca un problema, se puede ejecutar y dice cómo probarla.
- Los datos de comida los mira gente real para decidir una compra. Un producto de prueba, un precio inventado o un alérgeno mal grabado se queda en staging.

Con esto se puede seguir el resto de la guía. Lo que viene ahora ya es específico de Open Food Facts.

## Qué problema tiene delante el track

Open Food Facts es una base colaborativa de productos alimentarios (y proyectos hermanos: cosmética, comida para mascotas, productos genéricos). Cualquiera fotografía un envase, y de esas fotos salen ingredientes, alérgenos, tabla nutricional, Nutri-Score, NOVA y Green-Score. La base se publica como datos abiertos y la reutilizan apps, investigadores y servicios públicos.

El problema de fondo es conseguir que los datos estén completos, sean correctos y se puedan consultar. Casi todos los retos del subject responden a una parte de ese problema:

| Reto | Qué desbloquea |
| --- | --- |
| Food API & AI | Que un asistente responda citando los datos de OFF |
| Hunger Games y Nutri-Patrol | Que una persona corrija muchos productos en pocos minutos |
| Score my recipe | Que una receta casera tenga puntuación ambiental, no solo un producto envasado |
| Explorer y web components | La interfaz web de referencia, usable en el móvil |
| Search-a-licious | Encontrar productos por texto, facetas y gráficos |
| App móvil | Seguir escaneando y editando sin red, y volver a sincronizar |
| Open Prices | Precios e inflación a partir de tickets y etiquetas |
| Data imports | Meter catálogos enormes de productores sin romper la base |
| Robotoff | Convertir fotos en datos estructurados a escala |
| Rapid Product Acquisition | Dar de alta un producto desde una foto en menos de un minuto |

## Mapa del ecosistema

Casi todo cuelga de **Product Opener** ([`openfoodfacts-server`](https://github.com/openfoodfacts/openfoodfacts-server)): el servidor en Perl que guarda los productos, sirve la web clásica y la API. El producto vigente vive en MongoDB. El historial de cada versión se guarda en ficheros (Storable de Perl). Apache sirve las imágenes y hace de proxy hacia mod_perl. Cada cambio se publica en un stream de Redis, y de ahí se enteran el buscador y Robotoff.

Alrededor hay servicios con su propia base y su propio despliegue:

| Pieza | Para qué sirve | Stack habitual |
| --- | --- | --- |
| Product Opener | Productos, API, web clásica, imports | Perl, MongoDB, Apache, Redis |
| API pública | Leer y escribir productos | HTTP JSON. v3 es la actual (v3.6 añade el esquema de tags). v2 sigue viva y es la que tiene búsqueda estructurada |
| Robotoff | Predicciones e insights (propuestas de dato) a partir de fotos y OCR | Python, PostgreSQL, Redis + rq, Elasticsearch, Triton |
| Search-a-licious | Búsqueda full-text, facetas y gráficos | Python 3.11, FastAPI, Elasticsearch, Redis, Lit, Vega |
| Explorer | Frontend nuevo | SvelteKit, TypeScript, Vite, Tailwind, DaisyUI, pnpm |
| Web components | Piezas de UI compartidas (edición, búsqueda, Robotoff, Folksonomy) | Lit |
| smooth-app | App oficial Android / iOS | Flutter y Dart. La versión de Flutter va fijada con FVM en el repo |
| Open Prices | Precios colaborativos | Django, Python 3.11, PostgreSQL, Vue 3. Fotos de tickets y etiquetas analizadas con Triton y Gemini |
| Nutri-Patrol | Cola de moderación | FastAPI, Peewee, PostgreSQL. Frontend en React + Vite |
| Hunger Games | Minijuegos de anotación | React, Vite, MUI, TypeScript |
| Folksonomy | Atributos libres sobre un producto | Python, FastAPI |
| Knowledge Panels | Bloques listos para pintar Nutri-Score, Green-Score, etc. | Python, FastAPI |
| Events | Puntos, ranking e insignias | FastAPI. Existe, y todavía no es el backend de gamificación en producción |
| SDKs | La misma API envuelta en varios lenguajes | Dart, JS, Python, y otros (Java, PHP, Rust, Go, Elixir…) |

Producción es `openfoodfacts.org`. Staging es `openfoodfacts.net`. Las pruebas de escritura van a staging. La base de producción la consulta gente para decidir qué come, así que un producto de prueba, un precio inventado o un alérgeno mal grabado se ensaya en `.net`.

Las lecturas de la API son abiertas. Las escrituras piden cuenta. Las llamadas llevan un `User-Agent` que identifica la app. El código de los repos principales está en AGPL-3.0. Los datos de Open Prices están en ODbL: hay que citar la fuente, y solo se republican junto a datos que también puedan quedar en abierto.

Cuenta, Slack y documentación:

- Cuenta en <https://world.openfoodfacts.org>
- Slack: <https://slack.openfoodfacts.org>
- Documentación: <https://openfoodfacts.github.io/documentation/>
- API: <https://openfoodfacts.github.io/openfoodfacts-server/api/>
- Organización: <https://github.com/openfoodfacts>

## 0. Base común

Con la sección anterior ya se puede leer un repo. Esta parte es el dominio de Open Food Facts y hace falta en todos los retos.

### Qué aprender, de menos a más

1. **Leer un producto.** `GET https://world.openfoodfacts.org/api/v3/product/3017620422003.json` devuelve nombre, marca, ingredientes, nutrientes por 100 g, alérgenos, categorías, Nutri-Score, NOVA y Green-Score. El campo `code` es el código de barras. Muchos campos vacíos significan “nadie lo ha rellenado”, que es distinto de “el producto no lo tiene”.
2. **Los tres sistemas de puntuación.** Nutri-Score (calidad nutricional, A–E), NOVA (grado de procesado, 1–4) y Green-Score (impacto ambiental: análisis de ciclo de vida de Agribalyse, más bonus y malus —bonificaciones y penalizaciones— por etiquetas, origen, envase e ingredientes problemáticos). En la API el grado ambiental sigue apareciendo como `ecoscore_grade` / `environmental_score_grade`.
3. **Taxonomías.** Categorías, etiquetas, alérgenos y países no son texto libre: son tags (`en:beverages`, `en:milk`). Folksonomy añade propiedades que todavía no están en la taxonomía.
4. **El ciclo de una contribución.** Foto → OCR (reconocimiento del texto en la imagen) y modelos (Robotoff) → insight → una persona lo confirma (app, Hunger Games, web) → Product Opener actualiza el producto → el aviso en Redis llega al resto de servicios.
5. **Conocer los tres clientes reales.** La web (`world.openfoodfacts.org`), Explorer y la app Smoothie. Instálate la app y escanea algo de tu cocina antes de abrir un repo.

### Qué necesitas

- Terminal, git y `curl`, como en los conceptos fundamentales, y un editor.
- Cuenta de Open Food Facts y cuenta de GitHub.
- Slack, con una presentación de dos líneas: qué sabes y qué reto te interesa.
- Docker, cuando pases del nivel de “solo API”.

### Uso de IA

Pídele a un modelo que te explique un JSON de producto campo a campo, o que te resuma el hilo de un issue. La comprobación es tuya: abre el campo en la respuesta de la API. Para alérgenos, nutrientes y puntuaciones, el texto que vea una persona tiene que salir del campo de la API, con el código de barras al lado. Un modelo que “completa” una tabla nutricional está fabricando un dato sanitario.

Ejercicio de una hora: elige tres productos (uno muy completo, uno casi vacío, uno con Green-Score), pide `?fields=code,product_name,nutriscore_grade,nova_group,ecoscore_grade,allergens_tags,ingredients_text` y anota qué falta en cada uno. Esa lista es el backlog real del track.

---

## 1. Food API & AI (LLMs)

**Enunciado.** Dejar Open Food Facts como fuente de verdad para aplicaciones de IA, con un servidor MCP mantenido por la comunidad.

**Por qué es el primer reto.** No hace falta clonar Product Opener. La API de lectura ya está en producción. Un servidor [MCP](https://modelcontextprotocol.io) (Model Context Protocol) expone herramientas que un cliente (Claude, un IDE, un agente) puede llamar: buscar un producto, comparar dos códigos de barras, explicar un Nutri-Score. Hay servidores comunitarios de referencia ([`cyanheads/openfoodfacts-mcp-server`](https://github.com/cyanheads/openfoodfacts-mcp-server) y otros). El trabajo del hackathon es uno que la comunidad pueda mantener: herramientas pequeñas, esquemas estrictos, pruebas, y datos que siempre vuelven a OFF.

### Tecnologías

- HTTP y JSON contra la API v3 (lectura) y, si hace falta búsqueda estructurada, v2 `/api/v2/search`. El full-text nuevo vive en Search-a-licious (`search.openfoodfacts.org`), no en Product Opener.
- Un SDK oficial para no parsear el JSON a mano: [`openfoodfacts-js`](https://github.com/openfoodfacts/openfoodfacts-js), el paquete Python `openfoodfacts`, o [`openfoodfacts-dart`](https://github.com/openfoodfacts/openfoodfacts-dart).
- El SDK de MCP en TypeScript o Python.
- Knowledge Panels, si quieres devolver un bloque ya redactado y traducido en lugar de reinterpretar los campos.

### Qué aprender, de menos a más

1. Diseñar una herramienta con entrada y salida pequeñas: `get_product(barcode)`, `search(query, filters)`, `compare(barcodes)`.
2. Escribir el esquema (qué campos vuelves, cuáles pueden ser `null`).
3. Citar siempre `code`, `product_name` y la URL del producto.
4. Autenticación y escritura: crear o editar un producto pide cuenta, y el usuario tiene que confirmar antes de publicar.
5. Evaluación: una lista fija de preguntas (“¿este producto lleva leche?”, “compara estos dos barcodes”) con la respuesta esperada leída de la API.

### Qué necesitas

- Node o Python, y una clave o un cliente MCP para probar en local.
- No hace falta Docker ni la base de datos.
- Un `User-Agent` con nombre del proyecto y un contacto.

### Uso de IA

1. **Herramientas de solo lectura.** El modelo pide el producto y redacta. Si el campo viene vacío, la respuesta dice que OFF no lo tiene.
2. **Comparar y filtrar.** “Entre estos tres, el de menos sal” se resuelve con números de la API, no con la memoria del modelo.
3. **Lenguaje natural a filtros.** “vegano, sin aceite de palma, Nutri-Score A” se traduce a tags que existen. Si el tag no está en la taxonomía, no se inventa.
4. **Escribir con confirmación.** El modelo propone unos ingredientes o una categoría; la persona publica. Cada escritura queda atribuida a su usuario de OFF.
5. **Evaluar el servidor.** Un conjunto de preguntas reales y un diff cuando alguien cambia una herramienta. Eso es lo que permite que la comunidad lo mantenga.

Primer corte razonable para el hackathon: cinco herramientas de lectura, una página que explique qué no hace el servidor, y diez preguntas de evaluación en el repo.

---

## 2. Hunger Games

**Enunciado.** Junto con Nutri-Patrol: prototipar mecánicas de juego que aceleren las correcciones. Hunger Games es la mitad “anotar predicciones”.

**Por qué va aquí.** Es un frontend React que habla con APIs que ya están desplegadas. `yarn dev` y estás dentro. El dominio (qué es un insight, cuándo se aplica solo) se aprende usándolo.

Hunger Games ([`hunger-games`](https://github.com/openfoodfacts/hunger-games), <https://hunger.openfoodfacts.org>) son minijuegos: preguntas de Robotoff, validar logos, completar un atributo con un clic. La idea es que cualquier persona anote productos en unos minutos, en el escritorio o en el móvil. Robotoff genera la predicción; el juego recoge el sí o el no; si la persona está autenticada, el dato entra en el producto al momento. Sin sesión, hacen falta varios votos coherentes.

### Tecnologías

- React, TypeScript, Vite, MUI, Axios, Yarn. Se despliega en Netlify.
- APIs de Open Food Facts y de [Robotoff](https://github.com/openfoodfacts/robotoff) (`/questions`, `/insights`, anotación).
- [`openfoodfacts-js`](https://github.com/openfoodfacts/openfoodfacts-js) y [`openfoodfacts-webcomponents`](https://github.com/openfoodfacts/openfoodfacts-webcomponents).
- Canal de Slack: `#hunger-games`.

### Qué aprender, de menos a más

1. Jugar una partida en producción y mirar cómo cambia (o no) el producto.
2. Arrancar el repo: Node (versión del `.nvmrc`), `yarn install`, `yarn dev`. En local a veces hace falta una extensión que relaje CORS.
3. Seguir una pregunta desde la UI hasta la llamada a Robotoff.
4. Añadir un juego nuevo: el README invita exactamente a eso.
5. Rendimiento: hay issues abiertos de degradación en el servidor cuando el juego pide muchas preguntas.

### Qué necesitas

- Node y Yarn.
- Cuenta de OFF, para ver la diferencia entre anotar con sesión y sin ella.
- Ganas de mirar issues con etiqueta `good first issue`.

### Uso de IA

1. **Redactar la pregunta.** A partir del tipo de insight (“¿es este logo el de la marca X?”), generar un texto más corto y un ejemplo visual. El texto se revisa; la predicción sigue siendo la de Robotoff.
2. **Ordenar la cola.** Un modelo pequeño, o una heurística, pone delante las preguntas donde un humano desbloquea más campos (categoría que permite calcular el Green-Score, por ejemplo).
3. **Agrupar duplicados.** La misma duda repetida en veinte productos se presenta una vez.
4. **Explicar la duda.** “El OCR leyó esto; el modelo propone esta etiqueta; esto es lo que no cuadra.” La persona sigue votando.
5. **Medir la mecánica.** Si añades puntos, usa el servicio de Events (ranking e insignias, descrito en el reto 4) y hazlo en staging. Un juego que anota mal a gran velocidad empeora la base.

---

## 3. Nutri-Patrol

**Enunciado.** La otra mitad del reto de gamificación: moderar correcciones y reportes, no solo confirmar predicciones.

**Por qué sube un peldaño.** Son dos repos, hay autenticación de verdad y el flujo distingue a un moderador de un usuario normal. Aun así se puede empezar por el frontend contra la API de producción.

Nutri-Patrol (<https://nutripatrol.openfoodfacts.org>) es la cola de calidad. Llegan reportes de la web clásica, de Explorer, de Robotoff (por ejemplo avisos de Cloud Vision) y, cuando se conecte, de la app. Un moderador revisa el ticket, lo marca como arreglado o como falso positivo. El README del backend dice que hoy no tiene mantenedor fijo: hay sitio, y también menos camino marcado.

- Backend: [`nutripatrol`](https://github.com/openfoodfacts/nutripatrol). FastAPI, Peewee, PostgreSQL, Docker (`make up`), pytest contra SQLite sin levantar el stack.
- Frontend: [`nutripatrol-frontend`](https://github.com/openfoodfacts/nutripatrol-frontend). React y Vite.
- También hay un web component de “flag product” y métodos en los SDKs de JS y Dart.
- Slack: `#nutripatrol`. Wiki de calidad: <https://wiki.openfoodfacts.org/Data_quality>.

### Qué aprender, de menos a más

1. Abrir la instancia pública y mirar un ticket (imagen, producto, origen `web` / `mobile` / `robotoff`).
2. Levantar el backend con Docker y el frontend con `npm run dev`.
3. Entender la sesión. La cookie de OFF no llega a `localhost`. En local existen cabeceras `X-Dev-User-Id` y `X-Dev-Moderator`, que solo deben existir en tu máquina. Los endpoints que editan el producto de verdad siguen necesitando la cookie de sesión.
4. Permisos: sin moderador, solo ves tus flags; con moderador, la cola entera.
5. Nuevas mecánicas de juego sobre esa cola: filtros, rachas, “arregla el campo que falta”, prioridad por impacto (alérgeno por encima de una foto torcida).

### Qué necesitas

- Docker, Python 3, Node.
- Cuenta de OFF. Para probar escrituras reales contra Product Opener, la cookie de sesión y, mejor, staging.
- El wiki de data quality, para no inventar una regla que ya existe.

### Uso de IA

1. **Resumir el diff** de un producto reportado: qué cambió, quién, qué campo.
2. **Priorizar la cola** con señales que ya existen (flag de Robotoff, producto muy escaneado, campo de alérgeno tocado).
3. **Proponer el arreglo** (unidad nutricional absurda, categoría demasiado genérica) como borrador que el moderador acepta.
4. **Detectar ráfagas** de ediciones parecidas desde la misma cuenta.
5. **Dejar el cierre en manos del moderador.** Cerrar un ticket aplica o descarta una corrección sobre datos públicos. El modelo prepara el resumen y el borrador.

---

## 4. Score my recipe (Green-Score)

**Enunciado.** Una herramienta de puntuación ambiental para recetas, con gamificación.

**Por qué está en medio.** El framework es asequible (un Next.js ya existente). Lo que cuesta es el criterio: una receta no es un producto con código de barras, las cantidades importan, y el Green-Score no es una media de letras.

El repo público más cercano es [`rate-my-recipe`](https://github.com/openfoodfacts/rate-my-recipe): Next.js, React, Redux, TypeScript y Yarn. Hoy apunta a Nutri-Score y Eco-Score de una receta propia. El reto pide dar el salto al Green-Score y añadir juego.

El Green-Score de un producto de catálogo sale de Agribalyse (análisis de ciclo de vida por categoría: agricultura, procesado, envase, transporte, distribución, consumo) ajustado con bonus y malus. Para una receta hay que partir ingredientes, asignar a cada uno una categoría o un alimento de referencia, ponderar por cantidad y explicar el resultado. La letra sola no enseña nada: la persona tiene que ver qué ingrediente domina el impacto.

La gamificación puede apoyarse en [`openfoodfacts-events`](https://github.com/openfoodfacts/openfoodfacts-events) (FastAPI, ranking e insignias, escrito en un hackathon anterior y pendiente de integrarse de verdad).

### Tecnologías

- Next.js, React, Redux, TypeScript, Yarn. El README arranca con `yarn dev` en el puerto 3000.
- API de OFF para ingredientes que sí tienen ficha (un yogur concreto, una marca de garbanzos).
- Datos de referencia para alimentos a granel (Agribalyse / tablas nutricionales), porque “200 g de lentejas” no tiene código de barras.
- Knowledge Panels, si reutilizas la explicación oficial del grado en lugar de reescribirla.
- Events, si hay puntos, retos semanales o insignias.

### Qué aprender, de menos a más

1. Calcular a mano, en una hoja, el Nutri-Score aproximado de una receta de tres ingredientes. Así se ve el problema de las cantidades.
2. Arrancar `rate-my-recipe` y seguir de dónde sale la puntuación actual.
3. Leer cómo OFF calcula el Green-Score de un producto ya categorizado: <https://world.openfoodfacts.org/green-score>.
4. Modelar la receta: lista de `{alimento, gramos}` y un alimento de referencia con impacto por kg.
5. Bonus y malus (ecológico, temporada, envase) como ajustes visibles, no como una caja negra.
6. Reglas de juego que empujen a completar datos, no solo a cazar la letra A.

### Qué necesitas

- Node y Yarn.
- Una decena de recetas reales, con gramos. Una demostración con un solo plato inventado deja fuera las unidades raras y los ingredientes sin ficha.
- Paciencia con las unidades. Mezclar por 100 g y por ración rompe la cuenta.

### Uso de IA

1. **Parsear el texto** “200 g de lentejas, una cebolla, aceite de oliva” a una lista estructurada. Las cantidades que el modelo no sepa se quedan en blanco para que las rellene la persona.
2. **Emparejar cada línea** con un tag de OFF o con un alimento de Agribalyse. El emparejamiento se muestra y se puede corregir.
3. **Explicar el resultado** con los números ya calculados: “el 70 % del impacto viene de la carne; pasar a legumbre cambia la letra”.
4. **Sugerir un cambio** concreto (otro aceite, menos queso, producto de temporada) y volver a calcular con el mismo código, no con otra frase del modelo.
5. **Retos de juego** generados a partir de la despensa que la persona ya tiene en OFF (“esta semana, una receta por debajo de tal umbral”).

La letra que se muestra la calcula tu código con datos citados. El modelo traduce y explica.

---

## 5. Open Food Facts Explorer y web components

**Enunciado.** Construir la interfaz web de referencia sobre el stack de web components.

**Por qué sube aquí.** Ya no es un minijuego: es el frontend que quiere sustituir a la web clásica, habla con media docena de servicios y tiene login de verdad. Se puede contribuir con un cambio pequeño, pero orientarse lleva más tiempo.

Hay dos repos, y el reto los junta:

- [`openfoodfacts-explorer`](https://github.com/openfoodfacts/openfoodfacts-explorer). SvelteKit, TypeScript, Vite, Tailwind, DaisyUI, pnpm, Sentry. Node reciente (el `package.json` marca el rango). Login con Keycloak mediante OAuth2 PKCE: el frontend recibe un permiso y la contraseña se queda en Keycloak. Llama a Product Opener, Folksonomy, Open Prices, Search-a-licious, Robotoff, Nutri-Patrol y Matomo.
- [`openfoodfacts-webcomponents`](https://github.com/openfoodfacts/openfoodfacts-webcomponents). Lit. Piezas reutilizables: Robotoff, ficha de producto, autocompletado, editor Folksonomy, y Knowledge Panels en curso. Las pueden usar Explorer, la web Perl, Hunger Games y Open Prices. El tráfico web es en más de un 80 % móvil: la UI se diseña primero para esa pantalla.

Slack: `#off-explorer` y `#javascript`. Hay mockups y una lista de paridad de funcionalidades en el README de Explorer. El diseño vive en [`openfoodfacts-design`](https://github.com/openfoodfacts/openfoodfacts-design).

### Qué aprender, de menos a más

1. Usar la web actual y anotar tres cosas que Explorer todavía no cubre (la lista de paridad del README es la referencia).
2. SvelteKit: rutas, `load`, stores, formulario. Si vienes de React, el salto es el compilador y el modelo de estado, no HTTP.
3. Arrancar Explorer con pnpm. Corepack, que viene con Node, activa la versión de pnpm que fija el repo.
4. Lit: un custom element, propiedades, eventos, y cómo se embebe dentro de Svelte sin reescribirlo.
5. Keycloak en local y el flujo de “editar producto”.
6. Extraer un trozo de UI a un web component para que la web clásica y Explorer lo compartan.

### Qué necesitas

- Node, pnpm, y un navegador con vista móvil (el ancho estrecho es el caso principal, no un extra).
- Cuenta de OFF y, para login real, la configuración de Keycloak del README.
- Figma, si el cambio es de interfaz: el equipo de diseño pide mirar los mockups antes de improvisar.

### Uso de IA

1. **Pedir un mapa del repo**: qué ruta pinta la ficha, qué módulo llama a qué API. Luego abres ese fichero.
2. **Traducir un bloque de la web Perl** a un web component, con la misma información y los mismos estados vacíos.
3. **Un asistente en la ficha**, al lado del producto, que solo puede citar campos que la página ya cargó. Enlaza al código de barras y a la edición cuando el campo está vacío.
4. **Accesibilidad.** Texto alternativo propuesto para una foto de envase, revisado por una persona, sin afirmar ingredientes que no estén en la ficha.
5. **Pintar la puntuación tal como llega.** Nutri-Score y Green-Score se muestran con lo que devuelve la API o Knowledge Panels. El cliente no los recalcula.

---

## 6. Search-a-licious

**Enunciado.** El buscador que viene: arreglos de backend, despliegue a producción, integración del frontend con Explorer, o una adaptación (port) a Flutter/Dart.

**Por qué está aquí.** Hay que entender Elasticsearch, un pipeline de eventos y Docker. Las cuatro tareas del enunciado no cuestan lo mismo: un bug acotado de la API es lo más abordable; la adaptación a móvil es otro proyecto.

[`search-a-licious`](https://github.com/openfoodfacts/search-a-licious) (<https://search.openfoodfacts.org>, documentación en <https://openfoodfacts.github.io/search-a-licious/>) convierte una colección grande en algo consultable: texto, facetas (filtros que además cuentan cuántos resultados hay detrás: marca, Nutri-Score, país), sugerencias mientras escribes, ordenaciones y gráficos. Nació para OFF y se puede reutilizar con otro dataset definido en un YAML.

### Tecnologías

- API en Python 3.11 (el `pyproject.toml` pide `>=3.11,<3.12`), FastAPI, Pydantic, Typer. El parser de consultas usa [luqum](https://github.com/jurismarches/luqum).
- Elasticsearch (el cliente del repo es el DSL de la serie 8). OpenSearch es un objetivo deseado. El motor por defecto es Elasticsearch.
- Redis para el stream de actualizaciones. La carga inicial es un JSONL; luego el índice se mantiene con eventos.
- Frontend en Lit (carpeta `frontend`) y gráficos con Vega.
- Docker Compose para levantar API, índice y Redis.
- La integración con Explorer es un cliente más de `SEARCH_API_HOST`. La adaptación a Flutter consumiría la misma API desde `openfoodfacts-dart` o desde smooth-app.

### Qué aprender, de menos a más

1. Usar el buscador público: una query, dos facetas, un gráfico.
2. Leer el YAML de configuración de OFF y ver qué campos son texto, keyword o numérico.
3. Levantar Compose y hacer una búsqueda contra tu instancia.
4. Seguir un documento desde el JSONL hasta el mapping de Elasticsearch.
5. El stream: qué pasa cuando Product Opener edita un producto y el índice tiene que ponerse al día.
6. Publicar el servicio (comprobaciones de salud, Sentry, volúmenes del índice) o embeber los web components de búsqueda en Explorer.
7. La adaptación a Dart: cliente HTTP, los mismos filtros, y una UI que quepa en la app. Esto ya pide el nivel 7.

### Qué necesitas

- Docker con RAM de sobra: Elasticsearch se queda corto en máquinas justas. 8 GB de RAM es un mínimo razonable para indexar un subconjunto. El dump completo pide bastante más.
- Python 3.11 (pyenv o asdf evitan pelearte con el 3.12 del sistema).
- Un dump pequeño para desarrollar. Indexar todo OFF en el portátil no es el primer paso.

### Uso de IA

1. **Explicar una query** que ya ejecutaste: qué cláusula filtró qué.
2. **Lenguaje natural → consulta.** “bebida vegetal sin azúcar, vendida en España” pasa por el esquema YAML. Solo se emiten campos y tags que existen. luqum ya entiende la sintaxis: el modelo rellena esa sintaxis, no inventa un dialecto.
3. **Cero resultados.** Proponer qué filtro quitar, después de mirar las facetas reales.
4. **Un gráfico Vega** a partir de una agregación que la API ya devuelve.
5. **Dejar el orden de los resultados en manos del índice.** El ranking lo calcula Elasticsearch con la configuración del YAML, de forma que se pueda reproducir. Un reordenado “a ojo del modelo” se sale de ese contrato.

---

## 7. App móvil oficial (smooth-app)

**Enunciado.** Rediseñar la arquitectura del modo sin red, la sincronización en los dos sentidos, la integración con Open Prices y el onboarding (la primera configuración al abrir la app) personalizado.

**Por qué es claramente más difícil.** Es la app que la gente tiene instalada. Hay cadena de herramientas de Android e iOS, un plugin Dart que otros clientes reutilizan, estado local, y el problema de fondo: editar sin cobertura y reconciliar después con el servidor y con otras personas.

[`smooth-app`](https://github.com/openfoodfacts/smooth-app) es Flutter. El README fija la versión con FVM (en el momento de escribir esta guía, Flutter 3.44.x: manda el repo, no esta página). El acceso a la API va en [`openfoodfacts-dart`](https://github.com/openfoodfacts/openfoodfacts-dart), fuera de la UI: cualquier llamada nueva se añade primero al plugin. También está `openfoodfacts_flutter_lints`. Las versiones de escritorio (Linux, macOS y Windows) sirven para desarrollar.

El modo sin red importa porque en muchos supermercados no hay red usable. Hoy existe una idea de dump reducido por país e idioma (nombre, marca, puntuaciones) y un issue abierto para generarlo bajo demanda sin tumbar el servidor. La sincronización en los dos sentidos añade conflictos: tú corregiste un ingrediente en el pasillo y alguien publicó otra revisión del mismo producto.

Open Prices en la app significa escanear un precio (producto, tienda, comprobante) y que sobreviva sin conexión hasta que haya red. El onboarding personalizado recoge alergias, dieta y lo que la persona quiere ver primero: son preferencias locales que cambian filtros y paneles.

Slack: `#mobile`.

### Qué aprender, de menos a más

1. Instalar la app pública y usarla en modo avión: qué se puede escanear, qué ficha aparece, qué pasa al editar.
2. Flutter: widgets, `async`, navegación, el punto de entrada distinto de Android (`lib/entrypoints/android/main_google_play.dart`) e iOS (`lib/entrypoints/ios/main_ios.dart`). El paquete de la app está en `packages/smooth_app`.
3. El plugin Dart: leer un producto, mandar una foto, pedir una pregunta de Robotoff.
4. Estado local: qué se cachea, qué se encola, qué identifica a una edición pendiente.
5. Sincronización: cola duradera, reintentos, identidad de la operación (para no aplicar dos veces el mismo edit) y política de conflicto cuando la revisión del servidor ha cambiado.
6. Open Prices y onboarding, como dos clientes de esa misma cola, no como pantallas sueltas.

### Qué necesitas

- Flutter vía FVM, Android Studio o Xcode según la plataforma, y un dispositivo o emulador.
- Cuenta de OFF.
- Para precios: entender que el dato es ODbL y que va al servicio de Open Prices, no solo a la ficha del producto.
- Un sitio sin Wi-Fi para probar de verdad. El emulador con la red cortada se acerca a un supermercado con mala cobertura.

### Uso de IA

1. **Ayudarte a leer el código** de una pantalla y del plugin. Los cambios de sincronización se diseñan en papel antes de pedirle a un modelo que reescriba el almacenamiento.
2. **Onboarding.** A partir de cuatro preguntas (alergias, vegetariano, sal, origen) construir la configuración que la app ya sabe aplicar. La personalización es un mapeo a filtros existentes.
3. **Pre-rellenar una edición** con una foto, en cola, marcada como borrador hasta que la persona confirme. El envío usa el mismo camino que una edición manual. El detalle de extracción visual está en el nivel 11; aquí solo se consume.
4. **Explicar un conflicto de sincronización** con las dos revisiones ya descargadas. La persona elige. Ingredientes y alérgenos en conflicto se muestran, y los fusiona quien tiene el producto delante.
5. **Modelos en el dispositivo** solo para tareas estrechas (detectar el código de barras, enderezar una foto). La ficha nutricional de millones de productos no cabe en un modelo local: sin red se consulta el dump, no un chatbot.

---

## 8. Open Prices

**Enunciado.** Recolección colaborativa de precios e inflación: detección de códigos de barras y precios, anonimización de tickets, detección de anomalías, y una app móvil dedicada.

**Por qué va por delante de la app oficial en dificultad.** Además de producto y comunidad, hay visión por computador, un proveedor de modelo externo y datos personales metidos en un ticket de supermercado (nombre, tarjeta, número de fidelización).

[`open-prices`](https://github.com/openfoodfacts/open-prices) (<https://prices.openfoodfacts.org>) guarda precios con lugar y fecha. Hace falta cuenta para contribuir; explorar los datos se puede sin ella. El backend es Django sobre Python 3.11 y PostgreSQL. Los comprobantes (una foto de estantería o un ticket) se procesan en segundo plano con Django Q2.

Ese procesamiento, documentado en el `CONTRIBUTING.md` del repo, hace hoy esto:

- Triton (modelos que viven en el repo de Robotoff) clasifica el tipo de comprobante y detecta etiquetas de precio.
- Gemini extrae el contenido del ticket y el de cada etiqueta.

El frontend web es [`open-prices-frontend`](https://github.com/openfoodfacts/open-prices-frontend): Vue 3 y Yarn. La app dedicada es el otro extremo del enunciado: captura rápida en la tienda, cola offline (mira el nivel 7) y subida del comprobante ya recortado.

Slack: `#prices`, el canal que indica el README del repo. Licencia del dataset: ODbL.

### Qué aprender, de menos a más

1. Contribuir un precio a mano en la web y mirar el objeto que devuelve la API (`/api/docs`).
2. Modelo de datos: precio, producto, lugar, comprobante, moneda, fecha. La anomalía se compara contra ese historial.
3. Django y el panel o los comandos de gestión. `uv` es el gestor que usa el repo.
4. Un recorrido de punta a punta en local: Docker, Postgres y, si vas a tocar modelos, Triton más una clave de Gemini.
5. Detección: de una foto de estantería a cajas “código de barras + precio”.
6. Anonimización: localizar datos personales en el ticket y no almacenar el original.
7. Anomalías: un precio muy lejos de la mediana de ese producto en esa zona y esas fechas.
8. La app dedicada, cuando el flujo web ya sea claro.

### Qué necesitas

- Docker, Python 3.11, Node y Yarn para el frontend.
- Para la parte de modelos: el repo de Robotoff, `make dl-object-detection-models`, Triton, y una clave de API de Gemini. Sin esa clave puedes trabajar en API, UI y anomalías estadísticas.
- Fotos tuyas de tickets, y el hábito de tapar con el dedo el nombre y la tarjeta antes de subirlas a cualquier sitio, incluido un modelo.

### Uso de IA

1. **Clasificar el comprobante** (ticket, etiqueta, estantería) con el modelo que ya sirve Triton.
2. **Leer etiquetas** con el detector y un esquema JSON (producto, precio, moneda). Lo que no cuadre con una caja detectada no se publica.
3. **Leer un ticket** con un modelo de visión, a un JSON de líneas `{descripción, precio}`. Es el camino que ya usa Gemini: el trabajo fino es el esquema, los ejemplos y los fallos (descuentos, 2x1, pesos).
4. **Anonimizar antes de salir del teléfono o del servidor.** Detectar nombre, dirección, tarjeta y código de fidelización, enmascararlos, y guardar solo el ticket limpio más las líneas. Un modelo remoto no es el sitio donde mandar el ticket crudo “para ver qué pasa”.
5. **Anomalías.** Reglas numéricas primero (precio por kilo imposible, salto de un orden de magnitud respecto a la mediana). Un modelo puede ordenar la cola de revisión; no borra precios solo.
6. **Enlazar una línea del ticket** con un producto de OFF cuando hay código de barras, y dejarla como texto cuando no lo hay.

---

## 9. Data imports

**Enunciado.** Estructurar imports masivos de datos externos (más de medio millón de productos) y una infraestructura modular para que el siguiente import no sea un script nuevo.

**Por qué es de los más duros.** El código que escribe en la base es Perl, el modelo de producto es profundo (nutrientes, taxonomías, idiomas, fotos, revisiones) y un error se multiplica por el tamaño del fichero. No es un buen primer contacto con el proyecto, y es un muy buen reto si ya te sientes cómodo con la API.

Hoy el camino de productores está en Product Opener, módulo `ProductOpener::Import`: el fabricante manda una tabla, se convierte al CSV de Open Food Facts y `import_csv_file` crea o actualiza productos e imágenes, atribuido a un usuario y a una organización. El mismo módulo empuja datos desde la plataforma de productores hacia la base pública. La guía para quien manda el fichero está en <https://world.openfoodfacts.org/producers>: XLS o CSV separado por tabuladores, UTF-8, primera fila con nombres de campo, columnas obligatorias y opcionales.

El repo [`data-imports`](https://github.com/openfoodfacts/data-imports) ordena el material externo (productores, apps, certificadoras) con un README por fuente, licencia y datos en bruto. El subject pide ir más allá de “una carpeta por productor”: pipelines repetibles, validación, y reanudar un import a la mitad.

### Tecnologías

- Perl, el módulo `ProductOpener::Import`, MongoDB y el formato de producto de Product Opener.
- CSV / XLS de entrada, CSV canónico de OFF de salida.
- Taxonomías (categorías, etiquetas, países, nutrientes) contra las que hay que mapear columnas ajenas.
- Imágenes asociadas a cada código de barras (frontal, ingredientes, nutrición, y otras vistas con nombre corto).
- Python cabe en la etapa de limpieza y de validación. La escritura en la base sigue el camino de Product Opener, que conserva revisiones, usuarios y eventos.

### Qué aprender, de menos a más

1. El CSV canónico: qué columnas existen y cuáles son obligatorias. Importar 20 filas a staging y mirar el producto resultante en la API.
2. Diferencia entre crear, completar un campo vacío y pisar un valor que puso un voluntario. Esa política es el diseño del import, no un detalle.
3. Taxonomías: “Boisson gazeuse” tiene que acabar en un tag, no en un texto suelto.
4. Nutrientes: unidad, “por 100 g” frente a “por ración”, y valores que no pueden ser negativos ni sumar más que 100 g.
5. Idempotencia: lanzar el mismo fichero dos veces no duplica productos ni fotos.
6. Modularizar: conector de la fuente → mapeo a CSV canónico → informe de errores → import → informe de lo que cambió. Cada fuente aporta su mapeo dentro de ese mismo recorrido.
7. Volumen: medio millón de filas se procesa por lotes, con un punto de control para reanudar y un informe que una persona pueda leer.

### Qué necesitas

- El entorno de desarrollo de [`openfoodfacts-server`](https://github.com/openfoodfacts/openfoodfacts-server) (Docker del proyecto). Resérvale tiempo: es el setup más pesado de los repos en Perl.
- Un extracto real, aunque sean 100 filas representativas, con su licencia. Las columnas fusionadas, los idiomas mezclados y los códigos de barras cortos salen de un fichero de verdad.
- Staging. El primer sitio donde corre un import nuevo es `.net`.
- Alguien del equipo de Product Opener en Slack (`#productopener`) antes de diseñar el formato de salida.

### Uso de IA

1. **Describir un fichero sucio:** tipos de columna, porcentajes de vacío, valores raros. Eso es un perfilado, y se contrasta con pandas o con un script, no con la prosa del modelo.
2. **Proponer el mapeo** de columnas del productor a columnas OFF, con un ejemplo por fila. Una persona lo aprueba, y solo entonces se lanza sobre los cientos de miles de filas.
3. **Normalizar valores** (idiomas, “sí/oui/1”, comas decimales) con reglas explícitas. El modelo sugiere la regla; el código la aplica igual a todas las filas.
4. **Asignar taxonomía** por similitud con la lista oficial de tags, mostrando el segundo candidato cuando la confianza sea baja.
5. **Un informe de rechazos** agrupado (“12 400 filas sin código de barras”, “3 200 con azúcar > 100 g”). Ese informe es el entregable que hace el import mantenible.

---

## 10. Robotoff

**Enunciado.** Multiplicar las contribuciones con machine learning.

**Por qué está tan arriba.** Es el sistema nervioso de las predicciones. Tiene varios procesos, colas, dos bases de datos y modelos servidos aparte. Un cambio en el umbral de auto-aplicación modifica productos en producción sin que nadie pulse un botón.

Robotoff ([`robotoff`](https://github.com/openfoodfacts/robotoff), documentación en <https://openfoodfacts.github.io/robotoff/>) predice datos a partir de las fotos. Arquitectura real:

- API pública.
- Scheduler (tareas periódicas, descargar el dataset, auto-aplicar insights).
- Workers sobre Redis y [rq](https://python-rq.org/). Hay colas de alta prioridad (una por producto, para no procesar el mismo código de barras en paralelo), una cola baja y una cola de modelos.
- Update listener, enganchado al stream de Redis de Product Opener.
- PostgreSQL para predicciones e insights.
- Elasticsearch para buscar logos parecidos (vecinos más cercanos).
- Triton para los detectores (Nutri-Score, tabla nutricional, logos).
- En producción lee MongoDB de Product Opener para consultar la última versión del producto sin pasar por la API.

Cuando alguien sube una foto, Google Cloud Vision hace el OCR. Robotoff recibe imagen y JSON de OCR. A partir del texto salen predicciones por reglas (marcas, etiquetas, códigos de envasador, peso, fechas). A partir de la imagen, un detector de logos y un embedding CLIP (`clip-vit-base-patch32`) más k-NN proponen marca o sello; otro modelo lee la letra del Nutri-Score. Esas predicciones se convierten en insights. Hunger Games, la app y la web los confirman. Con confianza alta, algunos se aplican solos a los diez minutos. El repo [`openfoodfacts-ai`](https://github.com/openfoodfacts/openfoodfacts-ai) coordina el trabajo de investigación.

Slack: `#robotoff`. Reunión conjunta con Hunger Games algunos martes; el horario está en el README.

### Qué aprender, de menos a más

1. La diferencia entre predicción (lo que dijo el modelo) e insight (lo que se propone a un humano o se auto-aplica). Está en la documentación, y es la pieza que más se confunde.
2. Llamar a la API pública: preguntas aleatorias, insights de un código de barras, anotar uno.
3. Levantar Robotoff con Docker y mirar una cola de rq.
4. Seguir el camino de una imagen: evento → OCR → predicción → insight → anotación → `POST` a Product Opener.
5. Triton en local y uno de los detectores ya publicados (`make dl-object-detection-models` desde el flujo que también usa Open Prices).
6. Evaluación: precisión (de lo que el modelo marca, cuánto es correcto) y recall (de lo correcto, cuánto encuentra) sobre un conjunto etiquetado, antes de mover el umbral de auto-aplicación.
7. Entrenar o ajustar un modelo nuevo entra en el terreno de `openfoodfacts-ai` y de los datasets del proyecto. No es el primer parche.

### Qué necesitas

- Docker, Python, y disco para imágenes y pesos.
- Para OCR real, credenciales de Google Cloud Vision. Sin ellas puedes trabajar en la lógica de insights, en tests y en modelos ya exportados a Triton.
- GPU ayuda para entrenar. Para servir un detector pequeño en local a veces basta la CPU, y va lento.
- Criterio de producto: un insight automático sobre alérgenos se trata distinto de un insight sobre el peso del paquete.

### Uso de IA

1. **Leer insights que ya existen** y construir un mejor cliente de anotación (eso reutiliza el nivel 2). El modelo ya corrió.
2. **Reglas sobre el OCR** para un patrón que el sistema todavía no cubre (un sello nuevo, un formato de fecha). Una regla se testea con ejemplos positivos y negativos.
3. **Clasificar logos** con el índice CLIP + k-NN que ya está, añadiendo ejemplos vía Hunger Games. Más anotaciones suelen ganar a un modelo nuevo.
4. **Extracción con un modelo de visión** a JSON, usando el esquema de datasets que Robotoff ya documenta para LVLMs (imagen, instrucción, JSON de salida). El dataset de ejemplo es el de etiquetas de precio; el mismo esquema sirve para ingredientes o tabla nutricional.
5. **Decidir qué se auto-aplica.** Umbral por tipo de campo, medido en un conjunto etiquetado, con una muestra humana después del despliegue. “El modelo nuevo es mejor” se demuestra con ese conjunto, no con tres fotos de la cocina.

---

## 11. Rapid Product Acquisition

**Enunciado.** Extracción con un modelo de visión y lenguaje (LVLM) para dar de alta un producto a partir de una foto en menos de un minuto.

**Por qué cierra la guía.** Junta casi todo lo anterior: fotos, OCR, taxonomías, escritura en Product Opener, umbrales de Robotoff, y una persona confirmando en el móvil. El límite de un minuto es un presupuesto de producto (hacer las fotos, extraer, revisar, publicar), no solo un tiempo de inferencia.

Hoy, añadir un producto es una secuencia de fotos (código de barras, frontal, ingredientes, tabla nutricional) y bastante tecleo. Robotoff ya rellena parte después, en diferido. Este reto adelanta ese rellenado al momento de la captura: la persona fotografía, ve un formulario casi listo, corrige y publica antes de guardar el paquete.

### Tecnologías

- Captura: la app Flutter (nivel 7) o un prototipo web. Hace falta el código de barras sí o sí, porque es la clave del producto.
- Extracción: un LVLM con instrucción fija y salida JSON validada contra un esquema. Robotoff ya describe ese formato de dataset (Parquet en Hugging Face: `image_id`, imagen, `output` JSON, metadatos, más un fichero de instrucción y esquema). Ver la documentación de Robotoff, “LLM image extraction dataset”.
- Validación: taxonomías, unidades, rangos de nutrientes, alérgenos declarados frente a los de la lista de ingredientes.
- Escritura: API de Product Opener (imágenes seleccionadas por tipo e idioma, luego campos), atribuida al usuario que está dando de alta el producto.
- Evaluación: un conjunto de productos ya completos. Se tapan los campos, se extraen de las fotos y se comparan.

### Qué aprender, de menos a más

1. Dar de alta tú un producto con la app, cronómetro en mano. Anota en qué paso se van los segundos.
2. El esquema mínimo de alta: código, nombre, marca, cantidad, ingredientes, nutrientes por 100 g, alérgenos, categoría. Todo lo demás puede llegar después.
3. El contrato del LVLM: una instrucción, un JSON Schema, y una respuesta que se rechaza si no valida.
4. Fallos típicos de una foto de envase: texto curvado en una lata, brillo, varios idiomas, tabla nutricional en “por ración” y no por 100 g.
5. La UI de confirmación: cada campo muestra el recorte de donde salió, la confianza y un estado vacío honesto.
6. El presupuesto de un minuto: cuánto tarda la foto, cuánto la inferencia, cuánto la revisión. Si el modelo tarda 40 segundos, la UI tiene que estar lista para editar lo que ya llegó.
7. Publicar en staging y volver a leer el producto por la API. El alta no está hecha hasta que ese `GET` devuelve lo que la persona confirmó.

### Qué necesitas

- Acceso a un LVLM con visión (API o un modelo que puedas servir). Presupuesta el coste por imagen.
- Un conjunto de 30–50 productos que ya estén bien en OFF, con sus fotos públicas, para medir sin etiquetar desde cero.
- El SDK de escritura (Dart si el prototipo es la app, Python o JS si es un backend).
- Staging y una cuenta.
- Robotoff cerca, aunque sea solo como referencia de esquema y de insights: no construyas un segundo sistema de predicciones si puedes devolver la extracción como propuestas.

### Uso de IA

1. **Una sola foto, un solo campo.** La tabla nutricional a JSON `{nutriente, valor, unidad}`. Se valida y se compara con el producto ya existente.
2. **Ingredientes en el idioma de la foto**, más los alérgenos que el propio texto declara. El modelo no “deduce” alérgenos que la lista no menciona.
3. **Varias fotos en un formulario.** Frontal → nombre y marca. Ingredientes → lista. Nutrición → tabla. Código de barras → `code`, leído por un detector clásico, que en esto sigue siendo más fiable que un LVLM.
4. **Confianza por campo.** Por encima del umbral, el campo aparece relleno y editable. Por debajo, aparece vacío con el recorte al lado. Nada se publica sin el OK de la persona.
5. **Cerrar el minuto.** Medir las 30 altas de prueba: tiempo mediano, campos correctos, campos que la persona tuvo que borrar. Ese número es el resultado del hackathon. Una demostración con un solo producto favorecido se queda corta.

---

## Cómo elegir durante la semana

| Si llegas con… | Empieza en | Y evita, el primer día |
| --- | --- | --- |
| Ganas de probar la API y un cliente MCP | 1 | Escribir en producción |
| React y ganas de una UI que se pueda enseñar | 2 o 3 | Reescribir Robotoff |
| Next.js y curiosidad por el impacto de lo que cocinas | 4 | Reimplementar Agribalyse entero |
| Svelte, Lit o muchas ganas de frontend web | 5 | La web Perl, salvo un bug muy acotado |
| Python, Docker y Elasticsearch | 6 | Indexar el dump completo en el portátil |
| Flutter | 7, o la adaptación a móvil del reto 6 | Un rediseño de la sincronización sin una cola y una política de conflicto escritas |
| Django o visión, y cuidado con datos personales | 8 | Mandar tickets sin anonimizar a un modelo externo |
| Perl o pipelines de datos | 9 | Un import grande contra `.org` |
| ML y ganas de evaluar | 10 y, si el formulario te importa, 11 | Bajar el umbral de auto-aplicación “para que se vea” |

Un recorte que se puede enseñar el último día pesa más que un esqueleto de los once repos. Los mentores de OFF están en el sitio: el primer mensaje útil en Slack es el reto, lo que ya has leído y el primer error concreto que te ha parado.

## Referencias rápidas

- Subject del track: [`subject.md`](subject.md)
- API v3: <https://openfoodfacts.github.io/openfoodfacts-server/api/>
- Product Opener para desarrolladores: <https://openfoodfacts.github.io/openfoodfacts-server/>
- Robotoff: <https://openfoodfacts.github.io/robotoff/>
- Search-a-licious: <https://openfoodfacts.github.io/search-a-licious/>
- Green-Score: <https://world.openfoodfacts.org/green-score>
- Productores e imports: <https://world.openfoodfacts.org/producers>
- Wiki: <https://wiki.openfoodfacts.org>
