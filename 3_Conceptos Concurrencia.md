
## 1_SINCRONÍA Y ASINCRONÍA

| **Concepto**      | **Descripción?** |
|-------------------|----------------------|
| **Síncrono**      | El flujo de ejecución espera a que la tarea termine antes de continuar.  |
| **Asíncrono**     | La tarea se ejecuta en segundo plano, permitiendo que el flujo principal continúe. |

### Síncrono
Cuando una operación se ejecuta de manera `síncrona`, el flujo de ejecución `espera a que la operación termine` antes de continuar con las siguientes instrucciones. En otras palabras:
•	La tarea actual `bloquea el hilo` hasta que se `complete`.
•	`Es secuencial`: una tarea debe completarse antes de que comience la siguiente.

```js
// SWIFT
print("Inicio")
// Esto es una operación síncrona
for i in 1...3 {
    print("Tarea \(i)")
}
print("Fin")

// Inicio
// Tarea 1
// Tarea 2
// Tarea 3
// Fin
```

### Asíncrono
Cuando una operación es asíncrona, se `ejecuta en segundo plano` y `no bloquea el flujo principal`. Esto permite que `el programa continúe ejecutándose` mientras la operación está en progreso.
•	Las tareas pueden completarse en paralelo o en otro hilo.
•	El resultado o la acción de la tarea asíncrona puede manejarse a través de callbacks, promesas o clausuras.

```js
// SWIFT
print("Inicio")

DispatchQueue.global().async {
    for i in 1...3 {
        print("Tarea \(i)")
    }
}

print("Fin")

// Inicio
// Fin
// Tarea 1
// Tarea 2
// Tarea 3
```

## 2_SERIALIZACIÓN (3)

| **Significado**      | **Contexto?**        | **Ejemplo**                                                                                       |
|-------------------|----------------------|-------------------------------------------------------------------------------------------------------|
| **Serialización de datos**      | Transmisión y almacenamiento de objetos     | JSON, XML, Protobuf                                         |
| **Serialización de tareas**     | Las tareas se ejecutan en el orden en que fueron enviadas, una por una, incluso si son asíncronas.   | DispatchQueue en Swift. |
| **Serialización de bytes**      | Transformar datos complejos en un flujo de bytes                | Programación de sockets, persistencia binaria           |

### 1. Serialización de datos
La `serialización de datos` se refiere al proceso de convertir `un objeto o estructura de datos` en un `formato que pueda ser almacenado o transmitido` (JSON, XML, Protobuf o formatos binarios personalizados) y luego reconstruido posteriormente (deserialización).

•	Usado para enviar datos a través de `redes, guardarlos en archivos, o transferirlos entre diferentes sistemas`.
•	Los formatos más comunes incluyen `JSON, XML, Protobuf, o formatos binarios personalizados`.

```js
// SWIFT
struct User: Codable {
    let name: String
    let age: Int
}

let user = User(name: "Carlos", age: 25)

// Serialización a JSON
let jsonData = try JSONEncoder().encode(user)
```

### 2. Serialización de tareas o procesos
La `serialización de tareas` implica ejecutar tareas de manera secuencial (una tras otra sin superponerse como una serie (de hay viene serialmente)), en lugar de hacerlo de forma concurrente o paralela.

•	En `programación concurrente`, cuando necesitas asegurar que ciertas `operaciones se ejecuten en orden`, puedes usar `una cola o mecanismo serializado`.
•	Esto se utiliza para evitar `condiciones de carrera  (data race)` y garantizar consistencia en operaciones compartidas.

```js
// SWIFT
// La cola serialQueue es serial porque cada tarea enviada a ella se ejecutará una después de la otra, sin importar si son tareas síncronas o asíncronas.
let serialQueue = DispatchQueue(label: "com.example.serialQueue")

//  Ambas tareas (Tarea 1 y Tarea 2) se envían a la cola usando async. Esto significa que se programan para ejecutarse más tarde y no bloquean el flujo principal.
serialQueue.async {
    print("Tarea 1")
}

serialQueue.async {
    print("Tarea 2")
}

// Cuando la cola comienza a ejecutar las tareas, asegura que Tarea 1 se complete antes de empezar con Tarea 2.
// Esto es lo que hace que las tareas sean serializables, porque se ejecutan secuencialmente, respetando el orden en que fueron enviadas.
```
### 3. Serialización de bytes o flujo de datos
La serialización en este contexto implica `convertir datos complejos en un flujo de bytes` para ser enviados a través de un canal (como una red o un archivo) y luego reconstruidos en el destino.

•	Común en `sistemas distribuidos, programación de sockets o persistencia binaria`.
•	Ejemplo: Guardar un archivo en disco en su forma binaria o transmitir datos entre cliente y servidor.

```js
// JAVA
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("user.ser"));
out.writeObject(user); // Serializa un objeto a un archivo
```

## 3_RELACIÓN ENTRE ASÍNCRONO Y SERIALIZABLE

### ¿Qué significa que una cola sea “serializable”?
En el contexto de `DispatchQueue en Swift`, una cola serializada significa que `las tareas enviadas a esa cola se ejecutan en orden`, una por una, `incluso si se ejecutan de forma asíncrona`.
•	`Asíncrono aquí significa` que las tareas se `programan para ejecutarse más tarde (se guardan el cola del hilo)`, pero el orden en el que se ejecutan sigue siendo secuencial (serializado).
•	Esto asegura que las tareas 'no se ejecuten al mismo tiempo en hilos diferentes', `eliminando condiciones de carrera (data race)`.

```js
let serialQueue = DispatchQueue(label: "com.example.serialQueue")

serialQueue.async {
    print("Tarea 1")
}

serialQueue.async {
    print("Tarea 2")
}
```

* Explicación paso a paso:
	1.	Creación de una cola serial:
	•	La cola serialQueue es serial porque cada tarea enviada a ella se ejecuta una después de la otra, sin importar si son tareas síncronas o asíncronas.
	•	Solo una tarea puede ejecutarse en esa cola en un momento dado.
	2.	Envío de tareas asíncronas:
	•	Ambas tareas (Tarea 1 y Tarea 2) se envían a la cola usando async. Esto significa que se programan para ejecutarse más tarde y no bloquean el flujo principal.
	•	Sin embargo, debido a que la cola es serial, las tareas se ejecutan en el mismo orden en que fueron enviadas.
	3.	Ejecución ordenada:
	•	Cuando la cola comienza a ejecutar las tareas, asegura que Tarea 1 se complete antes de empezar con Tarea 2.
	•	Esto es lo que hace que las tareas sean serializables, porque se ejecutan secuencialmente, respetando el orden en que fueron enviadas.

* ¿Qué pasa si usas una cola concurrente en lugar de una serial?
```js
let concurrentQueue = DispatchQueue(label: "com.example.concurrentQueue", attributes: .concurrent)

concurrentQueue.async {
    print("Tarea 1")
}

concurrentQueue.async {
    print("Tarea 2")
}

// Tarea 2
// Tarea 1
// O al revés ahora no hay un control de cual se ejecuta antes o después
```

## 4_CONCURRENCIA --> A LA VEZ VARIAS TAREAS (UN HILO EN CADA TAREA EN APPLE SILICON DE FORMA PARALELA, SI NO QUEDAN HILOS SE ENCOLA HASTA QUE SE HABILITE UNO)

## 5_HILO --> Cada hilo es secuencial y no puede hacer más de una tarea (operación) a la vez en dicho HILO
`Hilo principal (main thread)`: se encarga de la interfaz de usuario. Ejecuta las tareas de `forma síncrona y serializada`.

## 6_CONCURRENCIA ESTRICTA --> SOLUCIONA EL PROBLEMA DE Los `data race` se forman cuando existen datos mutables (variables) que tienen un acceso compartido entre distintos hilos. Y el probelma se da cuando múltiples tareas acceden y modifican datos compartidos simultáneamente
SWIFT TE NOTIFICA DE `data race`: `no que lo vaya a hacer, que en dichas condiciones pueda darse`.

## 7_ACTORES: la herramienta principal para usar `async-await` sin `data race` son los `actores`.
`actores`: clases preparadas para concurrencia (que no soportan herencia) que convierten en asíncrono (no bloqueante) el acceso a métodos o propiedades en los mismos y con ello, obligan a esperar a todo el que quiera usarlo si alguien ya lo usa.

* `@MainActor` lo estamos "atando" al `main thread` Si marcamos una clase, struct, enumeración, método o propiedad como `@MainActor` lo estamos "atando" al `main thread` por lo que dicho método, propiedad o instancia que sea, solo será accesible en el hilo principal o instancia que sea, solo será accesible en el hilo principal y nada más y como hemos dicho este es hilo es secuencial y nos quita el problema de `los data race`.

Lo que no podemos olvidar es que no podemos poner `tareas lentas o pesadas` sobre el hilo principal porque estamos `parando la interfaz de la aplicación`.
Cualquier `proceso que sea lento` siempre debe de estar en un `contexto secundario (task)` para poder hacerlo bien.

Los actores (estado mutable compartido)
    1. Los actores son clases preparadas para asincronía.
    2. Sus propiedades evitan el data race bloqueando el acceso a cualquier propiedad, por lo que obligan al uso de sus instancias dentro de un contexto Task o cuando atamos un método o clase a un actor global.
    3. En la propia definición del actor, la protección a la concurrencia es implícita y no hay que programarla, no hace falta poner async-await, ni nada.
    4. Todos las propiedades mutables y los métodos que acceden a las mismas están protegidos por defecto con aislamiento y protección de su contexto de forma automática.
    5. Pero cuando creamos una instancia de un actor, el acceso a cualquier propiedad mutable o método que acceda a estas se vuelve de acceso asíncrono, siempre en un Task.
    6. Así evita que cuando dos procesos intenten leer a la vez un dato, puedan conseguirlo porque uno de ellos siempre queda a la espera que el otro termine.
    7. El actor es el ejemplo perfecto de cómo pasar un dato de un contexto a otro: usando para su acceso un await usado como elemento asíncrono.

## 8_PATRÓN SINGLETON
El patrón Singleton es un patrón de diseño que garantiza que una clase tenga una única instancia global accesible en todo el programa y proporciona un punto de acceso centralizado a esa instancia.

Cuando un Singleton contiene propiedades mutables, puede generar problemas en aplicaciones concurrentes porque:
- Acceso simultáneo no controlado: Si múltiples hilos acceden y modifican las propiedades al mismo tiempo, puede haber condiciones de carrera.
- Estado inconsistente: Los datos pueden corromperse si no se sincroniza correctamente el acceso.
- Falta de aislamiento: Todos los hilos tienen acceso a la misma instancia, y cualquier cambio afecta a todos.

Solucion Usar actores como comentamos antes
En Swift moderno, puedes reemplazar el Singleton tradicional con un actor para manejar la concurrencia automáticamente.

## 9_ACTORES GLOBALES (`@Globalactor`) ANTE ACTORES REGULARES

• A diferencia de los actores regulares que solo protegen sus propias propiedades, un actor global puede proteger cualquier código en cualquier parte de la aplicación, proporcionando un mecanismo de sincronización a nivel de sistema.

# Actores Regulares vs Actores globales

| **Característica**      | **Actor Regular?**        | **Actor Global**                                                                                       |
|-------------------|----------------------|-------------------------------------------------------------------------------------------------------|
| **Instancias**      | Pueden existir múltiples instancias.    | Solo hay una única instancia global.                                    |
| **Accesibilidad**     | Solo accesible desde donde fue instanciado.   | Accesible en cualquier parte del programa. |
| **Uso típico**      | Aislamiento local o funcionalidad específica.                | Estados compartidos o acceso global.           |
| **Declaración**      | Declarado como una clase actor regular.               | Declarado como una propiedad global o estática.           |


1. Actores Regulares

Los actores regulares son instancias de una clase de tipo actor que creas explícitamente en tu código. Cada instancia de un actor tiene su propio contexto aislado, lo que significa que protege sus datos internos del acceso concurrente.

Características de los actores regulares:
	1.	Aislamiento por instancia:
	•	Cada instancia tiene su propia cola serial para manejar el acceso a su estado.
	•	Esto asegura que dos hilos no puedan modificar las mismas propiedades de la misma instancia al mismo tiempo.
	2.	Creados explícitamente:
	•	Tú controlas cuándo y cuántas instancias del actor existen.
    	3.	Ámbito limitado:
	•	El ámbito del actor está limitado a donde fue creado (a menos que lo compartas explícitamente).
	4.	Uso típico:
	•	Útil para modelos aislados, como manejar estados de un componente específico o encapsular lógica relacionada con una funcionalidad particular.

2. Actores Globales

Un actor global es un actor que está disponible globalmente en toda la aplicación. Se utiliza para proporcionar un único punto de acceso a ciertos datos o funcionalidades compartidas en toda la app.

Características de los actores globales:
	1.	Instancia única:
	•	Solo hay una instancia global del actor, y esta instancia está creada por el sistema o definida explícitamente como estática.
	2.	Siempre accesible:
	•	Está disponible en cualquier lugar del programa sin necesidad de instanciarlo.
	3.	Declaración como propiedad estática:
	•	Por lo general, un actor global se define como una propiedad estática para garantizar que solo exista una única instancia.
	4.	Uso típico:
	•	Ideal para manejar estados compartidos o recursos comunes en toda la aplicación, como el manejo de configuraciones, almacenamiento en caché o acceso a recursos externos.

### 9.1_COMPARATIVA EJEMPLO PRÁCTICOS
* Por qué un actor regular no funcionaría bien
El problema surge cuando se crean `múltiples instancias` de un `actor regular`, ya que cada instancia tiene su propio aislamiento. Esto significa que el `estado` y las `operaciones` no se compartirían entre `instancias`, lo cual es crítico para recursos centralizados como las solicitudes API.

### Ejemplo: Actor regular con múltiples instancias

```js
actor RegularAPIActor {
    var requestCount = 0

    func fetchEmployees() async throws -> [String] {
        requestCount += 1
        print("Request Count: \(requestCount)")
        return ["Employee1", "Employee2"]
    }
}

func simulateMultipleAPIAccess() async {
    let actor1 = RegularAPIActor() // instancia 1
    let actor2 = RegularAPIActor() // instancia 2

    // Simulan diferentes partes de la app usando distintas instancias
    async let result1 = actor1.fetchEmployees()
    async let result2 = actor2.fetchEmployees()

    // Los resultados de las instancia 1 y 2 no se coordinan, poir lo que devolveran el mismo resultado
    let results = try? await (result1, result2)
    print("Results: \(results ?? ([], []))")
}

// Llamada
Task {
    await simulateMultipleAPIAccess()
}

// SALIDA
Request Count: 1
Request Count: 1
Results: (["Employee1", "Employee2"], ["Employee1", "Employee2"])
```

Problema:
1.	Inconsistencia de estado:
•	Cada instancia (`actor1` y `actor2`) tiene su propio `estado aislado (requestCount)`, lo que lleva a valores incoherentes. Ambas empiezan desde `requestCount = 0`, aunque están realizando la misma operación API.
2.	Acceso descoordinado:
•	Las partes de la aplicación `no comparten` un punto único de control para `las solicitudes API`, lo que puede generar conflictos y dificultar la depuración.

### Ejemplo: Solución con un actor global
Con un actor global, garantizamos que todas las operaciones pasen por una única instancia compartida.

```js
@globalActor
actor APIActor {
    static let shared = APIActor() // se declara una unica instacia

    var requestCount = 0

    func fetchEmployees() async throws -> [String] {
        requestCount += 1
        print("Request Count: \(requestCount)")
        return ["Employee1", "Employee2"]
    }
}

func simulateGlobalAPIAccess() async {
    // No hay que declarar las instancias porque solo hay una unica instancia: 'APIActor.shared'

    // Simulan diferentes partes de la app usando distintas instancias
    async let result1 = APIActor.shared.fetchEmployees()
    async let result2 = APIActor.shared.fetchEmployees()

    let results = try? await (result1, result2)
    print("Results: \(results ?? ([], []))")
}

// Llamada
Task {
    await simulateGlobalAPIAccess()
}

// SALIDA
Request Count: 1
Request Count: 2
Results: (["Employee1", "Employee2"], ["Employee1", "Employee2"])
```

Por qué funciona:
1.	Estado único compartido:
•	Todos los accesos a la API pasan por el actor global `(APIActor.shared)`, por lo que el estado `requestCount` se actualiza de forma consistente.
2.	Concurrencia segura:
•	El `actor global` serializa todas las operaciones, asegurando que las solicitudes no entren en conflicto.

=========================================================
## 10_¿Podría implementarse el código anterior con struct?

No de forma segura si deseas manejar concurrencia.

Razones:
	1.	Structs no tienen aislamiento concurrente:
	•	Las struct no tienen mecanismos internos para proteger datos compartidos del acceso concurrente. Si múltiples hilos acceden y modifican simultáneamente una instancia de struct, se producirán condiciones de carrera.
	2.	No pueden garantizar serialización de acceso:
	•	Un struct no puede asegurar que las operaciones se realicen en un orden controlado, lo que podría llevar a estados inconsistentes.
	3.	No pueden encapsular sincronización:
	•	Los actores tienen una cola serial integrada, mientras que con una struct tendrías que implementar manualmente mecanismos como colas (DispatchQueue) o bloqueos (NSLock) para garantizar la seguridad.

```js
struct APIManager {
    var requestCount = 0

    mutating func fetchEmployees() -> [String] {
        requestCount += 1
        print("Request Count: \(requestCount)")
        return ["Employee1", "Employee2"]
    }
}

func simulateStructAPIAccess() {
    var apiManager = APIManager()

    DispatchQueue.global().async {
        let _ = apiManager.fetchEmployees()
    }

    DispatchQueue.global().async {
        let _ = apiManager.fetchEmployees()
    }
}

simulateStructAPIAccess()

// SALIDA (Salida inconsistente debido a condiciones de carrera):
Request Count: 1
Request Count: 1
```

Conclusión
•	Actores regulares: Funcionan bien para instancias aisladas, pero no son adecuados para recursos compartidos entre diferentes partes de la aplicación, como en este caso.
•	Actor global: Es ideal para centralizar el acceso y garantizar coherencia en recursos compartidos como las solicitudes API.
•	Struct: Aunque podrían usarse, requerirían implementar manualmente mecanismos de sincronización y protección contra concurrencia, lo que los hace más propensos a errores.

## 11_¿Cuándo usar actores regulares, actores globales o structs en casos prácticos?

| **Caso**                          | **Actor Regular**         | **Actor Global**         | **Struct**            |
|------------------------------------|---------------------------|--------------------------|------------------------|
| **Entidad con estado aislado**    | ✅ Sí                     | 🚫 No (es global)        | 🚫 No (sin aislamiento) |
| **Recurso compartido**            | 🚫 No (cada instancia es independiente) | ✅ Sí                    | 🚫 No                 |
| **Datos simples e inmutables**    | 🚫 Ineficiente            | 🚫 Ineficiente           | ✅ Ideal              |
| **Concurrencia segura**           | ✅ Sí                     | ✅ Sí                    | 🚫 No                 |
| **Estado mutable compartido**     | 🚫 No                     | ✅ Sí                    | 🚫 No                 |

1. Actores regulares
Actores regulares son ideales cuando necesitas aislar el estado de una entidad específica para evitar condiciones de carrera, pero no necesariamente necesitas compartir este estado de forma global. Cada instancia de un actor tiene su propio aislamiento.

Cuándo usarlos:
	•	Cuando cada entidad debe manejar su propio estado concurrente de forma independiente.
	•	Para objetos que representan recursos únicos o encapsulan lógica relacionada con un elemento específico (como un usuario, documento, etc.).
	•	Si necesitas que diferentes instancias trabajen de forma independiente pero segura frente a accesos concurrentes.

Un ejemplo: Un sistema donde múltiples usuarios acceden a su propia configuración de cuenta.
```js
actor UserSettings {
    var preferences: [String: String] = [:]

    func updatePreference(key: String, value: String) {
        preferences[key] = value
    }

    func getPreference(key: String) -> String? {
        preferences[key]
    }
}

// Cada usuario tiene su propio aislamiento
let user1 = UserSettings()
let user2 = UserSettings()

Task {
    await user1.updatePreference(key: "theme", value: "dark")
    print(await user1.getPreference(key: "theme")) // Output: dark
}
Task {
    print(await user2.getPreference(key: "theme")) // Output: nil (usuario independiente)
}
```

2. Actores globales

Un actor global es útil cuando necesitas compartir un único estado o recurso de manera centralizada en toda la aplicación, asegurando que solo haya una instancia y que las operaciones sean seguras frente a concurrencia.

Cuándo usarlos:
	•	Cuando el recurso debe ser compartido globalmente en toda la aplicación.
	•	Para coordinar el acceso a un recurso único, como:
	•	Una API que maneja todas las solicitudes de red.
	•	Un almacenamiento de caché centralizado.
	•	Un controlador que coordina datos globales.
	•	Para simplificar el acceso a un recurso común evitando crear múltiples instancias.

Ventaja del actor global:
Permite que cualquier parte de tu aplicación acceda al recurso compartido sin preocuparse por crear múltiples instancias o manejar manualmente la sincronización.

Ejemplo práctico:
Un sistema centralizado para manejar solicitudes de API

```js
@globalActor
actor APIActor {
    static let shared = APIActor()
    var requestCount = 0

    func fetchData() async -> String {
        requestCount += 1
        return "Data fetched. Total requests: \(requestCount)"
    }
}

@APIActor
func performGlobalRequest() async {
    let data = await APIActor.shared.fetchData()
    print(data)
}

Task {
    await performGlobalRequest() // Data fetched. Total requests: 1
    await performGlobalRequest() // Data fetched. Total requests: 2
}
```

3. Structs

Las structs son ideales cuando el diseño es inmutable o no requiere concurrencia. Funcionan bien para datos ligeros, valores puros o configuraciones que no cambian con frecuencia.

Cuándo usarlas:
	•	Para datos simples e inmutables (o casi inmutables).
	•	Si no necesitas manejar concurrencia.
	•	Cuando el diseño es sencillo y no necesitas encapsular lógica compleja ni proteger acceso concurrente.
	•	Cuando el estado no será compartido entre hilos.

Problema con concurrencia:
Si intentas usar una struct mutable en un entorno concurrente, te enfrentarás a condiciones de carrera, ya que structs no tienen aislamiento integrado.

Ejemplo práctico:

Un modelo de datos para representar un empleado:
```js
struct Employee {
    let id: Int
    let name: String
    let position: String
}

// Uso de la struct
let employee = Employee(id: 1, name: "John Doe", position: "Developer")
print(employee.name) // Output: John Doe
```

### 12_¿Y las clases?

| **Característica**            | **Actor**                              | **Clase**                             | **Struct**                            |
|--------------------------------|----------------------------------------|---------------------------------------|---------------------------------------|
| **Tipo**                       | Referencia aislada (concurrencia segura) | Referencia (sin aislamiento)          | Valor (sin referencia)                |
| **Concurrencia segura**        | ✅ Sí                                  | ❌ No                                 | ✅ Sí (por valor, no se comparte)      |
| **Herencia**                   | ❌ No                                  | ✅ Sí                                 | ❌ No                                  |
| **Estado compartido mutable**  | ✅ Seguro                              | ✅ (pero inseguro en concurrencia)    | ❌ No                                  |
| **Simplicidad para concurrencia** | ✅ Muy sencillo                        | ❌ Necesita bloqueos manuales         | ✅ (no se comparte estado)             |
| **Overhead (coste adicional)** | Mayor que una clase                   | Menor que un actor                   | Menor que ambos (almacenado en pila)  |
| **Patrones orientados a objetos** | ❌ No                                  | ✅ Sí                                 | ❌ No                                  |

* Regla general
	1.	Usa structs cuando:
	•	El modelo representa datos simples e independientes (como un Point, User o Product).
	•	No necesitas herencia ni referencias compartidas.
	•	Quieres aprovechar las ventajas de rendimiento en la pila.
	2.	Usa clases cuando:
	•	Necesitas herencia.
	•	Varias partes del programa deben compartir el mismo estado.
	•	El modelo es complejo y mutará frecuentemente.


- ¿Cuándo deberías usar actores en lugar de clases?

Usa actores cuando:
	1.	Necesitas concurrencia segura:
	•	Si tu aplicación tiene múltiples hilos accediendo y modificando el mismo estado, los actores eliminan el riesgo de condiciones de carrera y hacen que la concurrencia sea más simple y predecible.
	•	Por ejemplo, manejar datos de red compartidos, como una caché central o el acceso a una API.
	2.	El estado debe estar aislado:
	•	Los actores aseguran que solo se pueda interactuar con su estado desde un único hilo a la vez. Esto te permite escribir código más limpio y evitar problemas de sincronización manual.
	3.	Prefieres delegar la complejidad de la sincronización:
	•	Con actores, no necesitas usar manualmente herramientas como DispatchQueue o bloqueos (NSLock), ya que el aislamiento de estado viene “de fábrica”.
	4.	Estás desarrollando aplicaciones modernas:
	•	Los actores están diseñados para aprovechar las capacidades de concurrencia introducidas en Swift (como async/await y structured concurrency), haciéndolos ideales para aplicaciones que dependen de la ejecución concurrente.

- ¿Cuándo es mejor seguir usando clases?

Usa clases si:
	1.	No necesitas concurrencia:
	•	Si el estado de tu aplicación no es accedido concurrentemente, las clases son más sencillas y no tienen la sobrecarga de los actores.
	•	Por ejemplo, manejar un modelo de datos en una aplicación simple.
	2.	Necesitas herencia:
	•	Los actores no soportan herencia. Si estás trabajando con un diseño orientado a objetos basado en herencia, las clases siguen siendo la mejor opción.
	•	Por ejemplo, si tienes una jerarquía de clases como Animal -> Mammal -> Dog.
	3.	Tu objeto no necesita aislamiento:
	•	Si el objeto será usado de forma local o no compartida, una clase puede ser suficiente y más eficiente.
	4.	Usas patrones de diseño como delegados:
	•	Muchos patrones de diseño comunes en iOS, como el patrón delegado, están basados en clases.

– Cuándo usar structs en lugar de actores o clases?

Usa structs si:
	1.	No necesitas compartir estado:
	•	Las structs son tipos por valor, por lo que no tienen problemas de concurrencia porque no se comparten entre hilos.
	2.	El estado es inmutable o casi inmutable:
	•	Si tus datos no cambian o solo se inicializan una vez, las structs son más ligeras y eficientes.
	3.	El diseño es funcional o simple:
	•	Las structs son ideales para modelos de datos simples, como User, Point, o Product.
	4.	No necesitas identidad de objeto:
	•	Las structs no tienen identidad propia como las clases. Si no necesitas que dos referencias apunten al mismo objeto, usa una struct.
