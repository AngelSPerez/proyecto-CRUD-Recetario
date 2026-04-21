# Guía completa — CRUD Flutter + Firebase Recetario

> Práctica guiada para estudiantes | VS Code / Android Studio | Flutter + Firestore

---

## Índice

1. [Preparación del entorno](#1-preparación-del-entorno)
2. [Configuración de Firebase desde consola](#2-configuración-de-firebase-desde-consola)
3. [Estructura de carpetas](#3-estructura-de-carpetas)
4. [pubspec.yaml — dependencias](#4-pubspecyaml--dependencias)
5. [Archivos Dart completos](#5-archivos-dart-completos)
   - [recipe.dart — Modelo](#51-recipedart--modelo)
   - [recipe_service.dart — Servicio CRUD](#52-recipe_servicedart--servicio-crud)
   - [recipe_provider.dart — Estado](#53-recipe_providerdart--estado)
   - [main.dart — Punto de entrada](#54-maindart--punto-de-entrada)
   - [recipe_list_screen.dart — Pantalla principal](#55-recipe_list_screendart--pantalla-principal)
   - [add_edit_recipe_screen.dart — Crear y editar](#56-add_edit_recipe_screendart--crear-y-editar)
   - [recipe_detail_screen.dart — Ver detalle](#57-recipe_detail_screendart--ver-detalle)
6. [Reglas de seguridad Firestore](#6-reglas-de-seguridad-firestore)
7. [Metodología de agentes, roles y skills](#7-metodología-de-agentes-roles-y-skills)
8. [Flujo de trabajo para estudiantes](#8-flujo-de-trabajo-para-estudiantes)
9. [Comandos para correr el proyecto](#9-comandos-para-correr-el-proyecto)

---

## 1. Preparación del entorno

### Herramientas necesarias

- **Flutter SDK** — [flutter.dev/docs/get-started/install](https://flutter.dev/docs/get-started/install)
- **VS Code** (recomendado) con los plugins:
  - Flutter
  - Dart
  - Firebase Explorer (opcional)
- **Android Studio** como alternativa (AntiGravity es compatible con ambos)
- **Node.js** — necesario para Firebase CLI
- **Firebase CLI**

```bash
# Instalar Firebase CLI
npm install -g firebase-tools

# Verificar instalación de Flutter
flutter doctor

# Crear la carpeta del proyecto
mkdir crudrecetario
cd crudrecetario
flutter create .
```

---

## 2. Configuración de Firebase desde consola

### Paso 1 — Crear proyecto en Firebase

1. Ve a [console.firebase.google.com](https://console.firebase.google.com)
2. Clic en **"Agregar proyecto"**
3. Nombre del proyecto: `crudrecetario`
4. Desactiva Google Analytics (opcional para práctica)
5. Clic en **"Crear proyecto"**

### Paso 2 — Registrar la app Android

1. En la consola, clic en el ícono de Android
2. Package name: `com.example.crudrecetario`
3. Descarga el archivo **`google-services.json`**
4. Colócalo en `android/app/google-services.json`

### Paso 3 — Activar Firestore

1. En el menú lateral, clic en **"Firestore Database"**
2. Clic en **"Crear base de datos"**
3. Selecciona **"Modo de prueba"** (test mode)
4. Elige la región más cercana y confirma

### Paso 4 — Crear la colección `recetas`

1. En Firestore, clic en **"Iniciar colección"**
2. ID de colección: `recetas`
3. Agrega un documento de prueba con los campos:
   - `nombre` (string): `Tacos de pollo`
   - `descripcion` (string): `Receta tradicional mexicana`
   - `fecha` (timestamp): fecha actual

### Paso 5 — Configurar android/build.gradle

En `android/build.gradle`, dentro de `buildscript > dependencies`:

```groovy
classpath 'com.google.gms:google-services:4.4.2'
```

En `android/app/build.gradle`, al final del archivo:

```groovy
apply plugin: 'com.google.gms.google-services'
```

---

## 3. Estructura de carpetas

```
crudrecetario/
├── android/
│   └── app/
│       └── google-services.json        ← descargado de Firebase
├── lib/
│   ├── main.dart                       ← punto de entrada de la app
│   ├── models/
│   │   └── recipe.dart                 ← modelo de datos Recipe
│   ├── services/
│   │   └── recipe_service.dart         ← lógica CRUD con Firestore
│   ├── providers/
│   │   └── recipe_provider.dart        ← gestión de estado con Provider
│   └── screens/
│       ├── recipe_list_screen.dart     ← pantalla principal con lista
│       ├── add_edit_recipe_screen.dart ← formulario crear / editar
│       └── recipe_detail_screen.dart   ← vista de detalle de receta
├── pubspec.yaml                        ← dependencias del proyecto
└── README.md
```

---

## 4. `pubspec.yaml` — dependencias

```yaml
name: crudrecetario
description: CRUD de recetas con Flutter y Firebase Firestore

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.6.0
  cloud_firestore: ^5.4.0
  provider: ^6.1.2
  intl: ^0.19.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.0.0

flutter:
  uses-material-design: true
```

Después de editar el archivo, ejecuta:

```bash
flutter pub get
```

---

## 5. Archivos Dart completos

### 5.1 `recipe.dart` — Modelo

**Ubicación:** `lib/models/recipe.dart`

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

class Recipe {
  final String? id;
  final String nombre;
  final String descripcion;
  final DateTime fecha;

  Recipe({
    this.id,
    required this.nombre,
    required this.descripcion,
    required this.fecha,
  });

  /// Crea un Recipe desde un documento de Firestore
  factory Recipe.fromMap(Map<String, dynamic> map, String id) {
    return Recipe(
      id: id,
      nombre: map['nombre'] ?? '',
      descripcion: map['descripcion'] ?? '',
      fecha: (map['fecha'] as Timestamp).toDate(),
    );
  }

  /// Convierte el Recipe a un Map para guardar en Firestore
  Map<String, dynamic> toMap() {
    return {
      'nombre': nombre,
      'descripcion': descripcion,
      'fecha': Timestamp.fromDate(fecha),
    };
  }

  /// Crea una copia del Recipe con campos modificados
  Recipe copyWith({
    String? id,
    String? nombre,
    String? descripcion,
    DateTime? fecha,
  }) {
    return Recipe(
      id: id ?? this.id,
      nombre: nombre ?? this.nombre,
      descripcion: descripcion ?? this.descripcion,
      fecha: fecha ?? this.fecha,
    );
  }
}
```

---

### 5.2 `recipe_service.dart` — Servicio CRUD

**Ubicación:** `lib/services/recipe_service.dart`

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/recipe.dart';

class RecipeService {
  // Referencia a la colección "recetas" en Firestore
  final CollectionReference _col =
      FirebaseFirestore.instance.collection('recetas');

  // CREATE — Agregar nueva receta
  Future<void> addRecipe(Recipe recipe) async {
    await _col.add(recipe.toMap());
  }

  // READ — Stream en tiempo real ordenado por fecha descendente
  Stream<List<Recipe>> getRecipes() {
    return _col
        .orderBy('fecha', descending: true)
        .snapshots()
        .map((snapshot) => snapshot.docs
            .map((doc) => Recipe.fromMap(
                doc.data() as Map<String, dynamic>, doc.id))
            .toList());
  }

  // UPDATE — Actualizar receta existente por ID
  Future<void> updateRecipe(Recipe recipe) async {
    await _col.doc(recipe.id).update(recipe.toMap());
  }

  // DELETE — Borrar receta por ID
  Future<void> deleteRecipe(String id) async {
    await _col.doc(id).delete();
  }
}
```

---

### 5.3 `recipe_provider.dart` — Estado

**Ubicación:** `lib/providers/recipe_provider.dart`

```dart
import 'package:flutter/material.dart';
import '../models/recipe.dart';
import '../services/recipe_service.dart';

class RecipeProvider extends ChangeNotifier {
  final RecipeService _service = RecipeService();
  bool isLoading = false;
  String? error;

  /// Expone el stream de recetas directamente desde el servicio
  Stream<List<Recipe>> get recipesStream => _service.getRecipes();

  /// Agregar nueva receta
  Future<void> addRecipe(Recipe recipe) async {
    isLoading = true;
    notifyListeners();
    try {
      await _service.addRecipe(recipe);
    } catch (e) {
      error = e.toString();
    } finally {
      isLoading = false;
      notifyListeners();
    }
  }

  /// Actualizar receta existente
  Future<void> updateRecipe(Recipe recipe) async {
    isLoading = true;
    notifyListeners();
    try {
      await _service.updateRecipe(recipe);
    } catch (e) {
      error = e.toString();
    } finally {
      isLoading = false;
      notifyListeners();
    }
  }

  /// Borrar receta por ID
  Future<void> deleteRecipe(String id) async {
    try {
      await _service.deleteRecipe(id);
    } catch (e) {
      error = e.toString();
      notifyListeners();
    }
  }
}
```

---

### 5.4 `main.dart` — Punto de entrada

**Ubicación:** `lib/main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:provider/provider.dart';
import 'providers/recipe_provider.dart';
import 'screens/recipe_list_screen.dart';

void main() async {
  // Necesario antes de cualquier llamada asíncrona
  WidgetsFlutterBinding.ensureInitialized();
  // Inicializar Firebase
  await Firebase.initializeApp();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => RecipeProvider(),
      child: MaterialApp(
        title: 'Recetario',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepOrange),
          useMaterial3: true,
        ),
        home: const RecipeListScreen(),
      ),
    );
  }
}
```

---

### 5.5 `recipe_list_screen.dart` — Pantalla principal

**Ubicación:** `lib/screens/recipe_list_screen.dart`

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:intl/intl.dart';
import '../providers/recipe_provider.dart';
import '../models/recipe.dart';
import 'add_edit_recipe_screen.dart';
import 'recipe_detail_screen.dart';

class RecipeListScreen extends StatelessWidget {
  const RecipeListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final provider = context.watch<RecipeProvider>();

    return Scaffold(
      appBar: AppBar(
        title: const Text('Recetario'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: StreamBuilder<List<Recipe>>(
        stream: provider.recipesStream,
        builder: (context, snapshot) {
          // Estado de carga inicial
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          // Error de conexión
          if (snapshot.hasError) {
            return Center(child: Text('Error: ${snapshot.error}'));
          }
          final recipes = snapshot.data ?? [];
          // Lista vacía
          if (recipes.isEmpty) {
            return const Center(
              child: Text(
                'Sin recetas. ¡Toca + para crear una!',
                style: TextStyle(fontSize: 16),
              ),
            );
          }
          // Lista de recetas
          return ListView.builder(
            itemCount: recipes.length,
            itemBuilder: (context, index) {
              final recipe = recipes[index];
              return Card(
                margin: const EdgeInsets.symmetric(
                    horizontal: 12, vertical: 6),
                child: ListTile(
                  title: Text(
                    recipe.nombre,
                    style: const TextStyle(fontWeight: FontWeight.w600),
                  ),
                  subtitle: Text(
                    DateFormat('dd/MM/yyyy').format(recipe.fecha),
                  ),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      // Botón editar
                      IconButton(
                        icon: const Icon(Icons.edit, color: Colors.blue),
                        tooltip: 'Editar',
                        onPressed: () => Navigator.push(
                          context,
                          MaterialPageRoute(
                            builder: (_) =>
                                AddEditRecipeScreen(recipe: recipe),
                          ),
                        ),
                      ),
                      // Botón borrar
                      IconButton(
                        icon: const Icon(Icons.delete, color: Colors.red),
                        tooltip: 'Eliminar',
                        onPressed: () => _confirmDelete(
                            context, provider, recipe.id!),
                      ),
                    ],
                  ),
                  // Toca para ver detalle
                  onTap: () => Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (_) =>
                          RecipeDetailScreen(recipe: recipe),
                    ),
                  ),
                ),
              );
            },
          );
        },
      ),
      // Botón para agregar nueva receta
      floatingActionButton: FloatingActionButton(
        onPressed: () => Navigator.push(
          context,
          MaterialPageRoute(
              builder: (_) => const AddEditRecipeScreen()),
        ),
        tooltip: 'Nueva receta',
        child: const Icon(Icons.add),
      ),
    );
  }

  /// Diálogo de confirmación antes de borrar
  void _confirmDelete(
      BuildContext context, RecipeProvider provider, String id) {
    showDialog(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text('Borrar receta'),
        content:
            const Text('¿Estás seguro de que deseas eliminar esta receta?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Cancelar'),
          ),
          ElevatedButton(
            style: ElevatedButton.styleFrom(
                backgroundColor: Colors.red),
            onPressed: () {
              provider.deleteRecipe(id);
              Navigator.pop(context);
            },
            child: const Text(
              'Eliminar',
              style: TextStyle(color: Colors.white),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

### 5.6 `add_edit_recipe_screen.dart` — Crear y editar

**Ubicación:** `lib/screens/add_edit_recipe_screen.dart`

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../models/recipe.dart';
import '../providers/recipe_provider.dart';

class AddEditRecipeScreen extends StatefulWidget {
  // Si recipe es null, se crea una nueva; si tiene valor, se edita
  final Recipe? recipe;
  const AddEditRecipeScreen({super.key, this.recipe});

  @override
  State<AddEditRecipeScreen> createState() => _AddEditRecipeScreenState();
}

class _AddEditRecipeScreenState extends State<AddEditRecipeScreen> {
  final _formKey = GlobalKey<FormState>();
  late TextEditingController _nombreCtrl;
  late TextEditingController _descripcionCtrl;
  DateTime _fecha = DateTime.now();

  bool get isEditing => widget.recipe != null;

  @override
  void initState() {
    super.initState();
    // Prellenar campos si se está editando
    _nombreCtrl =
        TextEditingController(text: widget.recipe?.nombre ?? '');
    _descripcionCtrl =
        TextEditingController(text: widget.recipe?.descripcion ?? '');
    if (isEditing) _fecha = widget.recipe!.fecha;
  }

  @override
  void dispose() {
    _nombreCtrl.dispose();
    _descripcionCtrl.dispose();
    super.dispose();
  }

  /// Abre el selector de fecha
  Future<void> _pickDate() async {
    final picked = await showDatePicker(
      context: context,
      initialDate: _fecha,
      firstDate: DateTime(2000),
      lastDate: DateTime(2100),
    );
    if (picked != null) setState(() => _fecha = picked);
  }

  /// Valida y guarda / actualiza la receta
  Future<void> _submit() async {
    if (!_formKey.currentState!.validate()) return;

    final provider = context.read<RecipeProvider>();
    final recipe = Recipe(
      id: widget.recipe?.id,
      nombre: _nombreCtrl.text.trim(),
      descripcion: _descripcionCtrl.text.trim(),
      fecha: _fecha,
    );

    if (isEditing) {
      await provider.updateRecipe(recipe);
    } else {
      await provider.addRecipe(recipe);
    }

    if (mounted) Navigator.pop(context);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(isEditing ? 'Editar receta' : 'Nueva receta'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: ListView(
            children: [
              // Campo nombre
              TextFormField(
                controller: _nombreCtrl,
                decoration: const InputDecoration(
                  labelText: 'Nombre de la receta',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.restaurant_menu),
                ),
                validator: (v) =>
                    v == null || v.isEmpty ? 'Campo requerido' : null,
              ),
              const SizedBox(height: 16),

              // Campo descripción
              TextFormField(
                controller: _descripcionCtrl,
                decoration: const InputDecoration(
                  labelText: 'Descripción',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.description),
                  alignLabelWithHint: true,
                ),
                maxLines: 5,
                validator: (v) =>
                    v == null || v.isEmpty ? 'Campo requerido' : null,
              ),
              const SizedBox(height: 16),

              // Selector de fecha
              Row(
                children: [
                  Icon(Icons.calendar_today,
                      color: Theme.of(context).colorScheme.primary),
                  const SizedBox(width: 8),
                  Text(
                    'Fecha: ${_fecha.day}/${_fecha.month}/${_fecha.year}',
                    style: const TextStyle(fontSize: 16),
                  ),
                  const SizedBox(width: 12),
                  OutlinedButton(
                    onPressed: _pickDate,
                    child: const Text('Cambiar'),
                  ),
                ],
              ),
              const SizedBox(height: 28),

              // Botón guardar
              ElevatedButton(
                onPressed: _submit,
                style: ElevatedButton.styleFrom(
                  padding: const EdgeInsets.symmetric(vertical: 14),
                ),
                child: Text(
                  isEditing ? 'Actualizar receta' : 'Guardar receta',
                  style: const TextStyle(fontSize: 16),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

### 5.7 `recipe_detail_screen.dart` — Ver detalle

**Ubicación:** `lib/screens/recipe_detail_screen.dart`

```dart
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import '../models/recipe.dart';
import 'add_edit_recipe_screen.dart';

class RecipeDetailScreen extends StatelessWidget {
  final Recipe recipe;
  const RecipeDetailScreen({super.key, required this.recipe});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(recipe.nombre),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        actions: [
          // Botón de edición desde la pantalla de detalle
          IconButton(
            icon: const Icon(Icons.edit),
            tooltip: 'Editar',
            onPressed: () => Navigator.pushReplacement(
              context,
              MaterialPageRoute(
                builder: (_) => AddEditRecipeScreen(recipe: recipe),
              ),
            ),
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Nombre
            Text(
              recipe.nombre,
              style: const TextStyle(
                fontSize: 26,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),

            // Fecha formateada
            Row(
              children: [
                Icon(Icons.calendar_today,
                    size: 16, color: Colors.grey[600]),
                const SizedBox(width: 6),
                Text(
                  DateFormat('dd \'de\' MMMM \'de\' yyyy', 'es_MX')
                      .format(recipe.fecha),
                  style: TextStyle(
                    color: Colors.grey[600],
                    fontSize: 14,
                  ),
                ),
              ],
            ),
            const Divider(height: 32),

            // Descripción
            const Text(
              'Descripción',
              style: TextStyle(
                fontSize: 18,
                fontWeight: FontWeight.w600,
              ),
            ),
            const SizedBox(height: 10),
            Text(
              recipe.descripcion,
              style: const TextStyle(
                fontSize: 16,
                height: 1.7,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## 6. Reglas de seguridad Firestore

En la consola de Firebase → Firestore → pestaña **Reglas**:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /recetas/{recipeId} {
      // Solo para desarrollo/práctica — en producción agregar autenticación
      allow read, write: if true;
    }
  }
}
```

> **Nota para producción:** Cambiar `if true` por `if request.auth != null` cuando se integre Firebase Authentication.

---

## 7. Metodología de agentes, roles y skills

### Definición de agentes

| Agente | Rol | Skill principal | Entregable |
|---|---|---|---|
| **Architect** | Define la arquitectura de capas y carpetas | Patrones MVC / Clean Architecture | Estructura de carpetas + diagrama |
| **UI Developer** | Construye todos los widgets y pantallas | Flutter Material 3, formularios | Archivos `*_screen.dart` |
| **Backend Developer** | Configura Firebase y escribe el servicio | Firestore, consultas, streams | `recipe_service.dart` |
| **State Manager** | Gestiona el estado de la app | Provider, ChangeNotifier | `recipe_provider.dart` |
| **QA Tester** | Valida el flujo completo | Flutter test, integración | Casos de prueba |

### Skills por agente

**Architect**
- Análisis de requisitos del CRUD
- Diseño de la colección Firestore (`recetas`)
- Definición del modelo `Recipe` con sus 3 campos
- Establecer las capas: UI → Provider → Service → Firestore

**UI Developer**
- Crear `RecipeListScreen` con `StreamBuilder`
- Crear `AddEditRecipeScreen` con `Form` y validaciones
- Crear `RecipeDetailScreen` con formato de fecha
- Implementar `DeleteDialog` con confirmación

**Backend Developer**
- Crear proyecto en Firebase console
- Configurar `google-services.json`
- Escribir los 4 métodos CRUD en `RecipeService`
- Configurar reglas de seguridad en Firestore

**State Manager**
- Implementar `RecipeProvider` con `ChangeNotifier`
- Exponer el stream de Firestore a la UI
- Manejar estados de carga (`isLoading`) y errores

**QA Tester**
- Verificar que se crea una receta y aparece en la lista
- Verificar que editar actualiza los datos en Firestore
- Verificar que borrar elimina el documento
- Verificar que el stream actualiza la UI en tiempo real

---

## 8. Flujo de trabajo para estudiantes

```
Sesión 1 — Setup
  └─ Instalar Flutter y VS Code
  └─ Crear proyecto: flutter create crudrecetario
  └─ Configurar Firebase console
  └─ Agregar google-services.json

Sesión 2 — Modelo y Servicio
  └─ Crear lib/models/recipe.dart
  └─ Crear lib/services/recipe_service.dart
  └─ Probar conexión con Firestore desde main.dart

Sesión 3 — Estado (Provider)
  └─ Agregar dependencia provider en pubspec.yaml
  └─ Crear lib/providers/recipe_provider.dart
  └─ Conectar ChangeNotifierProvider en main.dart

Sesión 4 — Pantallas UI
  └─ Crear recipe_list_screen.dart con StreamBuilder
  └─ Crear add_edit_recipe_screen.dart con Form
  └─ Crear recipe_detail_screen.dart

Sesión 5 — Integración y pruebas
  └─ Probar CRUD completo en emulador o dispositivo
  └─ Revisar logs en Firebase console
  └─ Ajustar reglas de seguridad Firestore
```

---

## 9. Comandos para correr el proyecto

```bash
# Entrar a la carpeta del proyecto
cd crudrecetario

# Instalar dependencias
flutter pub get

# Verificar que todo esté correcto
flutter doctor

# Correr en modo debug (emulador o dispositivo conectado)
flutter run

# Correr en un dispositivo específico
flutter run -d <device_id>

# Listar dispositivos disponibles
flutter devices

# Compilar APK de release
flutter build apk --release
```

---

## Resumen de operaciones CRUD

| Operación | Método en RecipeService | Pantalla que lo usa |
|---|---|---|
| **Create** | `addRecipe(recipe)` | `AddEditRecipeScreen` (sin ID) |
| **Read** | `getRecipes()` — Stream | `RecipeListScreen` vía `StreamBuilder` |
| **Update** | `updateRecipe(recipe)` | `AddEditRecipeScreen` (con ID) |
| **Delete** | `deleteRecipe(id)` | `RecipeListScreen` → `_confirmDelete()` |

---

*Guía elaborada para la práctica de desarrollo móvil con Flutter y Firebase Firestore.*
*Proyecto: crudrecetario | Versión Flutter: 3.x | Versión Firebase: 3.x*
