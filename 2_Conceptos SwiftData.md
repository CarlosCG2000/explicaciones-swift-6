## INDICE
¿Qué es Core Data?
¿Qué es SwiftData?
1_SwiftData vs Core Data

2_SwiftData --> Módelo compartido (contenedor, contexto)
    2.1_Contenedor en SwiftData
    2.2_Contexto en SwiftData

Ventajas del Modelo Compartido
Comparativa entre Modelo Compartido y Core Data

ANÁLISIS DE LA `APP` en `SwiftData`
    1. `ModelData.swift` (Model)
    Añadir funciones en el `ModelData.swift`
    2. `TaskSwiftDataApp.swift` (ModelContainer)
    3. `SampleData.swift` (ModelContainer en Preview)
        Añadir funciones en el `TaskSwiftDataApp.swift` y `SampleData.swift`
    4. `ContentView.swift` (Vista)
        Posibles Mejoras

¿Se puede combinar este diseño de `SwiftData` con el diseño de `Clean Arquitectura` junto `MVVM`? En este proyecto por ejemplo
¿Si tengo `dos modelos` uno para personajes y otro para citas para llamar con `Query` y acceder al `BD` como se a que `entidad de datos` estoy llamando?

## 1_`SwiftData` vs `Core Data`

### ¿Qué es Core Data?
`Core Data` es un `framework maduro` y `robusto de Apple` diseñado para administrar el modelo de datos en aplicaciones iOS, macOS, watchOS y tvOS. Ofrece funcionalidades avanzadas como `persistencia de datos`, `gestión del ciclo de vida de objetos`, `relaciones entre entidades` y `consultas avanzadas`.

### ¿Qué es SwiftData?
`SwiftData` es una `nueva API` introducida por Apple en WWDC `2023` como una alternativa más `moderna y simplificada a Core Data`. Está diseñada para integrarse mejor con `Swift y SwiftUI`, aprovechando las características del lenguaje Swift (como `@Observable` y el `protocolo Codable`) para facilitar el trabajo con `datos persistentes`.

# SwiftData vs Core Data
| Aspecto                       | SwiftData                                                                                   | Core Data                                                                                          |
|-------------------------------|---------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| **Lenguaje y sintaxis**  | Diseñado para Swift puro con una sintaxis `moderna` y `declarativa`.|  Usa una sintaxis más `antigua` basada en `Objective-C`, aunque funciona en Swift.    |
| **Modelo de datos**      | Define las entidades como estructuras o clases directamente en código con anotaciones como `@Model`. | Requiere un archivo de modelo (`.xcdatamodeld`) y configuraciones manuales en Xcode.   |
| **Compatibilidad con SwiftUI**| Integración nativa con SwiftUI y anotaciones como `@Query` para observar datos en tiempo real.| Necesita integraciones manuales con `@FetchRequest` y `NSFetchedResultsController`.               |
| **Complejidad**               | Más simple de usar y aprender, con `configuraciones automáticas`.                              | Requiere más configuración inicial y gestión manual del contexto.                                 |
| **Persistencia**              | Usa un contenedor y contexto simplificados bajo el capó.                                     | Gestiona explícitamente el contenedor (`NSPersistentContainer`) y los contextos (`NSManagedObjectContext`). |
| **Relaciones**                | Implementa relaciones de forma `declarativa` usando propiedades.                              | Requiere configurarlas `explícitamente` en el modelo gráfico o programático.                        |
| **Concurrencia**              | Diseñado para manejar `concurrencia automáticamente`.                                         | Requiere gestionar `concurrencia manualmente` (contextos privados).                                 |
| **Compatibilidad**            | Disponible desde iOS 17, macOS 14, tvOS 17 y watchOS 10.                                    | Compatible con versiones anteriores de iOS, macOS, tvOS y watchOS.                                |
| **Flexibilidad avanzada**     | No tan avanzado como `Core Data` para proyectos muy complejos o legacy.                       | Mayor flexibilidad para proyectos empresariales de gran escala.                                   |

---

## 2_SwiftData --> Módelo compartido (contenedor, contexto)
En `SwiftData`, el concepto de `modelo compartido` implica un diseño en el que el `contenedor de datos` y su `contexto` se comparten fácilmente en toda la aplicación, especialmente con `SwiftUI`. Esto `simplifica` enormemente el uso y la administración de `datos persistentes`.

### 2.1_Contenedor en SwiftData
El `contenedor (ModelContainer)` es el equivalente simplificado del `NSPersistentContainer` de `Core Data`. Se encarga de administrar el `almacenamiento de datos`, incluidos los `modelos` y las configuraciones de `persistencia`.

```js
import SwiftData

@main
struct MyApp: App {
    @StateObject private var container = ModelContainer(for: [MyModel.self]) // Se pasa el modelo que deseas persistir.

    var body: some Scene {
        WindowGroup {
            ContentView()
                .modelContainer(container) // Asocia el contenedor al entorno SwiftUI.
        }
    }
}
```

### 2.2_Contexto en SwiftData
En SwiftData, el `contexto` es gestionado de `forma automática` por el framework. No necesitas manejarlo manualmente como en `Core Data`. Las `vistas en SwiftUI` pueden acceder al `contexto` a través de anotaciones como `@Query`.

```js
import SwiftData
import SwiftUI

@Model
class MyModel {
    var title: String
    var completed: Bool

    init(title: String, completed: Bool = false) {
        self.title = title
        self.completed = completed
    }
}

struct ContentView: View {
    @Query var items: [MyModel] // Consulta automática de objetos MyModel

    var body: some View {
        List(items) { item in
            Text(item.title)
        }
    }
}
```

### Ventajas del Modelo Compartido
• `Simplificación de Configuración`: No necesitas preocuparte por inicializar o propagar manualmente el contexto.
• `Integración Directa con SwiftUI`: `@Query` hace que observar cambios en los datos sea trivial.
• `Concurrencia Automática`: SwiftData se encarga de gestionar múltiples hilos de forma segura.

### Comparativa entre Modelo Compartido y Core Data

| Aspecto                | SwiftData                                    | Core Data                                   |
|------------------------|----------------------------------------------|--------------------------------------------|
| **Inicialización**    | Usa un `ModelContainer` con modelos declarados directamente en código. | Requiere un `NSPersistentContainer` configurado manualmente. |
| **Uso en vistas**      | Anotaciones declarativas como `@Query`.      | Uso de `@FetchRequest` o `NSFetchedResultsController`. |
| **Propagación**        | Usa `.modelContainer()` para compartir el contenedor en el entorno. | Usa `.environment(\.managedObjectContext)` manualmente. |
| **Observación**        | Datos reactivos automáticos con `@Observable` y `@Query`. | Observación manual mediante `controladores` de resultados. |

## ANÁLISIS DE LA `APP` en `SwiftData`

### 1. `ModelData.swift` (Model):

- 1. La anotación `@Model` en la `clase Tareas` indica que esta clase es una entidad del modelo de datos en `SwiftData`. (esta anotación hace que `SwiftData` gestione automáticamente la persistencia de la entidad en la base de datos.)

Ventajas:
•	No necesitas configurar manualmente un modelo en un archivo `.xcdatamodeld`.
•	El modelo es `declarativo` y está integrado con Swift de `forma nativa`.

- 2. `Propiedades` del `Modelo`.
• `@Attribute(.unique)` en `id`: especifica que el atributo id debe ser único en la base de datos. Esto garantiza que cada tarea tenga `un identificador único (UUID)` para evitar duplicados. Es una excelente práctica para mantener la integridad de los datos.
• `nombre` y `descripcion`: son atributos `simples` para almacenar `cadenas`.
• `fecha`: un atributo de tipo `Date` que puede ser útil para `ordenar` o `filtrar` tareas según su programación.
• `estado`: es un atributo que utiliza un tipo `enumerado personalizado (EstadoTarea)`. Esto permite restringir los valores posibles a opciones predefinidas como `“Pendiente”`, `“En Progreso”`, y `“Completada”`.

3. Enumeración `EstadoTarea`.
• Declaración del tipo enumerado: `EstadoTarea` es una enumeración con casos específicos, y cada caso tiene una representación en cadena.
Al ser `CaseIterable`, permite `iterar automáticamente` sobre todos sus valores, lo cual es útil para crear `listas desplegables o filtros` en la interfaz de usuario.
• Conformidad con `Codable`: la conformidad con Codable asegura que el valor del estado se puede `serializar/deserializar` fácilmente, lo que es crucial si decides `sincronizar los datos con un servidor` o `exportarlos`.
• Conformidad con `Identifiable`: la propiedad `id` de cada `caso enum` facilita su uso directo en `SwiftUI`, por ejemplo, en vistas como `listas o menús`.

4. `Constructor personalizado`.
• El constructor permite `inicializar` todos los valores al crear una `instancia de Tareas`.
• Esto es especialmente útil para inicializar tareas con `datos predeterminados` o provenientes de la `entrada del usuario`.

#### Añadir funciones en el `ModelData.swift`
1. Agregar `relaciones entre entidades` (si aplicable): si tienes otras entidades relacionadas (por ejemplo, un modelo Usuario), puedes usar relaciones entre modelos declarando propiedades como:
```js
    @Relationship var usuario: Usuario
```

2. `Optimización` con `índices`: si necesitas realizar `búsquedas frecuentes` en ciertos atributos (como nombre o estado), podrías considerar agregar `índices declarativos` en `SwiftData`.
```js
@Attribute(.indexed) var nombre: String
```

3. `Extensiones` para funcionalidades `adicionales`: considera agregar `métodos` o `extensiones` a `Tareas` para realizar operaciones comunes.
```js
extension Tareas {
    var isOverdue: Bool {
        return fecha < Date()
    }
}
```

### 2. `TaskSwiftDataApp.swift` (ModelContainer)

1. Modelo compartido (`sharedModelContainer`)

• Declaración del `esquema`: defines el `esquema para el contenedor`, indicando que el modelo `Tareas` es la única entidad que se debe almacenar.

```js
    let schema = Schema([Tareas.self])
```

• `Configuración` del `contenedor de modelo`:
- `isStoredInMemoryOnly`: `false` significa que los datos se persistirán en el `almacenamiento local`, lo que permite que la aplicación conserve los datos entre `sesiones`.
- Es una buena configuración para `producción`, ya que almacena los datos de manera `permanente`.

```js
let modelConfiguration = ModelConfiguration(schema: schema, isStoredInMemoryOnly: false)
```

• Manejo de `errores` al `inicializar` el `contenedor`:
- Esta es una práctica recomendada: `manejar los errores` durante la `inicialización` y detener la ejecución si el contenedor no se puede crear.
- La `impresión del error` en la consola ayuda al depurador a identificar problemas en tiempo de desarrollo.

```js
do {
    return try ModelContainer(for: schema, configurations: [modelConfiguration])
} catch {
    print("Error al crear el ModelContainer: \(error)")
    fatalError("No se pudo crear el ModelContainer: \(error.localizedDescription)")
}
```

2. Vinculación del contenedor con la escena principal:
• `modelContainer` vincula el `contenedor al entorno`, permitiendo que cualquier vista que lo necesite pueda acceder al `contexto principal` (e.g., insertar, actualizar o consultar datos).

### 3. `SampleData.swift` (ModelContainer en Preview)
Este archivo es una implementación práctica para `pruebas` y `vistas previas`. La separación del código de muestra es ideal, ya que te permite probar y previsualizar la aplicación sin interferir con la lógica principal de producción.

1. Creación de un `contenedor de modelo` en memoria:
• `isStoredInMemoryOnly`: `true` asegura que los datos `no se persistan`, lo que es ideal para `pruebas` o `vistas previas`.
• Es una excelente práctica mantener el `entorno de prueba aislado` del entorno de producción.
```js
let modelConfiguration = ModelConfiguration(schema: schema, isStoredInMemoryOnly: true)
```

2. `Inserción` de datos de muestra e inserción directa en el `mainContext`:
Esto asegura que `los datos de muestra` estén disponibles para su uso inmediato en las vistas.
```js
let tarea1 = Tareas(id: UUID(),
                    nombre: "Realizar tareas de forma autodidacta",
                    descripcion: "Tarea con respecto a mejora y aprendizaje para prepararse al máster MIMO en la asignatura de iOS",
                    fecha: .now,
                    estado: .enProgreso)

container.mainContext.insert(tarea1)
```

3. Uso en `previsualizaciones`:
Permite usar el `modificador` de muestra en las vistas previas para proporcionar `datos predefinidos`. Esto es útil para probar la UI con datos representativos.

```js
extension PreviewTrait where T == Preview.ViewTraits {
    @MainActor static var sampleData: Self = .modifier(SampleData()) 
}
```

#### Añadir funciones en el `TaskSwiftDataApp.swift` y `SampleData.swift`

1. Uso de `actores` en `concurrencia`:
• Podrías considerar usar actores para proteger el `acceso al contenedor` y al `contexto` en aplicaciones con `múltiples hilos`. Por ejemplo:

+ ¿Como implementaria el punto de añadir funciones `1. Uso de actores en concurrencia.` en los `ModelContainer`?
Con este enfoque, todas las operaciones sobre el `ModelContainer` y el `mainContext` están centralizadas y protegidas por el `actor DataManager`, garantizando la seguridad en concurrencia.

```js
import Foundation
import SwiftData

actor DataManager {
    private let container: ModelContainer

    init() throws {
        let schema = Schema([Tareas.self])
        let modelConfiguration = ModelConfiguration(schema: schema, isStoredInMemoryOnly: false)
        self.container = try ModelContainer(for: schema, configurations: [modelConfiguration])
    }

    /// Inserta una tarea en el contexto principal
    func insert(tarea: Tareas) async throws {
        try await container.mainContext.insert(tarea)
    }

    /// Consulta todas las tareas desde el contexto principal
    func fetchAllTareas() async throws -> [Tareas] {
        let fetchDescriptor = FetchDescriptor<Tareas>()
        return try await container.mainContext.fetch(fetchDescriptor)
    }

    /// Elimina una tarea del contexto principal
    func delete(tarea: Tareas) async throws {
        try await container.mainContext.delete(tarea)
    }

    /// Guarda los cambios en el contexto principal
    func save() async throws {
        try await container.mainContext.save()
    }
}
```

Y con el uso del `DataManager` en las vistas o `ViewModels`
```js
import SwiftData
import SwiftUI

@Observable
final class TareasViewModel {
    private let dataManager: DataManager
    var tareas: [Tareas] = []

    init(dataManager: DataManager) {
        self.dataManager = dataManager
        Task {
            await fetchTareas()
        }
    }

    func fetchTareas() async {
        do {
            tareas = try await dataManager.fetchAllTareas()
        } catch {
            print("Error al obtener tareas: \(error)")
        }
    }

    func addTarea(nombre: String, descripcion: String, fecha: Date, estado: EstadoTarea) async {
        let nuevaTarea = Tareas(id: UUID(), nombre: nombre, descripcion: descripcion, fecha: fecha, estado: estado)
        do {
            try await dataManager.insert(tarea: nuevaTarea)
            await fetchTareas() // Refrescamos la lista de tareas
        } catch {
            print("Error al insertar tarea: \(error)")
        }
    }
}
```

2. Separación de `responsabilidades`:
• Actualmente, la `lógica de inicialización` y la `definición del modelo` están en la misma clase. Podrías separar las responsabilidades para que el contenedor de modelo sea manejado por un servicio o gestor dedicado.

+ Creación de un `Servicio` de Modelo
```js
import SwiftData
import Foundation

final class ModelContainerService {
    static let shared = ModelContainerService() // Singleton para compartir una única instancia
    private(set) var container: ModelContainer

    private init() {
        let schema = Schema([Tareas.self])
        let modelConfiguration = ModelConfiguration(schema: schema, isStoredInMemoryOnly: false)

        do {
            self.container = try ModelContainer(for: schema, configurations: [modelConfiguration])
        } catch {
            print("Error al crear el ModelContainer: \(error)")
            fatalError("No se pudo crear el ModelContainer: \(error.localizedDescription)")
        }
    }
}
```

+ Ajuste de la `Aplicación Principal`
En lugar de manejar el `ModelContainer` directamente en `TaskSwiftDataApp`, delegamos esa responsabilidad al servicio.
```js
import SwiftUI
import SwiftData

@main
struct TaskSwiftDataApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(ModelContainerService.shared.container)
    }
}
```

Actualiza el actor de ejemplo anterior para usar el servicio compartido:
```js
actor DataManager {
    private let container: ModelContainer = ModelContainerService.shared.container

    func fetchAllTareas() async throws -> [Tareas] {
        let fetchDescriptor = FetchDescriptor<Tareas>()
        return try await container.mainContext.fetch(fetchDescriptor)
    }

    func insert(tarea: Tareas) async throws {
        try await container.mainContext.insert(tarea)
    }

    func save() async throws {
        try await container.mainContext.save()
    }
}
```

- Beneficios de esta separación
• `Reutilización`: puedes usar el mismo servicio en diferentes partes de la aplicación (e.g., vistas, ViewModels, pruebas).
• `Mantenimiento`: si necesitas cambiar la configuración del contenedor (e.g., agregar modelos adicionales), solo necesitas actualizar el servicio.
• `Legibilidad`: la responsabilidad de configurar el contenedor se separa de la lógica de la aplicación principal.

3. Datos más `dinámicos` en `SampleData`:
• Podrías incluir datos generados `dinámicamente` usando bucles o estructuras de control:

```js
for i in 1...5 {
    let tarea = Tareas(
        id: UUID(),
        nombre: "Tarea \(i)",
        descripcion: "Descripción de la tarea \(i)",
        fecha: Date().addingTimeInterval(Double(i) * 3600),
        estado: .pendiente
    )
    container.mainContext.insert(tarea)
}
```

4. Pruebas `unitarias`:
• Implementa pruebas unitarias para validar que los datos se insertan y se consultan correctamente en el contenedor de modelo. Esto asegura que el modelo funciona como se espera en diferentes configuraciones.

5. `Migraciones`:
• Aunque `SwiftData` maneja automáticamente las `migraciones básicas`, si planeas expandir el esquema, asegúrate de probar cómo maneja los cambios de estructura y datos existentes.

### 4. `ContentView.swift` (Vista)
1. Uso de `@Query`
El uso de la propiedad `@Query` para recuperar tareas es una excelente elección, ya que aprovecha la integración de `SwiftData` con `SwiftUI`. Esto permite que las vistas `reaccionen automáticamente` a los cambios en los datos, lo cual es ideal para un `flujo reactivo`. Sin embargo: si planeas realizar operaciones más complejas (como filtrado o clasificación avanzada), considera personalizar tu consulta:

```js
@Query(sort: \.fecha, order: .reverse) private var tareas: [Tareas]
```

2. `Gestión de Contexto`
El uso de `@Environment(\.modelContext)` para obtener el contexto del modelo es correcto y sigue las mejores prácticas de `SwiftData`. Sin embargo, las operaciones como la eliminación de tareas están acopladas directamente a la vista.
```js
.onDelete { index in
    for i in index {
        context.delete(tareas[i])
    }
}
```

Sugerencia: Podrías abstraer esta `lógica en un servicio` o `función` para mejorar la separación de responsabilidades. Por ejemplo:
```js
func deleteTareas(at offsets: IndexSet) {
    for i in offsets {
        context.delete(tareas[i])
    }
}
```
Y luego llamar
```js
.onDelete(perform: deleteTareas)
```

3. `Gestión del Estado`
El uso de `@State` para manejar `showAdd` está bien implementado y sigue las convenciones de `SwiftUI`:
```js
@State private var showAdd = false
```
Sin embargo, podrías mejorar la experiencia de usuario al `resetear` este `estado automáticamente` cuando se cierra la vista `NewTareaView`. Esto se puede lograr mediante el modificador `.onDisappear` en la vista presentada:
```js
.fullScreenCover(isPresented: $showAdd) {
    NewTareaView()
        .onDisappear {
            showAdd = false
        }
}
```

4. Vista principal (main)
Tu lógica para mostrar las tareas en una `lista es clara y directa`. Además, manejar un estado vacío con `ContentUnavailableView` mejora la experiencia de usuario. Sin embargo, podrías considerar:
• Añadir un `encabezado` o pie de página informativo:

```js
List {
    Section(header: Text("Lista de tareas")) {
        ForEach(tareas) { tarea in
            TareaRow(tarea: tarea)
        }
        .onDelete { index in
            for i in index {
                context.delete(tareas[i])
            }
        }
    }

    Section(footer: Text("Pulsa el botón + para añadir nuevas tareas.")) {}
}
```

5. Estructura de Código
Tu `ContentView` está bien organizada, pero puede beneficiarse de pequeñas mejoras en la separación de responsabilidades:
• Por ejemplo, podrías extraer `ContentUnavailableView` a una función para evitar repetición de código si necesitas personalizar este estado en diferentes vistas:
```js
func emptyStateView() -> some View {
    ContentUnavailableView("No hay tareas",
                           systemImage: "list.bullet.clipboard",
                           description: Text("Aún no existen tareas en la app. Por favor, pulse el + arriba a la derecha para crear una nueva tarea."))
        .padding()
}
```

```js
if tareas.isEmpty {
    emptyStateView()
    Spacer()
} else {
    main
}
```

#### Posibles Mejoras
1. Separación de `Responsabilidades`:
• Actualmente, las operaciones `CRUD` están acopladas a la `vista`. Podrías moverlas a un `servicio o actor` dedicado para mejorar la organización y facilitar futuras expansiones.
2. Uso de `Concurrencia`:
• Aunque tu aplicación actual no parece requerir `concurrencia avanzada`, si en el futuro planeas `cargar datos desde una API` o realizar operaciones pesadas, sería útil usar `actores` para manejar la `concurrencia` y evitar posibles conflictos.
3. `Extensibilidad`:
• Podrías añadir `filtros` (por estado de la tarea) o agrupación (por fecha) para hacer la `lista de tareas más dinámica y organizada`.

```js
    @Query(filter: #Predicate { $0.estado == .pendiente }) var tareasPendientes: [Tareas]
```
4. `Componentización` de la UI:
	•	Si `TareaRow` se usa en múltiples vistas, asegúrate de que sea `modular` y `personalizable`. Por ejemplo:
```js
    struct TareaRow: View {
    let tarea: Tareas

    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Text(tarea.nombre).font(.headline)
                Text(tarea.descripcion).font(.subheadline).foregroundColor(.gray)
            }
            Spacer()
            Text(tarea.estado.rawValue)
                .foregroundColor(tarea.estado == .completada ? .green : .blue)
        }
    }
}
```

## ¿Se puede combinar este diseño de `SwiftData` con el diseño de `Clean Arquitectura` junto `MVVM`? En este proyecto por ejemplo
Sí, puedes combinar `SwiftData` con un diseño basado en `Clean Architecture` y `MVVM` para estructurar tu proyecto de manera modular, escalable y fácil de mantener. La idea principal sería separar:
- las `capas de la aplicación` (`Presentación`, `Dominio` y `Datos`).
- la `capa de persistencia` mientras utilizas `SwiftData`.

Cómo integrar Clean Architecture y MVVM con SwiftData

1. Capas principales
    1. `Capa de Datos` (`Data Layer`): contiene la `lógica de persistencia` y operaciones `CRUD` con `SwiftData`.
	•	Define las entidades de datos (`@Model`) y los `servicios` que interactúan con el `ModelContainer`.

	2.	`Capa de Dominio` (`Domain Layer`): contiene `casos de uso` (Use Cases) que encapsulan la `lógica de negocio`.
	• `No` tiene `dependencias` directas de `la capa de datos` ni de `SwiftData`.
	• Define `entidades de dominio` (pueden ser diferentes de las `entidades de datos` si es necesario).

	3. `Capa de Presentación` (Presentation Layer): implementa el `patrón MVVM`, donde el `ViewModel` interactúa con la `capa de dominio` y expone `datos procesados` a las `vistas`.

2. Estructura de ejemplo para combinar `SwiftData` con `Clean Architecture`

    1. `Capa de Datos`: Persistencia
    Define las `entidades de datos` y la `lógica` para acceder al `modelo` con `SwiftData`.

```js
// Entidad de Datos
import SwiftData

@Model
final class Personaje {
    var id: UUID
    var nombre: String
    var afiliacion: String

    init(id: UUID = UUID(), nombre: String, afiliacion: String) {
        self.id = id
        self.nombre = nombre
        self.afiliacion = afiliacion
    }
}
```
```js
// Servicio de Persistencia
import SwiftData

final class DataRepository {
    private let modelContext: ModelContext

    init(context: ModelContext) {
        self.modelContext = context
    }

    func fetchPersonajes() -> [Personaje] {
        // Consulta directa con SwiftData
        return modelContext.fetch(FetchDescriptor<Personaje>())
    }

    func addPersonaje(nombre: String, afiliacion: String) {
        let nuevoPersonaje = Personaje(nombre: nombre, afiliacion: afiliacion)
        modelContext.insert(nuevoPersonaje)
        saveContext()
    }

    private func saveContext() {
        do {
            try modelContext.save()
        } catch {
            print("Error al guardar el contexto: \(error)")
        }
    }
}
```

   2. `Capa de Dominio`: Casos de Uso
   Define `la lógica de negocio` desacoplada de `la capa de datos`.

```js
final class ObtenerPersonajesUseCase {
    private let repository: DataRepository

    init(repository: DataRepository) {
        self.repository = repository
    }

    func execute() -> [Personaje] {
        return repository.fetchPersonajes()
    }
}

final class AgregarPersonajeUseCase {
    private let repository: DataRepository

    init(repository: DataRepository) {
        self.repository = repository
    }

    func execute(nombre: String, afiliacion: String) {
        repository.addPersonaje(nombre: nombre, afiliacion: afiliacion)
    }
}
```

   3. `Capa de Presentación`: MVVM
   El `ViewModel` interactúa con los `casos de uso` para exponer` datos procesados` a la `vista`.

[ Si necesitas soporte para versiones anteriores de Swift o iOS (antes de Swift 5.9 o iOS 17), tendrás que seguir usando `@Published` para que las propiedades sean observables. Además, `@Published` es útil fuera del ecosistema de `SwiftUI`, como en contextos donde las anotaciones como `@Observable` no están disponibles. ]

```js
import SwiftUI

@Observable
final class PersonajesViewModel {
    private let obtenerPersonajesUseCase: ObtenerPersonajesUseCase
    private let agregarPersonajeUseCase: AgregarPersonajeUseCase

    var personajes: [Personaje] = [] // No hace falta @Published.

    init(obtenerPersonajesUseCase: ObtenerPersonajesUseCase, agregarPersonajeUseCase: AgregarPersonajeUseCase) {
        self.obtenerPersonajesUseCase = obtenerPersonajesUseCase
        self.agregarPersonajeUseCase = agregarPersonajeUseCase
        loadPersonajes()
    }

    func loadPersonajes() {
        personajes = obtenerPersonajesUseCase.execute()
    }

    func addPersonaje(nombre: String, afiliacion: String) {
        agregarPersonajeUseCase.execute(nombre: nombre, afiliacion: afiliacion)
        loadPersonajes() // Recargar los datos después de agregar
    }
}
```

Las `vistas` utilizan el `ViewModel` para mostrar `datos` y reaccionar a los `cambios`.
```js
struct PersonajesView: View {
    @StateObject private var viewModel = PersonajesViewModel(
        obtenerPersonajesUseCase: ObtenerPersonajesUseCase(repository: DataRepository(context: ModelContext.current)),
        agregarPersonajeUseCase: AgregarPersonajeUseCase(repository: DataRepository(context: ModelContext.current))
    )

    @State private var nombre: String = ""
    @State private var afiliacion: String = ""

    var body: some View {
        NavigationStack {
            List(viewModel.personajes) { personaje in
                VStack(alignment: .leading) {
                    Text(personaje.nombre).font(.headline)
                    Text(personaje.afiliacion).font(.subheadline).foregroundColor(.gray)
                }
            }
            .navigationTitle("Personajes")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Agregar") {
                        viewModel.addPersonaje(nombre: nombre, afiliacion: afiliacion)
                    }
                }
            }
        }
    }
}
```

## ¿Si tengo `dos modelos` uno para personajes y otro para citas para llamar con `Query` y acceder al `BD` como se a que `entidad de datos` estoy llamando?
Cuando usas `@Query` en una `vista`, esta se asocia a una `entidad específica` definida en tu modelo de datos (`@Model`). Por ejemplo:
```js
@Query private var personajes: [Personaje]
@Query private var citas: [Cita]
```

En este caso:
1. `@Query` `automáticamente` sabe que debe observar los datos de la entidad `Personaje` o `Cita` (según lo declarado).
2. Si tienes `múltiples` entidades en tu `ModelContainer`, `SwiftData` diferencia `automáticamente` las consultas en función del `tipo` especificado ([Personaje] o [Cita]).
3. `Filtrado o clasificación`: Puedes personalizar las consultas usando `predicados` o `descriptores` de orden.

```js
@Query(filter: #Predicate { $0.afiliacion == "Jedi" }) private var personajesJedi: [Personaje]
```

Ventajas de combinar `Clean Architecture` y `SwiftData`
1. `Modularidad`: Cada capa tiene responsabilidades claramente definidas.
2. `Reutilización`: Los casos de uso pueden ser reutilizados por diferentes vistas o componentes.
3. `Testabilidad`: Al desacoplar la lógica de negocio y la persistencia, puedes probar las capas de dominio y presentación de forma independiente.
4. `Escalabilidad`: La estructura facilita añadir nuevas entidades, funciones y vistas sin impactar negativamente en el código existente.
