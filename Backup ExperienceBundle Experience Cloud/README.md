# Backup de Experience Cloud con ExperienceBundle Metadata API

**Aplica a:** Salesforce Experience Cloud (cualquier nube que use sitios LWR)  
**Audiencia:** Admins, Developers, Partners  
**Nivel:** Intermedio

---

## ¿Para qué sirve esto?

Cuando configurás un sitio de Experience Cloud (colores, logo, banners, páginas, navegación), toda esa configuración vive dentro del org de Salesforce.

Este instructivo explica cómo exportar esa configuración como archivos versionables en GitHub, para tenerla respaldada, trabajar con versiones y poder restaurarla o llevarla a otro ambiente cuando lo necesites.

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

Deberías ver el `instanceUrl` y `username` del org. Si da error, autenticarte primero:

```bash
sf org login web --alias <alias-de-tu-org>
```

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
| Imágenes subidas al CMS | ❌ No — requieren descarga separada |
| Datos (productos, precios, pedidos) | ❌ No — requiere exportación de datos |
| Usuarios y perfiles | ❌ No |

---

## Sitios compatibles

| Template | Compatible |
|---|---|
| LWR (Lightning Web Runtime) — cualquier nube | ✅ Sí |
| Aura (templates antiguos) | ❌ No compatible con ExperienceBundle |

---

## Notas adicionales

- El retrieve es **no destructivo** — solo lee, nunca modifica el org
- Los archivos resultantes son JSON/XML legibles y versionables en cualquier sistema de control de versiones
- Funciona en producción, sandboxes y Developer Orgs

---

*Parte de la serie de tutoriales Salesforce Experience Cloud.*
