# Backup de Experience Cloud con ExperienceBundle Metadata API

**Aplica a:** Salesforce Experience Cloud (cualquier nube que use sitios LWR)  
**Audiencia:** Admins, Developers, Partners  
**Nivel:** Intermedio

---

## ¿Para qué sirve esto?

Cuando configurás un sitio de Experience Cloud (colores, logo, banners, páginas, navegación), toda esa configuración vive dentro del org de Salesforce.

Este instructivo explica cómo exportar esa configuración como archivos versionables en GitHub, para tenerla respaldada, trabajar con versiones y poder restaurarla o llevarla a otro ambiente cuando lo necesites.

---

## LWR vs Aura — ¿Por qué esto solo funciona en LWR?

Experience Cloud tiene dos frameworks de renderizado. Entender la diferencia explica por qué ExperienceBundle solo existe en uno de ellos.

### Aura (el framework original)
- **Lanzado:** 2014 con Salesforce Communities
- Arquitectura basada en componentes Aura (propietarios de Salesforce)
- La configuración del sitio está **embebida dentro del org** de forma no exportable como metadata estructurada
- No tiene soporte para ExperienceBundle
- Salesforce lo mantiene pero **no lo desarrolla activamente** — está en modo mantenimiento
- Templates Aura más comunes: Customer Community, Partner Community, Customer Community Plus

### LWR — Lightning Web Runtime (el framework moderno)
- **Lanzado:** 2021 (GA en Winter '22)
- Arquitectura moderna basada en estándares web (LWC, ES modules)
- La configuración del sitio se expone como **metadata estructurada y exportable** → esto es el ExperienceBundle
- Soporte completo para backup, versionado y deploy entre orgs via Salesforce CLI
- Toda la inversión nueva de Salesforce va a LWR
- Templates LWR más comunes: Build Your Own (LWR), Microsite (LWR), Commerce B2B/B2C

### Resumen de diferencias

| | Aura | LWR |
|---|---|---|
| Año de lanzamiento | 2014 | 2021 (GA Winter '22) |
| Framework base | Componentes Aura | LWC + estándares web |
| ExperienceBundle (backup/deploy) | ❌ No soportado | ✅ Soportado |
| Velocidad de carga | Más lenta | Más rápida (SSR) |
| Desarrollo activo por Salesforce | ❌ Solo mantenimiento | ✅ Sí |
| Recomendado para proyectos nuevos | ❌ No | ✅ Sí |

> **En resumen:** Aura no puede exportar su configuración como metadata porque fue diseñado antes de que Salesforce tuviera este modelo de versionado. LWR fue construido desde el inicio con esta capacidad. Si tu sitio es Aura, la única opción es reconfigurarlo manualmente o migrarlo a LWR.

---

## Pre-requisitos

- **[Salesforce CLI](https://developer.salesforce.com/tools/salesforcecli) (gratis)** — herramienta de línea de comandos oficial de Salesforce. Permite interactuar con tus orgs desde la terminal: descargar metadata, deployar cambios, correr scripts, etc.

- **Proyecto SFDX configurado localmente** — es una carpeta en tu computadora con una estructura específica que el CLI de Salesforce reconoce. Básicamente es donde van a quedar guardados los archivos que descargues del org. Si no tenés uno, crealo con:
  ```bash
  sf project generate --name mi-proyecto
  cd mi-proyecto
  ```

- Acceso de administrador al org

- Git instalado y repositorio configurado

---

## Paso 1 — Habilitar ExperienceBundle Metadata API en el org

Esta opción viene deshabilitada por defecto. Sin ella, el CLI no puede exportar el sitio.

1. Abrí el org en el navegador:
   ```bash
   sf org open --target-org <alias-de-tu-org>
   ```
2. En Setup, buscá **"Digital Experiences"** en la barra de búsqueda
3. Hacé click en **Settings**
4. Tildá la opción **"Enable ExperienceBundle Metadata API"**
5. Guardá

> **¿Qué hace esta opción?** Le indica a Salesforce que exponga la configuración visual del sitio (tema, colores, páginas, componentes) como metadata descargable via CLI. Sin esta opción, el retrieve devuelve "Nothing retrieved" sin dar un error claro.

---

## Paso 2 — Verificar que el org está conectado

```bash
sf org display --target-org <alias-de-tu-org>
```

**Si la conexión está ok**, vas a ver algo así:

```
=== Org Description
 KEY             VALUE
 ───────────     ──────────────────────────────────────────────
 Access Token    00DKa000...
 Alias           mi-sdo
 Client Id       PlatformCLI
 Instance Url    https://tu-org.my.salesforce.com
 Org Id          00DKa000000eNGGf
 Username        tu.usuario@ejemplo.com
```

**Si da error** porque el org no está autenticado, verás algo como:

```
Error (NoOrgFound): The org with alias or username "mi-sdo" is not connected.
Run "sf org login web --alias mi-sdo" to authenticate.
```

En ese caso, autenticarte primero:

```bash
sf org login web --alias <alias-de-tu-org>
```

Esto va a abrir el navegador para que ingreses con tu usuario y contraseña de Salesforce. Una vez que iniciás sesión, el CLI queda conectado y podés continuar con los pasos siguientes.

---

## Paso 3 — Ejecutar el retrieve del ExperienceBundle

Este comando descarga toda la configuración visual del sitio:

```bash
sf project retrieve start --metadata "ExperienceBundle" --target-org <alias-de-tu-org>
```

### ¿Qué trae este comando?

Una vez completado, encontrarás los archivos en:

```
force-app/main/default/experiences/<NombreDelSitio>/
├── config/                  ← Configuración general del sitio
├── routes/                  ← Rutas y URLs de las páginas
├── views/                   ← Páginas del sitio (Home, PDP, carrito, etc.)
├── theme/                   ← Colores, fuentes y variables de diseño
└── sfdc_cms__imageAssets/   ← Logo y assets de imagen configurados
```

### Ejemplo de salida exitosa:

```
Status: Succeeded
Retrieved source:
  force-app/main/default/experiences/AccountA/config/AccountA.json
  force-app/main/default/experiences/AccountA/theme/theme.json
  force-app/main/default/experiences/AccountA/views/home.json
  ... (más archivos)
```

---

## Paso 4 — Guardar en GitHub

Una vez descargados los archivos, hacelos commit y push:

```bash
# Ver qué archivos se descargaron
git status

# Agregar los archivos al staging
git add force-app/main/default/experiences/
git add force-app/main/default/sites/

# Crear el commit
git commit -m "Backup ExperienceBundle sitio Experience Cloud"

# Subir a GitHub
git push origin main
```

---

## Paso 5 — Restaurar en otro org (cuando lo necesites)

Para deployar el sitio en un nuevo org:

```bash
sf project deploy start --metadata "ExperienceBundle" --target-org <alias-del-nuevo-org>
```

> **Importante:** El nuevo org debe tener Experience Cloud habilitado y la opción "Enable ExperienceBundle Metadata API" activa antes del deploy.

---

## ¿Qué incluye y qué no incluye el ExperienceBundle?

| Qué | ¿Lo incluye? |
|---|---|
| Colores y tema visual | ✅ Sí |
| Logo del sitio | ✅ Sí |
| Estructura de páginas y componentes | ✅ Sí |
| Menú de navegación | ✅ Sí |
| Configuración general del sitio | ✅ Sí |
| Imágenes subidas al CMS | ❌ No — ver Sección A |
| Imágenes de productos (vinculadas al catálogo) | ❌ No — ver Sección B |
| Usuarios y perfiles | ❌ No |

---

## Sección A — Cómo descargar imágenes del CMS

Las imágenes subidas desde Experience Builder o desde la UI de Commerce Cloud viven en **ManagedContent** (el CMS de Salesforce), no en el ExperienceBundle. Hay que descargarlas por separado via API REST.

### Script Python — descargar todas las imágenes del CMS

Guardá este script como `download_cms_images.py` y correlo:

```python
#!/usr/bin/env python3
"""
Descarga todas las imágenes del CMS de Salesforce al directorio local.
Uso: python3 download_cms_images.py --org <alias> --output ./imagenes
"""
import argparse, json, os, subprocess, urllib.request

def get_org_info(org):
    r = subprocess.run(["sf", "org", "display", "--target-org", org, "--json"],
                       capture_output=True, text=True)
    result = json.loads(r.stdout)["result"]
    return result["accessToken"], result["instanceUrl"]

def soql(query, org):
    r = subprocess.run(["sf", "data", "query", "--query", query,
                        "--target-org", org, "--json"],
                       capture_output=True, text=True)
    return json.loads(r.stdout).get("result", {}).get("records", [])

def download(url, token, filepath):
    req = urllib.request.Request(url, headers={"Authorization": f"Bearer {token}"})
    with urllib.request.urlopen(req) as r:
        with open(filepath, "wb") as f:
            f.write(r.read())

parser = argparse.ArgumentParser()
parser.add_argument("--org", required=True)
parser.add_argument("--output", default="./cms_images")
args = parser.parse_args()

os.makedirs(args.output, exist_ok=True)
token, instance = get_org_info(args.org)

# Traer todos los contenidos de tipo imagen del CMS
records = soql("SELECT Id, Name, ContentKey FROM ManagedContent ORDER BY Name", args.org)
print(f"Encontrados: {len(records)} contenidos en CMS")

for r in records:
    key = r.get("ContentKey")
    name = r["Name"]
    if not key:
        continue

    # Obtener la URL real del archivo via API de CMS
    url = f"{instance}/services/data/v62.0/connect/cms/contents/{key}"
    req = urllib.request.Request(url, headers={"Authorization": f"Bearer {token}"})
    try:
        with urllib.request.urlopen(req) as resp:
            content = json.loads(resp.read())
    except Exception as e:
        print(f"  ✗ Error obteniendo metadata de {name}: {e}")
        continue

    # La URL del binario está en contentBody.source.url
    source = content.get("contentBody", {}).get("source", {})
    img_url = source.get("url")
    filename = source.get("filename") or f"{name}.jpg"

    if not img_url:
        print(f"  ✗ Sin URL de imagen: {name}")
        continue

    full_url = instance + img_url if img_url.startswith("/") else img_url
    filepath = os.path.join(args.output, filename)

    try:
        download(full_url, token, filepath)
        print(f"  ✓ {filename}")
    except Exception as e:
        print(f"  ✗ Error descargando {filename}: {e}")

print(f"\nImágenes guardadas en: {args.output}")
```

### ¿Cómo se corre este script?

Tenés dos opciones:

**Opción 1 — Terminal (recomendado)**

Es la forma más directa. Abrís una terminal, navegás a la carpeta del proyecto y corrés:

```bash
python3 download_cms_images.py --org <alias-de-tu-org> --output ./cms_images
```

**Salida esperada en terminal:**

```
Encontrados: 8 contenidos en CMS
  ✓ logo-empresa.png
  ✓ banner-hero-inicio.jpg
  ✓ banner-categoria-verano.png
  ✓ banner-promocion-descuento.jpg
  ✓ banner-landing-servicio.png
  ✓ imagen-destacada-home.jpg
  ... (una línea por archivo)

Imágenes guardadas en: ./cms_images
```

**Opción 2 — Postman**

Si preferís no usar la terminal, podés hacer la misma descarga con Postman llamando directamente a la API REST de Salesforce. Son dos llamadas por imagen:

---

**Llamada 1 — Obtener la metadata de la imagen (y su URL de descarga)**

Antes de hacer esta llamada, necesitás el `accessToken` y el `instanceUrl` de tu org:

```bash
sf org display --target-org <alias> --json
```

Luego en Postman:

```
Método:  GET
URL:     https://<instanceUrl>/services/data/v62.0/connect/cms/contents/<ContentKey>

Headers:
  Authorization: Bearer <accessToken>
  Content-Type:  application/json
```

Respuesta que devuelve Salesforce:

```json
{
  "contentKey": "MC2ZRH5GYHAZESRAEJEMNMIIJFRI",
  "name": "banner-hero-inicio",
  "contentBody": {
    "source": {
      "nodeType": "MEDIA",
      "mediaType": "IMAGE",
      "filename": "banner-hero-inicio.png",
      "mimeType": "image/png",
      "url": "/cms/media/MC2ZRH5GYHAZESRAEJEMNMIIJFRI?cb=05TKa00002CVe0k"
    },
    "title": "banner-hero-inicio"
  }
}
```

El campo que te interesa es `contentBody.source.url` — esa es la ruta al binario de la imagen.

---

**Llamada 2 — Descargar el binario de la imagen**

Con la URL del paso anterior:

```
Método:  GET
URL:     https://<instanceUrl>/cms/media/MC2ZRH5GYHAZESRAEJEMNMIIJFRI?cb=05TKa00002CVe0k

Headers:
  Authorization: Bearer <accessToken>
```

Postman va a recibir el archivo de imagen directamente. Para guardarlo: en la respuesta hacé click en **Save Response → Save to a file**.

---

> El script Python hace exactamente estas dos llamadas por cada imagen del CMS, de forma automática para todas a la vez. Con Postman hacés lo mismo pero imagen por imagen, lo cual sirve para inspeccionar o descargar una imagen puntual sin necesidad de correr código.

---

**¿Qué necesitás instalado para correr el script Python?**

Solo dos cosas, ambas gratuitas:

| Requisito | Para qué sirve | Cómo instalarlo |
|---|---|---|
| **Python 3** | Correr el script | [python.org/downloads](https://www.python.org/downloads/) — en Mac ya viene preinstalado |
| **Salesforce CLI** | Obtener el token del org automáticamente | [developer.salesforce.com/tools/salesforcecli](https://developer.salesforce.com/tools/salesforcecli) |

El script no usa ninguna librería externa — solo módulos que vienen incluidos con Python (`json`, `os`, `subprocess`, `urllib`). No necesitás correr `pip install` ni nada adicional.

### Guardar en GitHub

```bash
git add cms_images/
git commit -m "Backup imágenes CMS"
git push origin main
```

---

## Sección B — Imágenes de productos vinculadas al catálogo

Cuando un sitio de Experience Cloud muestra productos (una landing de catálogo, una comunidad con productos destacados), las imágenes que aparecen no vienen del ExperienceBundle — viven en **`ProductMedia`**, que las vincula al CMS (`ManagedContent`).

Es el mismo mecanismo que usa Commerce Cloud para mostrar imágenes en el storefront. Son el mismo objeto, la misma imagen, consumida por distintas superficies.

---

### Cómo funciona la relación

```
Product2 (el producto)
    └── ProductMedia (el vínculo)
            └── ManagedContent (la imagen en el CMS)
```

El ExperienceBundle no exporta ninguna de estas tres capas. Para respaldarlas necesitás dos cosas:

1. **El mapeo** — qué imagen está asociada a qué producto (via SOQL)
2. **Los binarios** — los archivos de imagen físicos (via Sección A)

---

### Paso 1 — Exportar el mapeo producto → imagen

```bash
sf data query \
  --query "SELECT Id, ProductId, ManagedContentId, SortOrder, ElectronicMediaGroupId FROM ProductMedia ORDER BY ProductId, SortOrder" \
  --target-org <alias-de-tu-org> \
  --result-format csv > export/product_media.csv
```

**Salida esperada en `export/product_media.csv`:**

```
Id,ProductId,ManagedContentId,SortOrder,ElectronicMediaGroupId
02uKa000001XAAB,01tKa00000C02nAIAR,20YKa000000KZ3kKAG,0,2mgKa000000etOXIAY
02uKa000001XAAC,01tKa00000C02nBIAR,20YKa000000KZ3lKAG,0,2mgKa000000etOXIAY
02uKa000001XAAD,01tKa00000C02nBIAR,20YKa000000KZ3mKAG,1,2mgKa000000etOXIAY
...
```

Cada fila es una imagen asociada a un producto. El campo `SortOrder` indica el orden cuando hay varias imágenes para el mismo producto. El `ManagedContentId` es la referencia al archivo en el CMS.

---

### Paso 2 — Descargar los binarios de las imágenes

Con el mapeo guardado, descargá los archivos físicos usando el script de la **Sección A**. Ese script recorre todos los `ManagedContent` del CMS y descarga cada imagen a una carpeta local.

---

### Guardar en GitHub

```bash
git add export/product_media.csv
git add cms_images/
git commit -m "Backup imágenes de productos — mapeo y binarios"
git push origin main
```

> Para exportar los datos del catálogo (productos, precios, pedidos, cuentas) ese tema corresponde a un tutorial separado — el foco acá es el respaldo visual del sitio.

---

### Guardar todo en GitHub

```bash
git add export/
git commit -m "Backup datos — productos, precios, pedidos, cuentas, contactos"
git push origin main
```

> **Nota:** Los datos exportados son una foto del momento. No se restauran automáticamente con un deploy — para reimportarlos en otro org necesitás un script de carga que lea los CSVs e inserte los registros via API.

---

## Notas adicionales

- El retrieve es **no destructivo** — solo lee, nunca modifica el org
- Los archivos resultantes son JSON/XML legibles y versionables en cualquier sistema de control de versiones
- Funciona en producción, sandboxes y Developer Orgs

---

*Parte de la serie de tutoriales Salesforce Experience Cloud.*






