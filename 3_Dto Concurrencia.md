
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
La `serialización de datos` se refiere al proceso de convertir `un objeto o estructura de datos` en un `formato que pueda ser almacenado o transmitido`(JSON, XML, Protobuf o formatos binarios personalizados) y luego reconstruido posteriormente (deserialización).

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
1. `Hilo principal (main thread)`: se encarga de la interfaz de usuario. Ejecuta las tareas de `forma síncrona y serializada`. 

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

## Patrón Singleton y sus riesgos
El patrón Singleton es un patrón de diseño que garantiza que una clase tenga una única instancia global accesible en todo el programa y proporciona un punto de acceso centralizado a esa instancia.

Cuando un Singleton contiene propiedades mutables, puede generar problemas en aplicaciones concurrentes porque:
- Acceso simultáneo no controlado: Si múltiples hilos acceden y modifican las propiedades al mismo tiempo, puede haber condiciones de carrera.
- Estado inconsistente: Los datos pueden corromperse si no se sincroniza correctamente el acceso.
- Falta de aislamiento: Todos los hilos tienen acceso a la misma instancia, y cualquier cambio afecta a todos.

Solucion Usar actores como comentamos antes
En Swift moderno, puedes reemplazar el Singleton tradicional con un actor para manejar la concurrencia automáticamente.

