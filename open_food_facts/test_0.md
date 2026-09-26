# Laboratorio: ficha de producto en la terminal

Una demo pequeña del primer peldaño del track: leer Open Food Facts y comparar dos productos con los datos que devuelve la API, sin inventar nutrientes ni alérgenos.

Al terminar tienes un programa en Python que, a partir de un código de barras, imprime nombre, marca, Nutri-Score, NOVA, Green-Score, alérgenos, sal y azúcares. Si le pasas dos códigos, dice cuál trae menos sal y menos azúcares por 100 g. La cuenta sale de los números de la API. El contexto del track está en [`track.md`](track.md).

Tiempo orientativo: 45–60 minutos. Hace falta internet. No hace falta cuenta, clave, Docker ni clonar los repos de Open Food Facts. Esta práctica solo lee.

## Qué vas a usar

| Pieza | Para qué | ¿Hay que instalarla? |
| --- | --- | --- |
| Un sistema con terminal | Escribir los comandos | Ya lo tienes: Linux, macOS o Windows con WSL |
| `python3` 3.9 o posterior | El programa de la ficha | A menudo ya viene instalado. Abajo se comprueba |
| `curl` | Ver la respuesta cruda de la API antes de programar | Igual: se comprueba y, si falta, se instala |
| Un editor de texto | Crear el fichero `ficha.py` | `nano` vale. También gedit, Kate, Vim o VS Code |
| La librería estándar de Python | Hablar con la API (`urllib`, `json`, `argparse`) | Viene dentro de Python. No hay `pip install` |

Módulos de la librería estándar que usa el programa, y por qué:

- `urllib.request` hace la petición HTTP.
- `urllib.error` distingue “no existe” de “no hay red”.
- `urllib.parse` coloca el código de barras en la URL.
- `json` convierte el texto de la respuesta en datos.
- `argparse` lee los códigos que escribes al lanzar el programa.
- `sys` termina con un mensaje claro cuando la llamada no se puede completar.

## 1. Abre una terminal

En Linux: la aplicación Terminal, o `Ctrl+Alt+T` en Ubuntu y derivadas.

En macOS: Terminal, dentro de Aplicaciones → Utilidades.

En Windows: instala [WSL](https://learn.microsoft.com/windows/wsl/install) con Ubuntu y abre “Ubuntu” desde el menú Inicio. El resto de la guía son comandos de Linux. PowerShell también puede ejecutar `python` y `curl` en Windows 10/11 recientes, pero los ejemplos de instalación de abajo están escritos para la terminal de Linux y de macOS.

Comprueba que la terminal responde:

```bash
pwd
```

`pwd` imprime la carpeta en la que estás. Algo como `/home/tu-usuario`.

## 2. Comprueba Python y curl

```bash
python3 --version
curl --version
```

Una salida válida de Python empieza por `Python 3.` y el número siguiente es 9 o mayor (`Python 3.9.x`, `3.10`, `3.11`, `3.12`…). Este laboratorio se ha ejecutado con Python 3.12.

`curl --version` tiene que imprimir `curl` y un número de versión. Si el comando no existe, la shell responde `command not found` o `no se ha encontrado la orden`.

Si ambas órdenes responden, salta al paso 4.

## 3. Instala solo lo que falte

### Linux con apt (Ubuntu, Debian, Linux Mint)

```bash
sudo apt update
sudo apt install python3 curl
```

`sudo` pide tu contraseña de usuario. `apt update` refresca la lista de paquetes. `apt install` instala los que falten.

### Linux con dnf (Fedora)

```bash
sudo dnf install python3 curl
```

### Linux con pacman (Arch, Manjaro)

```bash
sudo pacman -S python curl
```

En Arch el ejecutable suele llamarse `python` además de `python3`. Si `python3 --version` no existe y `python --version` sí, usa `python` en los comandos de más abajo.

### macOS

Si `python3` falta, instálalo desde [python.org](https://www.python.org/downloads/) o, si ya usas Homebrew:

```bash
brew install python curl
```

`curl` suele estar ya en el sistema.

### Windows con WSL Ubuntu

Dentro de la terminal Ubuntu:

```bash
sudo apt update
sudo apt install python3 curl
```

Vuelve a lanzar las dos comprobaciones del paso 2 antes de seguir.

## 4. Crea una carpeta de trabajo

El laboratorio vive fuera de este repositorio, para no mezclar el programa de prueba con la guía.

```bash
cd
mkdir -p laboratorio-off
cd laboratorio-off
pwd
```

`cd` sin argumentos va a tu carpeta personal. `mkdir -p` crea `laboratorio-off` si no existía. El último `pwd` debe terminar en `/laboratorio-off`.

## 5. Primera llamada a la API, con curl

Open Food Facts publica una API HTTP. Una lectura es una URL. Este código de barras es el de un bote de Nutella muy usado en la documentación del proyecto:

```bash
curl -sS \
  -H "User-Agent: DigitalCommonsWeekLab - laboratorio (contacto: tu-email@ejemplo.org)" \
  "https://world.openfoodfacts.org/api/v3/product/3017620422003.json?fields=code,product_name,brands,nutriscore_grade,nova_group,ecoscore_grade,allergens_tags"
```

Qué hace cada trozo:

- `curl` pide la URL y escribe la respuesta en la terminal.
- `-sS` oculta la barra de progreso y deja visibles los errores.
- `-H` añade una cabecera. Open Food Facts pide un `User-Agent` que identifique al cliente y dé un contacto. Cambia `tu-email@ejemplo.org` por el tuyo, o por tu usuario de GitHub.
- La URL es `GET` (curl lo usa por defecto): no modifica la base.
- `3017620422003` es el código de barras.
- `fields` pide solo esos campos, para que la respuesta quepa en la pantalla. Sin `fields` el JSON trae cientos de claves.

La respuesta es JSON. Con los campos de arriba se parece a esto (el texto puede variar: la base la edita la comunidad):

```json
{
  "code": "3017620422003",
  "product": {
    "allergens_tags": ["en:milk", "en:nuts", "en:soybeans"],
    "brands": "Nutella, Ferrero",
    "code": "3017620422003",
    "ecoscore_grade": "unknown",
    "nova_group": 4,
    "nutriscore_grade": "e",
    "product_name": "Nutella"
  },
  "status": "success"
}
```

Cómo leerlo:

- `status: success` significa que el servidor ha encontrado el producto.
- `product` es la ficha.
- `code` es el código de barras.
- `product_name` y `brands` son textos.
- `nutriscore_grade` es una letra de la `a` a la `e` cuando el dato existe.
- `nova_group` es un entero del 1 al 4.
- `ecoscore_grade` es el Green-Score. Puede ser una letra, `unknown` (no se ha podido calcular) o `not-applicable` (el cálculo no se aplica a ese producto). Las tres son respuestas de la API: se muestran tal cual.
- `allergens_tags` es una lista de identificadores de la taxonomía. `en:milk` es el tag inglés de la leche, no una traducción hecha por nosotros. Si la lista no viene, la ficha no declara alérgenos: eso no demuestra que el producto esté libre de ellos.

La sal y el azúcar no salen en este `curl` a propósito. Están dentro de `nutriments`, un objeto largo. El programa de abajo pide ese objeto y se queda con `salt_100g` y `sugars_100g`.

## 6. Crea el programa

Abre un fichero nuevo en la carpeta del laboratorio:

```bash
nano ficha.py
```

En nano: pegas el programa, guardas con `Ctrl+O` e Intro, y sales con `Ctrl+X`. Si usas otro editor, crea el mismo fichero `ficha.py` dentro de `laboratorio-off` con el mismo contenido.

Antes de pegar, cambia en `USER_AGENT` el texto `cambia-este-texto` por tu email o tu usuario. Es la única línea que tienes que tocar.

```python
#!/usr/bin/env python3
"""Ficha de un producto de Open Food Facts, leída de la API pública."""

import argparse
import json
import sys
import urllib.error
import urllib.parse
import urllib.request

API = "https://world.openfoodfacts.org/api/v3/product/{code}.json"
FIELDS = ",".join(
    [
        "code",
        "product_name",
        "brands",
        "nutriscore_grade",
        "nova_group",
        "ecoscore_grade",
        "allergens_tags",
        "nutriments",
    ]
)
USER_AGENT = "DigitalCommonsWeekLab - laboratorio ficha (contacto: cambia-este-texto)"


def fetch(code: str) -> dict:
    """Pide la ficha de un código de barras y devuelve el objeto product."""
    url = API.format(code=urllib.parse.quote(code)) + "?fields=" + FIELDS
    request = urllib.request.Request(url, headers={"User-Agent": USER_AGENT})
    try:
        with urllib.request.urlopen(request, timeout=30) as response:
            payload = json.load(response)
    except urllib.error.HTTPError as exc:
        if exc.code == 404:
            sys.exit(f"No hay ningún producto con el código {code} en Open Food Facts.")
        sys.exit(f"La API ha respondido {exc.code} {exc.reason}.")
    except urllib.error.URLError as exc:
        sys.exit(f"No se ha podido conectar: {exc.reason}")

    product = payload.get("product")
    if payload.get("status") != "success" or not product:
        sys.exit(f"La API no ha devuelto un producto para {code}.")
    return product


def show_value(value) -> str:
    if value is None or value == "":
        return "(sin dato en Open Food Facts)"
    return str(value)


def allergens(product: dict) -> str:
    tags = product.get("allergens_tags") or []
    if not tags:
        return "(la ficha no trae allergens_tags)"
    return ", ".join(tags)


def per_100g(product: dict, nutrient: str) -> str:
    nutriments = product.get("nutriments") or {}
    value = nutriments.get(f"{nutrient}_100g")
    if not isinstance(value, (int, float)):
        return "(sin dato en Open Food Facts)"
    return f"{value} g"


def show(product: dict) -> None:
    code = product.get("code", "?")
    print(f"Código:         {code}")
    print(f"Nombre:         {show_value(product.get('product_name'))}")
    print(f"Marca:          {show_value(product.get('brands'))}")
    print(f"Nutri-Score:    {show_value(product.get('nutriscore_grade'))}")
    print(f"NOVA:           {show_value(product.get('nova_group'))}")
    print(f"Green-Score:    {show_value(product.get('ecoscore_grade'))}")
    print(f"Alérgenos:      {allergens(product)}")
    print(f"Sal / 100 g:    {per_100g(product, 'salt')}")
    print(f"Azúcares / 100 g: {per_100g(product, 'sugars')}")
    print(f"Ficha:          https://world.openfoodfacts.org/product/{code}")


def nutrient_value(product: dict, nutrient: str):
    nutriments = product.get("nutriments") or {}
    value = nutriments.get(f"{nutrient}_100g")
    if isinstance(value, (int, float)):
        return float(value)
    return None


def compare(left: dict, right: dict, nutrient: str, label: str) -> None:
    left_value = nutrient_value(left, nutrient)
    right_value = nutrient_value(right, nutrient)
    left_name = show_value(left.get("product_name"))
    right_name = show_value(right.get("product_name"))
    print(f"\n{label} por 100 g")
    print(f"  {left_name}: {left_value if left_value is not None else '(sin dato)'} g")
    print(f"  {right_name}: {right_value if right_value is not None else '(sin dato)'} g")
    if left_value is None or right_value is None:
        print("  Sin comparación: falta el dato en una de las dos fichas.")
        return
    if left_value < right_value:
        winner = f"{left_name} ({left.get('code')})"
    elif right_value < left_value:
        winner = f"{right_name} ({right.get('code')})"
    else:
        print("  Empate.")
        return
    print(f"  Menos {label.lower()}: {winner}.")


def main() -> None:
    parser = argparse.ArgumentParser(
        description="Muestra la ficha de Open Food Facts de uno o dos códigos de barras."
    )
    parser.add_argument("codes", nargs="+", help="Uno o dos códigos de barras")
    args = parser.parse_args()
    if len(args.codes) > 2:
        sys.exit("Pasa un código, o dos para compararlos.")

    products = [fetch(code) for code in args.codes]
    for product in products:
        print()
        show(product)
    if len(products) == 2:
        compare(products[0], products[1], "salt", "Sal")
        compare(products[0], products[1], "sugars", "Azúcares")


if __name__ == "__main__":
    main()
```

El programa, en orden:

1. Construye la URL del producto y pide solo los campos de la ficha.
2. Envía el `User-Agent`.
3. Si el servidor responde 404, el código no está en la base.
4. Si `status` es `success`, se queda con `product`.
5. Imprime cada campo. Un valor ausente se escribe como “(sin dato en Open Food Facts)”.
6. Con dos códigos, compara `salt_100g` y `sugars_100g`. Si a uno le falta el número, no elige ganador.

## 7. Lanza la ficha de un producto

Desde `laboratorio-off`:

```bash
python3 ficha.py 3017620422003
```

Salida de referencia, tomada en septiembre de 2026. Nombre, letras y cifras pueden haber cambiado; la forma de la ficha es lo que tiene que coincidir:

```text
Código:         3017620422003
Nombre:         Nutella
Marca:          Nutella, Ferrero
Nutri-Score:    e
NOVA:           4
Green-Score:    unknown
Alérgenos:      en:milk, en:nuts, en:soybeans
Sal / 100 g:    0.107 g
Azúcares / 100 g: 56.3 g
Ficha:          https://world.openfoodfacts.org/product/3017620422003
```

Abre esa URL en el navegador. El nombre y el Nutri-Score de la página tienen que ser los que acaba de imprimir la terminal. Esa comprobación es la demo: el programa enseña la ficha de Open Food Facts, no un texto generado.

`unknown` en el Green-Score es un valor devuelto por la API. El programa no lo sustituye por una letra.

## 8. Compara dos productos

```bash
python3 ficha.py 3017620422003 5449000000996
```

El segundo código es una Coca-Cola presente en la base. Verás dos fichas y, debajo, la comparación. En la misma prueba de septiembre de 2026 los números fueron:

```text
Sal por 100 g
  Nutella: 0.107 g
  Coca-Cola: 0.0 g
  Menos sal: Coca-Cola (5449000000996).

Azúcares por 100 g
  Nutella: 56.3 g
  Coca-Cola: 10.6 g
  Menos azúcares: Coca-Cola (5449000000996).
```

La frase “Menos sal” cita el nombre y el código. El 0.0 sale de `salt_100g` de esa ficha. Si mañana alguien corrige el producto, el programa imprimirá el número nuevo sin que cambies el código.

En esa misma respuesta la Coca-Cola traía `Green-Score: not-applicable` y `Alérgenos: (la ficha no trae allergens_tags)`. Las dos líneas se dejan así. Rellenarlas “porque una bebida de cola no lleva leche” sería inventar un dato que la ficha no trae.

## 9. Prueba un código que no existe

```bash
python3 ficha.py 0000000000000
```

El programa termina con:

```text
No hay ningún producto con el código 0000000000000 en Open Food Facts.
```

Y el código de salida es 1. Puedes verlo con:

```bash
python3 ficha.py 0000000000000
echo $?
```

`echo $?` imprime el código de la orden anterior. `0` es “ha ido bien”. `1` es “ha parado con un error controlado”.

Tres códigos también paran, con el mensaje `Pasa un código, o dos para compararlos.`

## 10. Usa un producto de tu cocina

1. Coge un envase con código de barras.
2. Lee el número que hay debajo de las barras. Suele tener 8 o 13 dígitos. Escríbelo seguido, sin espacios.
3. Lánzalo:

```bash
python3 ficha.py 1234567890123
```

Sustituye `1234567890123` por el número real.

Puede pasar cualquiera de estas tres cosas, y las tres son un resultado del laboratorio:

- Sale una ficha completa. Ábrela en el navegador y compárala con el envase.
- Sale la ficha con varios “(sin dato en Open Food Facts)”. Esos huecos son contribuciones que faltan. Anótalos: son el tipo de trabajo que luego hacen Hunger Games, Robotoff o la app.
- El programa dice que no hay producto. Ese código todavía no está en la base.

Para comparar el tuyo con la Nutella:

```bash
python3 ficha.py 3017620422003 TU_CODIGO
```

## 11. Si algo falla

| Lo que ves | Qué hacer |
| --- | --- |
| `python3: no se ha encontrado la orden` | Vuelve al paso 3. En algunos sistemas el comando es `python` |
| `command not found: curl` | Instala `curl` con el paso 3. El programa en sí no usa curl |
| `No se ha podido conectar` | Revisa la red. Prueba `curl -I https://world.openfoodfacts.org` |
| `La API ha respondido 503` o `429` | El servidor público está saturado o ha limitado la ráfaga. Espera un minuto y repite una sola llamada |
| `SyntaxError` al lanzar `ficha.py` | El pegado se ha cortado. Vuelve a copiar el programa entero, desde `#!/usr/bin/env python3` hasta la última línea |
| La ficha sale vacía o con otro producto | Comprueba que el código no tiene espacios y que es el de las barras, no el de un lote o una fecha de caducidad |
| `ModuleNotFoundError` | No hace falta instalar nada. Estás lanzando otro Python, o el fichero se ha guardado con otro nombre |

Comprueba que estás en la carpeta correcta:

```bash
pwd
ls
```

Tiene que verse `ficha.py` dentro de `laboratorio-off`.

## Qué has practicado del track

Esto es la base común y el principio del reto Food API & AI, en [`track.md`](track.md):

- Un `GET` a la API v3, con `User-Agent` y con `fields`.
- La diferencia entre un dato presente (`e`, `unknown`, `0.107`), un campo ausente y un producto que no existe.
- Una comparación que solo usa números de la ficha y cita el código de barras.
- El hábito de contrastar la terminal con la página del producto.

El paso siguiente del mismo reto, cuando esta ficha ya te resulte familiar, es envolver `fetch` y `compare` en herramientas de un servidor MCP. Ese servidor no hace falta para esta demo.
