

##  1_SWIFT BASADO EN `PROTOCOLOS`
se basa siempre en `contratos` que tu le dices al sistema que tiene que `cumplir` e incluso puede llegar a incorporar ciertas funcionalidades concretas.

## 2_Swift UI funciona con el concepto de `MVVM` ❌
Se cogen los `eventos` y se traspasaban de un lado a otro, es decir, por un lado tengo la parte del código que me da la `funcionalidad` de la aplicación (lo que seria la `lógica de negocio`) y por otro lado seria la `vista` que es lo que ve el `usuario`. Entonces la `conexión` entre la lógica de usuario y la vista estaba realizada.

### ¿Qué es MVVM?
MVVM es un patrón de diseño arquitectónico que separa la `lógica de negocio`, `la interfaz de usuario` y la `conexión entre ambas` en tres componentes principales:

+------------+         +------------+         +------------+
|   Model    | <--->   | ViewModel  | <--->   |    View     |
| (Datos)    |         | (Lógica)   |         | (Interfaz)  |
+------------+         +------------+         +------------+

Uso de MVVM:
| **Ventajas**                                   | **Desventajas**                                       |
|-----------------------------------------------|------------------------------------------------------|
| Separación clara entre lógica y presentación  | Puede ser excesivo para aplicaciones simples         |
| Mejor mantenimiento y escalabilidad           | La implementación inicial puede ser más compleja     |
| Facilita la reutilización de código           | Puede haber sobrecarga en la sincronización entre View y ViewModel |
| Compatible con pruebas unitarias              | Requiere aprender conceptos como `@StateObject`, `@Observable`, etc. |

1.	`Model (Modelo)`: Representa los `datos` y la `lógica de negocio`.

El modelo es el componente más simple. Representa los datos de tu aplicación y encapsula la `lógica de negocio` relacionada. Es completamente independiente de la interfaz de usuario.

```js
struct Task: Identifiable {
    let id = UUID()
    var title: String
    var isCompleted: Bool
}
```

2.	`ViewModel (Vista-Modelo)`: Actúa como un puente entre el modelo y la vista. Contiene la `lógica de presentación` y `expone los datos` de manera que la vista pueda observarlos y reaccionar a los cambios.
El ViewModel es el intermediario entre el `modelo` y la `vista`. Es responsable de transformar los datos del modelo en algo que la vista pueda entender, y viceversa. También gestiona la `lógica de presentación` y actúa como una fuente de verdad observable.
- Responsabilidades principales del ViewModel:
    • Gestionar el estado de la vista.
    • Transformar datos del modelo en formato listo para la vista.
    • Manejar acciones de usuario desde la vista y actualizar el modelo.

```js
@Observable
class TaskViewModel {
    // Estado observable que las vistas pueden observar
    var tasks: [Task] = []

    // Función para agregar una nueva tarea
    func addTask(title: String) {
        let newTask = Task(title: title, isCompleted: false)
        tasks.append(newTask)
    }

    // Función para marcar una tarea como completada
    func toggleTaskCompletion(_ task: Task) {
        if let index = tasks.firstIndex(where: { $0.id == task.id }) {
            tasks[index].isCompleted.toggle()
        }
    }
}
```

3.	`View (Vista)`: La `interfaz de usuario`. En SwiftUI, las vistas están directamente vinculadas al `ViewModel` mediante un `enlace reactivo`.
La vista es el componente encargado de mostrar la interfaz de usuario. En SwiftUI, las vistas son declarativas y reactivas, lo que significa que se actualizan automáticamente cuando los datos en el ViewModel cambian.
- Responsabilidades principales de la Vista:
	• Mostrar los datos proporcionados por el ViewModel.
	• Interactuar con el usuario y notificar al ViewModel sobre las acciones.

### ¿Puedo usar @State en lugar de @StateObject para vincular el ViewModel?
Si usas @State en lugar de @StateObject, se recreará el ViewModel cada vez que se reconstruya la vista, lo que significa que perderás el estado actual.
La razón principal es que @State está diseñado para manejar datos simples y locales dentro de la vista. Sin embargo, un ViewModel es un objeto de referencia que necesita mantenerse vivo mientras la vista exista, lo cual es la razón por la que usamos @StateObject.

```js
struct TaskListView: View {
    @StateObject private var viewModel = TaskViewModel() // ViewModel vinculado correctamente

    var body: some View {
        NavigationView {
            List {
                ForEach(viewModel.tasks) { task in
                    HStack {
                        Text(task.title)
                            .strikethrough(task.isCompleted, color: .black)
                        Spacer()
                        Button(action: {
                            viewModel.toggleTaskCompletion(task)
                        }) {
                            Image(systemName: task.isCompleted ? "checkmark.circle.fill" : "circle")
                        }
                    }
                }
            }
            .navigationTitle("Tareas")
            .toolbar {
                Button(action: {
                    viewModel.addTask(title: "Nueva Tarea")
                }) {
                    Image(systemName: "plus")
                }
            }
        }
    }
}
```

### ¿Por qué es importante MVVM en SwiftUI?
	1.	Reactividad nativa:
	•	SwiftUI es un framework reactivo, lo que significa que las vistas reaccionan automáticamente a los cambios de estado en el ViewModel. Esto se alinea perfectamente con el patrón MVVM.
	2.	Separación de responsabilidades:
	•	MVVM organiza el código, separando la lógica de negocio (ViewModel) de la lógica de interfaz (View). Esto hace que el código sea más fácil de leer, mantener y probar.
	3.	Escalabilidad:
	•	En aplicaciones más grandes, MVVM facilita la adición de nuevas funcionalidades, ya que cada vista tiene su propio ViewModel y modelo.
	4.	Facilita las pruebas:
	•	Puedes probar la lógica de negocio y el estado en el ViewModel sin necesidad de cargar la interfaz de usuario.

### ¿Cuándo usar MVVM?
Usa MVVM cuando:
	•	Estás desarrollando una aplicación con múltiples vistas y lógica de negocio compleja.
	•	Necesitas mantener una arquitectura modular y escalable.
	•	Quieres aprovechar las características de reactividad de SwiftUI.

En proyectos más pequeños, es posible que MVVM no sea necesario y puedas simplificar tu arquitectura.

### ¿Puedo tener varias ViewModels para varias vistas en MVVM?
Sí, puedes y deberías tener una ViewModel diferente para cada vista que requiera manejar lógica específica o estado independiente. Esto permite mantener la separación de responsabilidades en MVVM. Si varias vistas comparten la misma lógica o datos, puedes reutilizar una única ViewModel.

Por ejemplo:
	•	Si tienes una pantalla de lista de tareas (TaskListView), usarías una TaskViewModel.
	•	Si tienes una pantalla de detalle para una tarea (TaskDetailView), podrías usar una TaskDetailViewModel.

Esto asegura que cada vista tenga solo la lógica y los datos que necesita.

### ¿Uso de actores en lógica de negocio (Modelo) o lógica de presentación (ViewModel)?

| **Escenario**                                | **Usar Actor**                   | **Usar Clase**                  | **Usar Struct**                   |
|----------------------------------------------|-----------------------------------|----------------------------------|-----------------------------------|
| Datos compartidos en concurrencia            | ✅ Sí                             | ❌ No                            | ❌ No                             |
| Modelo de datos puro sin lógica compleja     | ❌ No                             | ❌ No                            | ✅ Sí                             |
| Lógica de presentación en ViewModel          | ✅ Si hay concurrencia            | ✅ Si no hay concurrencia        | ❌ No                             |
| Referencias compartidas                      | ✅ Sí (con concurrencia segura)   | ✅ Sí                            | ❌ No                             |
| Rendimiento (bajo overhead)                  | ❌ No (mayor costo por aislamiento)| ✅ Más eficiente que un actor    | ✅ Más eficiente que ambos        |


Uso de actores en lógica de negocio (Modelo):
	•	Si tu modelo necesita manejar datos compartidos entre varias partes de la aplicación, y esos datos pueden ser accedidos desde múltiples hilos.
	•	Ejemplo: Una base de datos en memoria o un caché que debe ser concurrentemente seguro.

```js
actor TaskManager {
    private var tasks: [Task] = []

    func addTask(_ task: Task) {
        tasks.append(task)
    }

    func getTasks() -> [Task] {
        return tasks
    }
}
```

Uso de actores en lógica de presentación (ViewModel):
	•	Cuando necesitas manejar estados o eventos concurrentes dentro del ViewModel.
	•	Ejemplo: Un ViewModel que interactúa con varias fuentes de datos asíncronas.

```js
@Observable
actor TaskViewModel {
    var tasks: [Task] = []

    func addTask(title: String) {
        let newTask = Task(title: title, isCompleted: false)
        tasks.append(newTask)
    }

    func toggleTaskCompletion(_ task: Task) {
        if let index = tasks.firstIndex(where: { $0.id == task.id }) {
            tasks[index].isCompleted.toggle()
        }
    }
}
```

Cuándo usar clases o estructuras:
	1.	Usar clases:
	•	Cuando necesitas un objeto de referencia que pueda ser compartido entre varias partes de la aplicación.
	•	Útil en entornos no concurrentes o donde puedes garantizar manualmente la seguridad en concurrencia.
	2.	Usar estructuras:
	•	Cuando tus datos son de valor y no necesitas compartirlos entre múltiples vistas o partes de la aplicación.
	•	Son ideales para `modelos de datos puros` y se benefician de su bajo overhead.


# DIFERENCIA ENTRE `UIKIT` CON `MVC` Y `SWIFT UI` CON `MVVM`.
Comparar UIKit con MVC y SwiftUI con MVVM es esencial para entender cómo ambas arquitecturas y frameworks abordan la construcción de interfaces de usuario y la gestión de lógica en aplicaciones iOS. A continuación, te detallo las principales diferencias y cuál puede ser más conveniente según tu caso.

UIKit con MVC

Modelo-Vista-Controlador (MVC) es un patrón arquitectónico tradicional utilizado en UIKit. Su flujo es lineal:
	1.	Modelo (Model): Representa los datos y la lógica de negocio.
	2.	Vista (View): Es la interfaz gráfica que ve el usuario.
	3.	Controlador (Controller): Coordina las interacciones entre la vista y el modelo.

Ventajas:
	•	Familiaridad: Ha sido el estándar en iOS durante años.
	•	Soporte extendido: Compatible con una amplia gama de herramientas y frameworks.
	•	Acceso detallado: Permite un control más granular sobre cada aspecto de la interfaz.

Desventajas:
	•	Massive View Controller: Es común que los controladores crezcan desproporcionadamente al manejar lógica de presentación y eventos de usuario.
	•	Baja reactividad: Requiere actualizar manualmente la interfaz al cambiar datos.
	•	Dificultad para manejar la concurrencia: Los desarrolladores deben asegurarse de ejecutar tareas en el hilo principal al interactuar con la interfaz.

SwiftUI con MVVM

Modelo-Vista-ViewModel (MVVM) se alinea perfectamente con la naturaleza declarativa y reactiva de SwiftUI.
	1.	Modelo (Model): Datos y lógica de negocio (igual que en MVC).
	2.	ViewModel: Traduce el modelo a algo que la vista pueda mostrar. Maneja el estado y las interacciones de forma reactiva.
	3.	Vista (View): Es una declaración de cómo debería ser la interfaz. Se actualiza automáticamente cuando cambian los datos observados.

Ventajas:
	•	Reactividad nativa: Las vistas se actualizan automáticamente cuando los datos cambian.
	•	Simplicidad: Menos código repetitivo gracias al enfoque declarativo.
	•	Mejor manejo de estado: Propiedades como @State, @Binding y @ObservableObject facilitan el control del flujo de datos.
	•	Separación de responsabilidades: La lógica de presentación (ViewModel) está separada de la vista, lo que mejora la mantenibilidad.

Desventajas:
	•	Compatibilidad limitada: Requiere iOS 13 o superior.
	•	Curva de aprendizaje: La programación declarativa y MVVM pueden ser confusas para los desarrolladores acostumbrados a UIKit.

¿Cuál es más conveniente?

La elección depende del tipo de proyecto, el equipo y las necesidades:

Cuándo elegir UIKit con MVC:
	1.	Compatibilidad: Si necesitas compatibilidad con versiones antiguas de iOS (< iOS 13).
	2.	Equipo experimentado: Si el equipo ya está familiarizado con UIKit y MVC.
	3.	Proyectos complejos con personalización detallada: UIKit ofrece más control y flexibilidad para personalizar animaciones, transiciones y vistas.

Cuándo elegir SwiftUI con MVVM:
	1.	Proyectos nuevos: Si el proyecto tiene como objetivo dispositivos modernos (iOS 13 o superior), SwiftUI es ideal por su simplicidad y escalabilidad.
	2.	Prototipos rápidos: SwiftUI es perfecto para crear interfaces rápidamente.
	3.	Mantenimiento a largo plazo: La separación de lógica en MVVM y la reactividad de SwiftUI facilitan la evolución del código con el tiempo.
	4.	Equipos pequeños: SwiftUI reduce la cantidad de código necesario y simplifica el flujo de trabajo.

| **Aspecto**                | **UIKit con MVC**                                                                                       | **SwiftUI con MVVM**                                                                                 |
|-----------------------------|-------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| **Arquitectura**            | Basado en Model-View-Controller (MVC): La vista y el controlador suelen estar fuertemente acoplados.  | Basado en Model-View-ViewModel (MVVM): La vista y el ViewModel están conectados mediante enlaces reactivos. |
| **Interacción**             | Manual: Los datos se transfieren explícitamente entre el modelo, el controlador y la vista.           | Reactivo: Los datos se actualizan automáticamente gracias a las propiedades observables y la reactividad de SwiftUI. |
| **Framework**               | UIKit es imperativo: defines explícitamente cada paso para construir y actualizar la interfaz.        | SwiftUI es declarativo: describes cómo debería ser la interfaz, y SwiftUI se encarga de las actualizaciones. |
| **Complejidad del código**  | Puede haber clases de controlador muy grandes (“Massive View Controller”) debido al acoplamiento de lógica de vista y presentación. | La lógica de la vista se abstrae en el ViewModel, lo que da lugar a vistas más simples y código más organizado. |
| **Concurrencia**            | Necesita manualmente gestionar los hilos (por ejemplo, usar `DispatchQueue.main.async` para actualizar la interfaz). | La concurrencia es más sencilla, gracias al soporte nativo para `@State`, `@ObservableObject` y el uso de `async/await`. |
| **Reutilización de código** | La vista y la lógica están entrelazadas, lo que dificulta la reutilización de código en diferentes vistas. | La vista y la lógica están separadas; el ViewModel se puede reutilizar fácilmente entre diferentes vistas. |
| **Manejo del estado**       | No existe un sistema integrado para manejar el estado de forma reactiva. Necesitas manualmente mantener y sincronizar el estado. | Manejo de estado incorporado mediante `@State`, `@Binding`, `@ObservableObject` y otros. |
| **Compatibilidad**          | Soportado en versiones más antiguas de iOS. Requiere UIKit y es más maduro con soporte extendido.     | SwiftUI requiere iOS 13 o superior y aún está en constante evolución. |
| **Curva de aprendizaje**    | Más fácil de aprender inicialmente, pero puede complicarse a medida que el proyecto crece.            | Puede ser más complejo al principio (por la arquitectura MVVM y programación declarativa), pero más fácil de mantener en proyectos grandes. |
| **Desempeño**               | Más optimizado para aplicaciones con mucha personalización visual o comportamientos complejos.        | Más eficiente para prototipos rápidos y aplicaciones que dependen de cambios de estado reactivos. |
| **Pruebas unitarias**       | Más difícil de probar debido al acoplamiento de lógica y vistas en los controladores.                 | Más fácil de probar porque la lógica está en el ViewModel, independiente de la vista. |

## 3_PASO PARCIAL ACTUAL DE LIBRERÍA DE `COMBINE` A LA LIBRERÍA DE `Swift Macros`

- Librería Combine
Combine es un framework de Apple diseñado para trabajar con programación reactiva. Se introdujo en iOS 13 y macOS 10.15 para manejar eventos asincrónicos y flujos de datos de forma declarativa. La principal característica de Combine es que permite gestionar suscripciones, transformaciones y combinaciones de eventos.

Características principales de Combine
• Publicadores (Publishers): Emisores de datos que pueden transmitir valores a lo largo del tiempo.
• Suscriptores (Subscribers): Entidades que escuchan los datos emitidos por los publicadores.
• Operadores: Permiten transformar, filtrar y combinar flujos de datos de manera declarativa (por ejemplo, map, filter, combineLatest).
• Cadenas de procesamiento: Se pueden construir cadenas reactivas complejas para transformar datos entre el publicador y el suscriptor.

```js
import Combine

class ExampleViewModel: ObservableObject {
    @Published var searchQuery: String = ""
    @Published var searchResults: [String] = []

    private var cancellables = Set<AnyCancellable>()

    init() {
        $searchQuery
            .debounce(for: .seconds(0.5), scheduler: RunLoop.main) // Espera medio segundo
            .removeDuplicates() // Evita procesar valores repetidos
            .sink { [weak self] query in
                self?.searchResults = self?.performSearch(for: query) ?? []
            }
            .store(in: &cancellables)
    }

    private func performSearch(for query: String) -> [String] {
        // Lógica para realizar la búsqueda
        return ["Resultado 1", "Resultado 2"]
    }
}
```

Ventajas de Combine
	•	Proporciona un enfoque declarativo para trabajar con flujos de datos y eventos.
	•	Soporte integrado para @Published, facilitando la reactividad en ObservableObject.
	•	Funciona bien con arquitecturas como MVVM y frameworks como SwiftUI.

Desventajas de Combine
	•	Requiere un conocimiento sólido de la programación reactiva, lo que puede ser complicado para principiantes.
	•	Puede volverse excesivo en aplicaciones simples.
	•	El manejo de cancelaciones puede ser tedioso (requiere almacenar los cancellables).

- Librería Swift Macros
Con la llegada de Swift Macros en Swift 5.9 (iOS 17, macOS Sonoma), Apple introdujo un mecanismo que permite generar automáticamente código repetitivo o boilerplate durante el tiempo de compilación. Las macros están diseñadas para hacer el código más conciso y legible, especialmente en escenarios donde antes se requería mucho código adicional, como la implementación manual de protocolos, propiedades, o configuraciones.

Características principales de Swift Macros
• Macros de Declaración: Permiten definir propiedades, métodos o configuraciones automáticamente. Por ejemplo:
```js
@Observable
struct Task {
    var title: String
    var isCompleted: Bool
}
```

Gracias a @Observable, se elimina la necesidad de usar manualmente el protocolo ObservableObject y de marcar cada propiedad con @Published. Las vistas que usen este ViewModel recibirán automáticamente las actualizaciones.

Transición parcial de Combine hacia Swift Macros

Con la introducción de macros como @Observable, Apple ha comenzado a reemplazar parte de la funcionalidad de Combine, especialmente en lo relacionado con la reactividad en SwiftUI. Antes, se necesitaba usar @Published y Combine explícitamente para observar y responder a cambios en el estado. Ahora, las macros permiten simplificar este proceso.

# ¿Qué es Boilerplate?
El término boilerplate se refiere al código repetitivo o estándar que se necesita escribir para realizar una tarea, pero que en sí mismo no aporta funcionalidad directa o valor específico. Es común en muchas aplicaciones y frameworks donde se requieren configuraciones, declaraciones o estructuras repetitivas.

Ejemplo típico de Boilerplate:

En Swift, antes de que existieran herramientas como @Observable en Swift Macros, para observar cambios en un estado, había que escribir mucho código adicional:
```js
class ViewModel: ObservableObject {
    @Published var title: String = ""
    @Published var isCompleted: Bool = false
}
```

## 4_`Clearn Architecture` que es un tipo de `arquitectura`
Clean Architecture es un enfoque de diseño de software creado por Robert C. Martin (Uncle Bob). Su objetivo es crear aplicaciones modulares, mantenibles y fáciles de escalar, separando claramente las responsabilidades y minimizando las dependencias entre las diferentes capas.

Se basa en el principio de separación de responsabilidades y utiliza capas para organizar el código.

Principios fundamentales de Clean Architecture
	1.	Independencia de frameworks: La lógica de la aplicación no debe depender de ningún framework o herramienta. Esto facilita cambiar frameworks o adaptarse a nuevas tecnologías.
	2.	Independencia de UI: La interfaz de usuario debe ser completamente intercambiable sin modificar la lógica de la aplicación.
	3.	Pruebas facilitadas: La arquitectura está diseñada para que todas las capas sean probables (unit tests, integración, etc.).
	4.	Regla de dependencia: Las dependencias siempre deben ir hacia adentro, hacia los niveles más centrales de la aplicación.

 +---------------------+      +--------------------+
 |        5. UI (View)       |      | 4. Frameworks/Drivers |
 +---------------------+      +--------------------+
          ▲                            ▲
          |                            |
 +---------------------+      +--------------------+
 |  3. Interface Adapters  | <--> |  2. Use Cases         |
 | (Presenters/ViewModels)|  | (Repository)      |
 +---------------------+      +--------------------+
          ▲                            ▲
          |                            |
 +---------------------------------------------+
 |              1. Entities (Model)             |
 +---------------------------------------------+

Capas principales:
1. Entities (Módelos):
	• Representan las reglas de negocio puras, sin dependencias externas.
	• Ejemplo: Modelos de datos como User, Order.
2.	Use Cases (Casos de Uso - Repositorio):
	• Contienen la lógica de negocio específica de la aplicación.
	• Responden preguntas como: “¿Qué hace mi aplicación?”
	• Ejemplo: Registrar un usuario, procesar un pedido.
3.	Interface Adapters (Adaptadores - VM):
	• Adaptan los datos entre la lógica central (casos de uso) y el framework externo.
	• Ejemplo: ViewModels en MVVM, Presenters en MVP.
4.	Frameworks y Drivers:
	• Contienen las herramientas externas necesarias, como bases de datos, APIs, y UI.
	• Ejemplo: SwiftUI, Alamofire, SwiftData, CoreData.
5. La vista

Ventajas de Clean Architecture
• Escalabilidad: al separar responsabilidades, es más fácil añadir nuevas funcionalidades sin romper el sistema existente.
• Mantenibilidad: si un componente cambia, es menos probable que impacte en otras partes del sistema.
• Pruebas: la lógica central es fácilmente testeable porque está aislada de frameworks y herramientas externas.
• Reutilización: las entidades y casos de uso son independientes y reutilizables en diferentes plataformas o interfaces (por ejemplo, puedes reutilizar la lógica entre una app iOS y una app macOS).

## MVVM se puede usar junto con Clean Architecture. De hecho, MVVM puede ser una parte de la capa de “Interface Adapters” dentro de Clean Architecture.
## Esta combinación permite una separación clara de responsabilidades (propio de Clean Architecture) mientras aprovecha el patrón MVVM para gestionar la lógica de presentación.

EJEMPLO:

1. Entidades - Modelos:
```js
struct Task {
    let id: UUID
    var title: String
    var isCompleted: Bool
}
```

2. Caso de Uso - Repositorio:
```js
protocol TaskUseCase {
    func fetchTasks() -> [Task]
    func addTask(title: String)
}

class TaskInteractor: TaskUseCase {
    private var tasks: [Task] = []

    func fetchTasks() -> [Task] {
        return tasks
    }

    func addTask(title: String) {
        let newTask = Task(id: UUID(), title: title, isCompleted: false)
        tasks.append(newTask)
    }
}
```

3. Adaptador (ViewModel):
```js
import Foundation

@Observable
class TaskViewModel {
    private let taskUseCase: TaskUseCase
    var tasks: [Task] = []

    init(taskUseCase: TaskUseCase) {
        self.taskUseCase = taskUseCase
        self.tasks = taskUseCase.fetchTasks()
    }

    func addTask(title: String) {
        taskUseCase.addTask(title: title)
        self.tasks = taskUseCase.fetchTasks()
    }
}
```

4. y 5. Framework (Vista en SwiftUI):
```js
import SwiftUI

struct TaskListView: View {
    @StateObject private var viewModel = TaskViewModel(taskUseCase: TaskInteractor())

    var body: some View {
        List(viewModel.tasks) { task in
            Text(task.title)
        }
        .navigationTitle("Tareas")
        .toolbar {
            Button(action: {
                viewModel.addTask(title: "Nueva Tarea")
            }) {
                Image(systemName: "plus")
            }
        }
    }
}
```

| Aspecto                | Clean Architecture             | Otras arquitecturas (MVC, MVVM simple) |
|------------------------|--------------------------------|----------------------------------------|
| **Separación de responsabilidades** | ✅ Claramente definido                | ❌ Puede estar acoplado                |
| **Mantenibilidad**     | ✅ Alta                         | ❌ Media o baja                        |
| **Flexibilidad**       | ✅ Alta (fácil de adaptar)      | ❌ Menos flexible                      |
| **Escalabilidad**      | ✅ Diseñado para crecer         | ❌ Difícil de escalar                  |
| **Pruebas unitarias**  | ✅ Fácil                        | ❌ Más difícil debido al acoplamiento  |

## =========================== PROYECTO ======================================

## 5_Struct, extensiones, propiedad calculada, protocolos Codable, Identifiable, Hashable (EN EL MODELO)
- Es una extensión de `struct StarCardDTO` (por lo que se podria poner en un fichero aparte). Todo lo que se ponga aquí es algo que se va a incluir a lo que se tiene en el `struct StarCardDTO`

Las extensiones en Swift permiten añadir nuevas funcionalidades a tipos existentes (estructuras, clases, enumeraciones o protocolos) sin necesidad de modificar el código original.

Ventajas:
•	Facilita la organización del código.
•	Evita duplicar lógica o modificar tipos originales.
•	Promueve la reutilización

- toCard: es una propiedad calculada, que no almacena valores sino que cada vez que se la llama devuelve algo que calcula o transforma en tiempo real (como un patrón setter - getter en Java)

Una propiedad calculada no almacena un valor. En lugar de eso, su valor es calculado en tiempo de ejecución cada vez que se accede a ella. Se define con un getter (y opcionalmente un setter).
Es útil porque la transformación de datos (como separar cadenas o capitalizar palabras) se hace solo cuando es necesario, sin almacenar valores redundantes en memoria.

- Protocolo 'Codable' para que pueda cargarse la información del json y transformarse en instancias.
El protocolo Codable combina los protocolos Encodable y Decodable, lo que permite convertir estructuras o clases de Swift a JSON (y viceversa).

- Protocolo 'Identifiable': va a tener un campo 'id' que sera igualable, es decir único.
El protocolo Identifiable define una propiedad id, que se usa para diferenciar objetos de manera única (muy útil con SwiftUI).

- Protocolo 'Hashable': permite tener un valor de hash, un valor de comprobación único. Cuando en todas sus estancias sean iguales es cuando se va a poder comparar por igualdad. Es decir pueden ser dos instancias diferentes pero si tienen todos los mismos valores los compararía como iguales.
El protocolo Hashable permite que un objeto sea almacenado en colecciones como Set o usado como clave en un diccionario (Dictionary).

## 6_Protocolo: Creación de protocolos, extensión a protocolo, struct con dicho protocolo (EN EL REPOSITORIO)
- Creamos un protocolo para aislar a un variable la URL, para que se pueda inyectar un valor distinto de URL y poder itercambiar de forma sencilla entre el json de producción y de testing
El struct tiene el protocolo personalizado el cual tiene los métodos 'url' y 'getData()' (la extensión), la variable 'url' como sucede en los protocolos obliga a implementarla  en el struct al que se llama.

Un protocolo en Swift define un conjunto de requisitos que las clases, estructuras o enumeraciones que lo adopten deben cumplir. Es similar a una interfaz en otros lenguajes como Java o TypeScript. Permite la abstracción, lo que significa que podemos trabajar con cualquier tipo que implemente el protocolo, independientemente de su implementación específica.

- ¿SI EN VEZ DE UN JSON FUERA UNA API SERIA IGUAL?
No sería exactamente igual, pero la estructura se mantiene similar. En lugar de leer el contenido desde un archivo local, se haría una solicitud HTTP para obtener los datos de la API. Esto requeriría algunas modificaciones:

```js
// En lugar de trabajar con url como una propiedad, podríamos agregar un método para hacer una solicitud a la API:
protocol DataRepository {
    func fetchData() async throws -> [StarCard]
}

// Utilizaríamos URLSession para hacer la solicitud a la API:
extension DataRepository {
    func fetchData() async throws -> [StarCard] {
        let (data, _) = try await URLSession.shared.data(from: url) // Realiza una solicitud HTTP.
        let decoder = JSONDecoder()
        decoder.keyDecodingStrategy = .convertFromSnakeCase
        return try decoder.decode([StarCardDTO].self, from: data).map(\.toCard)
    }
}

// Las estructuras seguirían definiendo la URL, pero ahora apuntarían a una API en lugar de un archivo local:
struct Repository: DataRepository {
    var url: URL {
        URL(string: "https://api.example.com/starcards")! // URL de la API.
    }
}
```

# Comparativa: JSON Local vs API

| **Aspecto**           | **JSON Local**                                   | **API**                                                    |
|-----------------------|-------------------------------------------------|-----------------------------------------------------------|
| **Acceso a los datos** | Lee datos directamente desde un archivo en el dispositivo. | Realiza solicitudes HTTP para obtener datos de un servidor. |
| **Rendimiento**       | Más rápido (no depende de la red).             | Más lento debido a la latencia de red.                    |
| **Conexión a internet** | No requiere internet.                             | Requiere conexión a internet.                             |
| **Actualización de datos** | Los datos son estáticos.                           | Los datos pueden ser dinámicos y actualizados en tiempo real. |
| **Gestión de errores**  | Controla errores al leer archivos (por ejemplo, archivo no encontrado). | Controla errores de red (por ejemplo, tiempo de espera, servidor no disponible). |

## 7_Observable, final class, variable con protocolo personalizado, init (EN EL VISTA-MODELO)
- Se anota la clase de tipo observable, que es lo que me va a permitir que cualquier propiedad variable que haya en la clase cuando se actualice la clase al ponerla en una vista se refresque la vista
- Protocolo DataRepository, que lo curioso es que tambien es un tipo de dato. Y dentro de la variable va a estar tanto la instancia del Struct de 'Repository' como de 'RepositoryTest', las cuales tienen el protocolo DataRepository. Basicamente la viarable va apoder acceder a todos los datos de que tiene dicho protocolo.
- Creamos el inicializador que ponemos que por defecto reciba Repository() (producción) pero se va a poder cambiar en la vista donde se invoca (ContentView.swift) por el de RepositoryTest (desarrollo) de manera muy sencilla y escalable.
- final class: ¿Porque no puede ser un actor o struct?
En Swift, una clase marcada como final no puede ser heredada por otras clases. Esto significa que no puedes crear subclases a partir de ella.

- Motivo para usar final:
• Mejor rendimiento: El compilador puede optimizar las llamadas de métodos porque sabe que no habrá subclases que puedan sobrescribirlos.
• Seguridad: Si sabes que la clase no debe ser extendida (por diseño o por lógica del negocio), el uso de final refuerza esa intención.
• En el contexto de un ViewModel, no tiene mucho sentido que otras clases lo hereden, ya que está diseñado para contener la lógica específica de una vista.

## 8_Llamada a struct (como tipo de dato, que herera a las extensiones) (EN LA VISTA SECUNDARIA)
- Cargamos ya una instancia de una variable llamada 'test' (definida antes en 'Model.swift') donde la variable es una una extensión StarCard (struct). Para la Preview

## 9_@State, #Preview (EN LA VISTA)
- @State: indica al sistema que cuando cambie ciertas propiedades provocara que la vista se actualice. En este caso el observable StarCardVM() el cual tiene la variable cards en el momento que esta cambie automaticamente refrescará la pantalla y lo cambiará
- #Preview, esto lo que hace es dar un contexto en el que creo la instancia de 'ContentView' para que se vea en el simulador de la derecha de Xcode.

## 10_@testable, Suite Testing, .tags (EN EL TEST)
-  es mi aplicación StarWarsDemo (import StarWarsDemo), pero añadimos el @testable, para que acceda a todos los componentes de nuestra aplicación y se pueda invocar dichos componentes

-  No son Struct sino Suite los test

- puede haber diferentes elementos como para deshabilitar que este en el ipad, etc. Además del elemento etiqueta que ponemos para decir que es un Suite del repositorio nuestro aplicación el test que estamos realizando

### 11_¿SE DEBERIA DE IMPLEMENTAR ACTORES EN ALGUN MOMENTO?
¿Se debería implementar actores en este caso?

Los actores son ideales para manejar concurrencia segura, ya que garantizan que solo un hilo pueda acceder a su estado a la vez. En este caso:
	1.	¿Por qué no es necesario?
	•	En tu implementación actual, el ViewModel (StarCardVM) carga todos los datos de manera síncrona al inicializarse. No hay cambios concurrentes en cards, ya que los datos se cargan una sola vez y se mantienen estáticos.
	•	El acceso a las propiedades (cards) ocurre solo desde la vista (un solo hilo).
	2.	¿Cuándo sería útil implementar actores?

Si en el futuro planeas:
	•	Cargar datos asincrónicamente: Por ejemplo, desde una API o un servicio remoto en segundo plano.
	•	Actualizar datos dinámicamente: Por ejemplo, si los datos de las cartas pueden cambiar (editar, eliminar, agregar) desde múltiples fuentes simultáneamente.
	•	Evitar condiciones de carrera: Cuando diferentes partes de tu app intenten modificar el estado del ViewModel al mismo tiempo.

En estos casos, podrías refactorizar StarCardVM como un actor:
```js
actor StarCardVM {
    private let repository: DataRepository
    private(set) var cards: [StarCard] = []

    init(repository: DataRepository = Repository()) {
        self.repository = repository
    }

    func loadCards() async {
        do {
            cards = try repository.getData()
        } catch {
            cards = []
        }
    }
}
```
Y la vista se ajustaría para interactuar con este actor:
```js
struct ContentView: View {
    @StateObject private var vm = StarCardVM()

    var body: some View {
        NavigationStack {
            List {
                ForEach(vm.cards, id: \.id) { card in
                    StarCardView(card: card)
                }
            }
            .navigationTitle("Star Wars")
            .task {
                await vm.loadCards()
            }
        }
    }
}
```

# AÑADIR FUNCIONES A LA APP

### 1. Buscador de cartas
•	Descripción: Agregar un campo de búsqueda que permita filtrar las cartas por nombre o por alguna característica (raza, afiliación, etc.).
•	Cómo implementarlo:
1.	Añade un estado en la vista para almacenar el texto de búsqueda:

```js
    @State private var searchText = ""
```

Filtra las cartas en función del texto ingresado:
```js
var filteredCards: [StarCard] {
    if searchText.isEmpty {
        return vm.cards
    } else {
        return vm.cards.filter { $0.nombre.lowercased().contains(searchText.lowercased()) }
    }
}
```
Agrega un SearchBar en la vista:
```js
NavigationStack {
    List {
        ForEach(filteredCards, id: \.id) { card in
            StarCardView(card: card)
        }
    }
    .navigationTitle("Star Wars")
    .searchable(text: $searchText, prompt: "Buscar por nombre...")
}
```

### 2. Favoritos
• Descripción: Permitir que los usuarios marquen cartas como favoritas y puedan verlas en una sección separada.
• Cómo implementarlo:
1.	Añade un estado booleano en el modelo StarCard para indicar si es favorita:
```js
struct StarCard: Identifiable, Hashable {
    let id: Int
    ...
    var isFavorite: Bool = false
}
```

2.	Crea una función en el ViewModel para alternar el estado de favorito:
```js
func toggleFavorite(for card: StarCard) {
    if let index = cards.firstIndex(where: { $0.id == card.id }) {
        cards[index].isFavorite.toggle()
    }
}
```

3.	Agrega un botón en la vista para marcar como favorito:
```js
Button(action: {
    vm.toggleFavorite(for: card)
}) {
    Image(systemName: card.isFavorite ? "star.fill" : "star")
        .foregroundColor(card.isFavorite ? .yellow : .gray)
}
```

### 3. Filtrado por afiliación, raza o habilidades
•	Descripción: Implementa un filtro dinámico para seleccionar cartas basadas en su afiliación, raza o habilidades.
•	Cómo implementarlo:
1.	Añade un menú desplegable (Picker) con opciones de filtro:
```js
@State private var selectedFilter: String = "Todas"
let filters = ["Todas", "Alianza Rebelde", "Nueva República", "Uso de la Fuerza", "Humano"]
```

2.	Filtra las cartas según la selección:
```js
var filteredCards: [StarCard] {
    if selectedFilter == "Todas" {
        return vm.cards
    } else {
        return vm.cards.filter { $0.afiliacion.contains(selectedFilter) || $0.habilidades.contains(selectedFilter) || $0.raza == selectedFilter }
    }
}
```

3.	Agrega un Picker en la vista:
```js
Picker("Filtrar", selection: $selectedFilter) {
    ForEach(filters, id: \.self) { filter in
        Text(filter)
    }
}
.pickerStyle(.segmented)
```

### 4. Mini Juego: Trivia Simpson
Funcionalidad
	•	Un juego de preguntas de trivia sobre personajes y eventos de “Star Wars”.

Implementación
	•	View: Una vista TriviaGameView que muestra preguntas con opciones múltiples.
	•	ViewModel: Crear un TriviaGameVM para manejar la lógica del juego.
	•	Actor:
	•	Usar un actor GameStateManager para manejar la puntuación y las preguntas de forma concurrente.

```js
actor GameStateManager {
    private(set) var score: Int = 0
    private var questions: [TriviaQuestion]

    init(questions: [TriviaQuestion]) {
        self.questions = questions
    }

    func getNextQuestion() -> TriviaQuestion? {
        return questions.isEmpty ? nil : questions.removeFirst()
    }

    func updateScore(correct: Bool) {
        score += correct ? 1 : 0
    }

    func getScore() -> Int {
        return score
    }
}
```

### 5. Comparador de Cartas (StarCard Comparator)
Funcionalidad
	•	Una vista para comparar dos cartas lado a lado, mostrando atributos como habilidades, armas, etc.

Implementación
	•	View: Una vista llamada ComparatorView para mostrar las cartas seleccionadas.
	•	ViewModel: Crear un ComparatorVM que maneje las dos cartas seleccionadas.
	•	Actor:
	•	Usar un actor ComparisonManager para realizar comparaciones de atributos en segundo plano.

```js
actor ComparisonManager {
    func compare(card1: StarCard, card2: StarCard) -> [String: String] {
        return [
            "Habilidades": card1.habilidades.joined(separator: ", ") == card2.habilidades.joined(separator: ", ") ? "Empate" : "Diferente",
            "Armas": card1.armas.joined(separator: ", ") == card2.armas.joined(separator: ", ") ? "Empate" : "Diferente"
        ]
    }
}
```