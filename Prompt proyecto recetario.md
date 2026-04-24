## Prompt: App Flutter "RecipeRecive"

**Crear la habilidad agente global `.agents` con un `SKILL.md` incluyendo las carpetas: `scripts`, `ejemplos`, `resources`.**

El usuario crea:
- **Skill de diseño** (UI/UX para recetario)
- **Skill de código** (lógica Flutter/Dart)
- **Skill de scraping** (opcional: importar recetas externas)

---

### Prerrequisitos

Verificar e instalar las siguientes herramientas desde terminal:

- **Flutter SDK** instalado y en `PATH`
- **FlutterFire CLI** instalado (`dart pub global activate flutterfire_cli`); guiar al usuario si no está disponible
- **Firebase CLI** instalado (`npm install -g firebase-tools`)
- Ejecutar `firebase login` para establecer conectividad con Firebase Console
- Proyecto creado en **Firebase Console** con:
  - Base de datos **Firestore** habilitada
  - App Flutter/Android registrada desde la consola
- Verificar configuración del archivo `pubspec.yaml` con las dependencias necesarias (`cloud_firestore`, `firebase_core`, etc.)
- **IDE:** VS Code / IDE Antigravity
- **Lenguaje:** Dart | **Framework:** Flutter | **Backend:** Firebase Firestore

---

### Objetivo del Proyecto

Crear una **app móvil de recetario** llamada **`RecipeRecive`**, construida con Flutter, cuya carpeta principal de trabajo será `recipe_recive/`.

> ⚠️ **Este proyecto NO es una tienda.** Es un **recetario digital**: una lista organizada de recetas con categorías, ingredientes y pasos de preparación.

---

### Alcance Inicial — CRUD de Recetas

Implementar las operaciones básicas sobre recetas:

| Operación | Descripción |
|---|---|
| **Crear** | Agregar nueva receta (nombre, categoría, ingredientes, pasos, imagen) |
| **Leer** | Listar todas las recetas y ver el detalle de cada una |
| **Actualizar** | Editar una receta existente |
| **Eliminar** | Borrar una receta de la lista |

---

### Pantallas UI a Generar

1. **Pantalla de Inicio** — Bienvenida con logo y acceso al recetario
2. **Lista de Recetas** — Vista general con tarjetas por categoría (desayunos, postres, platos principales, etc.)
3. **Detalle de Receta** — Ingredientes + pasos de preparación
4. **Formulario Crear/Editar Receta** — Campos estructurados
5. **Navegación principal** — Bottom navigation bar entre secciones

---

### Instrucciones de Generación

- Generar la **estructura completa de carpetas y archivos `.dart`** del proyecto
- Proveer el **código funcional y completo** para cada pantalla y componente
- Seguir una **secuencia lógica de desarrollo**:
  1. Configuración del entorno y Firebase
  2. Estructura del proyecto
  3. Modelos de datos (Recipe model)
  4. Servicios Firestore (CRUD)
  5. Pantallas UI
  6. Navegación
  7. Pruebas de funcionalidad
- Usar el agente `.agents` en cada etapa del proceso
- El proyecto debe quedar **totalmente funcional** al finalizar
