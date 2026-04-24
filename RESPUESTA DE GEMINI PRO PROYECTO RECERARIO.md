¡Hola! Es un excelente proyecto. Construir un recetario digital como **RecipeRecive** con Flutter y Firebase es una arquitectura sólida y escalable. Actuaré como tu asistente experto para generar toda la estructura, la configuración del agente y el código base funcional.

A continuación, ejecutaremos el plan paso a paso siguiendo tus instrucciones.

---

### Fase 1: Configuración de la Habilidad Global (`.agents`)

Primero, vamos a definir el espacio de trabajo del agente en tu repositorio. Ejecuta esto en tu terminal para crear la estructura:

```bash
mkdir -p .agents/scripts .agents/ejemplos .agents/resources
touch .agents/SKILL.md
```

**Contenido para `.agents/SKILL.md`:**

```markdown
# Agent Skill: RecipeRecive DevTeam
Este directorio contiene las habilidades y herramientas para el desarrollo de la app RecipeRecive.

## 🎨 Skill de Diseño (UI/UX)
- **Objetivo:** Definir la experiencia de usuario y la interfaz para un recetario digital.
- **Enfoque:** Tarjetas limpias, tipografía legible, colores cálidos (enfocados en comida).
- **Directorio de recursos:** `.agents/resources/design/`

## 💻 Skill de Código (Lógica Flutter/Dart)
- **Objetivo:** Implementar la arquitectura de la app usando Dart y Flutter.
- **Enfoque:** Patrón de repositorios, separación de UI y lógica, integración nativa con Firebase Firestore.
- **Directorio de scripts:** `.agents/scripts/` (Scripts de generación de boilerplate).

## 🕷️ Skill de Scraping (Opcional)
- **Objetivo:** Importar recetas estructuradas desde fuentes externas.
- **Enfoque:** Extracción en Python/Dart y conversión a formato JSON compatible con nuestra colección `recipes` en Firestore.
- **Directorio de ejemplos:** `.agents/ejemplos/scraping_templates/`
```

---

### Fase 2: Prerrequisitos y Entorno (Firebase & Flutter)

Asegúrate de ejecutar los siguientes comandos en tu terminal integrada (VS Code o Antigravity) para preparar tu entorno:

1. **Instalar dependencias globales:**
   ```bash
   npm install -g firebase-tools
   dart pub global activate flutterfire_cli
   ```
2. **Autenticación e Inicialización:**
   ```bash
   firebase login
   # Crea tu proyecto en Firebase Console web primero, luego inicializa FlutterFire
   flutterfire configure
   ```
3. **Configurar `pubspec.yaml`:**
   Añade las siguientes dependencias bajo `dependencies:` y ejecuta `flutter pub get`:
   ```yaml
   dependencies:
     flutter:
       sdk: flutter
     firebase_core: ^2.24.2
     cloud_firestore: ^4.13.6
     cupertino_icons: ^1.0.2
   ```

---

### Fase 3: Estructura del Proyecto Flutter

Dentro de la carpeta de tu app (`recipe_recive/lib/`), organizaremos el código así para mantener una arquitectura limpia:

```text
lib/
├── models/
│   └── recipe_model.dart       # Modelo de datos de la receta
├── services/
│   └── firestore_service.dart  # Lógica CRUD con Firebase
├── screens/
│   ├── home_screen.dart        # Bienvenida
│   ├── recipe_list_screen.dart # Lista de recetas (Read/Delete)
│   ├── recipe_detail_screen.dart# Detalle de la receta
│   └── recipe_form_screen.dart # Formulario (Create/Update)
├── widgets/
│   └── recipe_card.dart        # Tarjeta reutilizable de UI
└── main.dart                   # Punto de entrada y navegación
```

---

### Fase 4: Implementación del Código (CRUD)

Copia y pega estos bloques de código en sus respectivos archivos dentro de `lib/`.

#### 1. Modelos de Datos (`lib/models/recipe_model.dart`)
```dart
class Recipe {
  String id;
  String title;
  String category;
  List<String> ingredients;
  List<String> steps;
  String imageUrl;

  Recipe({
    this.id = '',
    required this.title,
    required this.category,
    required this.ingredients,
    required this.steps,
    this.imageUrl = '',
  });

  Map<String, dynamic> toMap() {
    return {
      'title': title,
      'category': category,
      'ingredients': ingredients,
      'steps': steps,
      'imageUrl': imageUrl,
    };
  }

  factory Recipe.fromMap(Map<String, dynamic> map, String documentId) {
    return Recipe(
      id: documentId,
      title: map['title'] ?? '',
      category: map['category'] ?? '',
      ingredients: List<String>.from(map['ingredients'] ?? []),
      steps: List<String>.from(map['steps'] ?? []),
      imageUrl: map['imageUrl'] ?? '',
    );
  }
}
```

#### 2. Servicios Firestore (`lib/services/firestore_service.dart`)
```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/recipe_model.dart';

class FirestoreService {
  final CollectionReference _recipesCollection =
      FirebaseFirestore.instance.collection('recipes');

  // Crear
  Future<void> addRecipe(Recipe recipe) async {
    await _recipesCollection.add(recipe.toMap());
  }

  // Leer (Stream para tiempo real)
  Stream<List<Recipe>> getRecipes() {
    return _recipesCollection.snapshots().map((snapshot) {
      return snapshot.docs.map((doc) {
        return Recipe.fromMap(doc.data() as Map<String, dynamic>, doc.id);
      }).toList();
    });
  }

  // Actualizar
  Future<void> updateRecipe(Recipe recipe) async {
    await _recipesCollection.doc(recipe.id).update(recipe.toMap());
  }

  // Eliminar
  Future<void> deleteRecipe(String id) async {
    await _recipesCollection.doc(id).delete();
  }
}
```

#### 3. Pantalla de Bienvenida (`lib/screens/home_screen.dart`)
```dart
import 'package:flutter/material.dart';

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.restaurant_menu, size: 100, color: Colors.orange),
          SizedBox(height: 20),
          Text(
            '¡Bienvenido a RecipeRecive!',
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          SizedBox(height: 10),
          Text('Tu recetario digital personal.'),
        ],
      ),
    );
  }
}
```

#### 4. Pantalla de Lista (`lib/screens/recipe_list_screen.dart`)
```dart
import 'package:flutter/material.dart';
import '../models/recipe_model.dart';
import '../services/firestore_service.dart';
import 'recipe_detail_screen.dart';
import 'recipe_form_screen.dart';

class RecipeListScreen extends StatelessWidget {
  final FirestoreService _firestoreService = FirestoreService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Mis Recetas')),
      body: StreamBuilder<List<Recipe>>(
        stream: _firestoreService.getRecipes(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return Center(child: CircularProgressIndicator());
          }
          if (!snapshot.hasData || snapshot.data!.isEmpty) {
            return Center(child: Text('No hay recetas aún. ¡Añade una!'));
          }

          final recipes = snapshot.data!;
          return ListView.builder(
            itemCount: recipes.length,
            itemBuilder: (context, index) {
              final recipe = recipes[index];
              return Card(
                margin: EdgeInsets.symmetric(horizontal: 10, vertical: 5),
                child: ListTile(
                  leading: Icon(Icons.fastfood, color: Colors.orange),
                  title: Text(recipe.title, style: TextStyle(fontWeight: FontWeight.bold)),
                  subtitle: Text(recipe.category),
                  onTap: () => Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => RecipeDetailScreen(recipe: recipe),
                    ),
                  ),
                  trailing: IconButton(
                    icon: Icon(Icons.delete, color: Colors.red),
                    onPressed: () => _firestoreService.deleteRecipe(recipe.id),
                  ),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => Navigator.push(
          context,
          MaterialPageRoute(builder: (context) => RecipeFormScreen()),
        ),
        child: Icon(Icons.add),
        backgroundColor: Colors.orange,
      ),
    );
  }
}
```

#### 5. Pantalla de Detalle (`lib/screens/recipe_detail_screen.dart`)
```dart
import 'package:flutter/material.dart';
import '../models/recipe_model.dart';
import 'recipe_form_screen.dart';

class RecipeDetailScreen extends StatelessWidget {
  final Recipe recipe;

  RecipeDetailScreen({required this.recipe});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(recipe.title),
        actions: [
          IconButton(
            icon: Icon(Icons.edit),
            onPressed: () => Navigator.push(
              context,
              MaterialPageRoute(
                builder: (context) => RecipeFormScreen(recipe: recipe),
              ),
            ),
          )
        ],
      ),
      body: SingleChildScrollView(
        padding: EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Chip(label: Text(recipe.category, style: TextStyle(color: Colors.white)), backgroundColor: Colors.orange),
            SizedBox(height: 20),
            Text('Ingredientes:', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            ...recipe.ingredients.map((item) => Text('• $item')).toList(),
            SizedBox(height: 20),
            Text('Preparación:', style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
            ...recipe.steps.asMap().entries.map((entry) {
              return Padding(
                padding: const EdgeInsets.only(top: 8.0),
                child: Text('${entry.key + 1}. ${entry.value}'),
              );
            }).toList(),
          ],
        ),
      ),
    );
  }
}
```

#### 6. Pantalla de Formulario (`lib/screens/recipe_form_screen.dart`)
```dart
import 'package:flutter/material.dart';
import '../models/recipe_model.dart';
import '../services/firestore_service.dart';

class RecipeFormScreen extends StatefulWidget {
  final Recipe? recipe;

  RecipeFormScreen({this.recipe});

  @override
  _RecipeFormScreenState createState() => _RecipeFormScreenState();
}

class _RecipeFormScreenState extends State<RecipeFormScreen> {
  final _formKey = GlobalKey<FormState>();
  final FirestoreService _firestoreService = FirestoreService();

  late TextEditingController _titleController;
  late TextEditingController _categoryController;
  late TextEditingController _ingredientsController;
  late TextEditingController _stepsController;

  @override
  void initState() {
    super.initState();
    _titleController = TextEditingController(text: widget.recipe?.title ?? '');
    _categoryController = TextEditingController(text: widget.recipe?.category ?? '');
    _ingredientsController = TextEditingController(
      text: widget.recipe?.ingredients.join(',') ?? '',
    );
    _stepsController = TextEditingController(
      text: widget.recipe?.steps.join(',') ?? '',
    );
  }

  void _saveRecipe() {
    if (_formKey.currentState!.validate()) {
      final newRecipe = Recipe(
        id: widget.recipe?.id ?? '',
        title: _titleController.text,
        category: _categoryController.text,
        ingredients: _ingredientsController.text.split(',').map((e) => e.trim()).where((e) => e.isNotEmpty).toList(),
        steps: _stepsController.text.split(',').map((e) => e.trim()).where((e) => e.isNotEmpty).toList(),
      );

      if (widget.recipe == null) {
        _firestoreService.addRecipe(newRecipe);
      } else {
        _firestoreService.updateRecipe(newRecipe);
      }
      Navigator.pop(context); // Vuelve a la pantalla anterior, o dos veces si estabas en detalle
      if (widget.recipe != null) Navigator.pop(context); // Cierra el detalle viejo
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(widget.recipe == null ? 'Nueva Receta' : 'Editar Receta')),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: ListView(
            children: [
              TextFormField(
                controller: _titleController,
                decoration: InputDecoration(labelText: 'Título de la receta'),
                validator: (value) => value!.isEmpty ? 'Requerido' : null,
              ),
              TextFormField(
                controller: _categoryController,
                decoration: InputDecoration(labelText: 'Categoría (ej. Postres)'),
                validator: (value) => value!.isEmpty ? 'Requerido' : null,
              ),
              TextFormField(
                controller: _ingredientsController,
                decoration: InputDecoration(labelText: 'Ingredientes (separados por coma)'),
                maxLines: 3,
              ),
              TextFormField(
                controller: _stepsController,
                decoration: InputDecoration(labelText: 'Pasos (separados por coma)'),
                maxLines: 3,
              ),
              SizedBox(height: 20),
              ElevatedButton(
                onPressed: _saveRecipe,
                child: Text('Guardar'),
                style: ElevatedButton.styleFrom(backgroundColor: Colors.orange),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

#### 7. Punto de Entrada y Navegación (`lib/main.dart`)
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home_screen.dart';
import 'screens/recipe_list_screen.dart';
// import 'firebase_options.dart'; // Generado por flutterfire configure

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    // options: DefaultFirebaseOptions.currentPlatform, // Descomenta tras correr flutterfire config
  );
  runApp(RecipeReciveApp());
}

class RecipeReciveApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'RecipeRecive',
      theme: ThemeData(
        primarySwatch: Colors.orange,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: MainNavigator(),
    );
  }
}

class MainNavigator extends StatefulWidget {
  @override
  _MainNavigatorState createState() => _MainNavigatorState();
}

class _MainNavigatorState extends State<MainNavigator> {
  int _currentIndex = 0;
  final List<Widget> _screens = [HomeScreen(), RecipeListScreen()];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _screens[_currentIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) => setState(() => _currentIndex = index),
        selectedItemColor: Colors.orange,
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Inicio'),
          BottomNavigationBarItem(icon: Icon(Icons.book), label: 'Recetario'),
        ],
      ),
    );
  }
}
```

---

¿Te gustaría que profundicemos ahora en la creación de las plantillas para la habilidad de **Scraping** o prefieres hacer pruebas de compilación primero?
