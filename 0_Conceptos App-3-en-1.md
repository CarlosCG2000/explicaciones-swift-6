
# COMANDOS
Para organizar el código pillamos todo el código (`cmd + a`) y pulsamos (`ctrl + i`)
Para crear un fichero (`command + n`)
Para duplicar la línea (`comando + d`)
Para las ayudas de Apple (`option`)

# CONFIGURACIÓN GENERAL Y COLORES E IMÁGENES GLOBALES DE LA APP
* Ejecutable de la aplicación (congiguración general)
* `Assets` (colores e imágenes globales de la app)

# PÁGINAS PRINCIPALES
* `__Curso_iOSApp`: `punto de entrada` de la proyecto (configuración general del proyecto, ejemplo: el aspecto de la barra superior (`Toolbar`))
* `MainView`: es la `primera vista` (la principal), solo llama a la primera vista funcional que es el menu (`MenuView`).
* `MenuView`: es la primera `vista funcional`, donde se encuentra el `menú` para navegar a las aplicaciones (`IMC`, `SuperHeroe`, `Mis Sitios`).

# APP 1 (IMC)
Propiedad `@State` --> para que se reflejen los cambios de las vistas secundarias (@Binding) de forma automatica, añadir si se modifica en esta vista principal el '$'.

- `VStack` y `HStack`
- `Button` y `Image` (como icono con SF)
- `Slider`
- `Navegación con paso de parámetros`

[Buena idea (que no esta en el proyecto como tal)] módular en un fichero las vistas (`Struct`) de como van a ser los `textos` para simplemente llamarlo durante todo el proyecto y no tener que definir constantemente su aspecto. Tambien se puede hacer con `botones`, etc... Y asi lo tengo todo reutilizable.

- `Función` con uso de `switch` y `tuplas` para devolver varios valores de diferente tipo según un dato.

# APP 2 (SuperHero)
- Llamada a datos de una `API` (lo consideraría como la `presentacion (módelo)` y la `lógica del módelo (repositorio)`) en este caso en un mismo fichero estan.
- `guard` (para protección al contrario que un if, devuelve si la condición es 'false').

- `Task`: la tarea permite ejecutar código de forma asincrónica (en segundo plano) sin bloquear el hilo principal, parando dicho hilo solo con el uso de await. En este caso para llamar a la funciones de la API.
Forma parte del modelo de `concurrencia` introducido en `Swift 5.5` y que se usa para ejecutar código de forma `asincrónica` y `concurrente`.
- ` if let results = wrapper?.results{ ... }`: desempaquetado opcional.
- `List` con `Navegación`
- `WebImage` (imagenes sacadas de la web) --> Instalar librerías `SDWebImage`
- `ProgressView` (spinner de carga)
- `Chart` y `SectorMark` (gráfica de circulo) y leyendas.

# APP 3 (Sitios Fav Mapas)
- `Codificar` y `decodificar` modelo para usarlo en BD de `UserDefault`.
- `Almacenar` datos en formato DB de `UserDefaults`.
- `Mapa`: mapa de Apple.
- `Dialogo`: dialogo personalizado con el que se puede cerrar desde cualquier para de la pantalla si se desea y que contiene un formulario.
- `Formulario`: es un formulario el cual tiene validación de datos.
- `Botón superpuesto`: boton que se encuentra encima de la pantalla y sirve para abir ventana superpuesta.
- `Sheet`: ventana superpuesta que se abre hasta el punto que se desee de la pantalla.
- `Listado horizontal`: listado en forma horizontal con scroll horizontal.

## Como seria la APP 3, usando `SwiftData` en vez de `UserDefault`:
1. `Place.swift`

• Si este modelo se usará con `SwiftData`, sería necesario marcarlo con` @Model`. Además, deberías usar propiedades `declarativas` compatibles con `SwiftData`. Por ejemplo:
• `CLLocationCoordinate2D` no es directamente compatible con `SwiftData`. En su lugar, puedes almacenar las coordenadas como propiedades separadas (latitude y longitude) y construir el `objeto CLLocationCoordinate2D` según sea necesario.
• Elimina la necesidad de usar `Codable` porque SwiftData se encargará de la `persistencia`.
• `latitude` y `longitude` se almacenan como propiedades separadas, lo que facilita la `persistencia` en bases de datos.

```js
@Model
final class Place {
    @Attribute(.unique) var id: UUID
    var name: String
    var latitude: Double
    var longitude: Double
    var fav: Bool
    // propiedad calculada
    var coordinates: CLLocationCoordinate2D {
        get {
            CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
        }
        set {
            latitude = newValue.latitude
            longitude = newValue.longitude
        }
    }

    init(id: UUID = UUID(), name: String, latitude: Double, longitude: Double, fav: Bool) {
        self.id = id
        self.name = name
        self.latitude = latitude
        self.longitude = longitude
        self.fav = fav
    }
}
```

`DecodingError` específico:
• Aunque ya verificas que las coordenadas son válidas, puedes hacer que el `debugDescription` del error sea aún más útil, indicando qué valores de `latitud/longitud` eran inválidos.

```js
throw DecodingError.dataCorruptedError(forKey: .latitude,
                                       in: container,
                                       debugDescription: "Latitud inválida (\(latitude)), debe estar entre -90 y 90. Longitud: (\(longitude)) debe estar entre -180 y 180.")
```

2. Configuración del `ModelContainer`
En tu archivo principal de la app (`App`), configura el `ModelContainer` para usar `SwiftData`:
```js
import SwiftUI
import SwiftData

@main
struct PlacesApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: [Place.self]) // Añadimos el modelo Place
    }
}
```

3. `Migrar` la lógica de `guardar` y `cargar`
Extiende `SitiosFavoritos` para manejar las operaciones de almacenamiento usando `SwiftData`. Aquí tienes las funciones de guardar y cargar usando un `ModelContext`:
```js
import SwiftData

extension SitiosFavoritos {
    func guardarPlaces(using context: ModelContext) {
        for place in places {
            context.insert(place)
        }
        do {
            try context.save()
        } catch {
            print("Error al guardar los lugares: \(error.localizedDescription)")
        }
    }

    func cargarPlaces(using context: ModelContext) {
        let fetchDescriptor = FetchDescriptor<Place>() // FetchDescriptor sin filtros trae todos los lugares
        do {
            places = try context.fetch(fetchDescriptor)
        } catch {
            print("Error al cargar los lugares: \(error.localizedDescription)")
            places = []
        }
    }
}
```

4. `Filtrar` lugares `favoritos`
Para trabajar con filtros, puedes usar un `FetchDescriptor` personalizado:
```js
func cargarFavoritos(using context: ModelContext) {
    let fetchDescriptor = FetchDescriptor<Place>(predicate: #Predicate { $0.fav == true })
    do {
        places = try context.fetch(fetchDescriptor)
    } catch {
        print("Error al cargar los lugares favoritos: \(error.localizedDescription)")
        places = []
    }
}
```

5. `Actualizar` la vista `ContentView`
En ContentView, usa las anotaciones `@Query` y `@Environment` para interactuar con el `ModelContext`:
```js
import SwiftUI
import SwiftData

struct ContentView: View {
    @Query private var places: [Place] // Consulta reactiva de SwiftData
    @Environment(\.modelContext) private var context // Contexto para operaciones

    var body: some View {
        NavigationStack {
            List {
                ForEach(places) { place in
                    HStack {
                        Text(place.name)
                        Spacer()
                        if place.fav {
                            Image(systemName: "star.fill")
                                .foregroundColor(.yellow)
                        }
                    }
                }
                .onDelete(perform: deletePlace)
            }
            .navigationTitle("Lugares")
            .toolbar {
                ToolbarItem(placement: .primaryAction) {
                    Button("Añadir") {
                        addPlace()
                    }
                }
            }
        }
    }

    private func addPlace() {
        let nuevoLugar = Place(name: "Nuevo Lugar",
                               latitude: 40.4168,
                               longitude: -3.7038,
                               fav: false)
        context.insert(nuevoLugar)
        do {
            try context.save()
        } catch {
            print("Error al guardar el lugar: \(error.localizedDescription)")
        }
    }

    private func deletePlace(at offsets: IndexSet) {
        for index in offsets {
            context.delete(places[index])
        }
        do {
            try context.save()
        } catch {
            print("Error al eliminar el lugar: \(error.localizedDescription)")
        }
    }
}
```

6. Probar datos de ejemplo
Si quieres usar datos de ejemplo durante el `desarrollo`, crea un `PreviewModifier`:
```js
import SwiftData

struct SampleData: PreviewModifier {
    static func makeSharedContext() async throws -> ModelContainer {
        let schema = Schema([Place.self])
        let container = try ModelContainer(for: schema)

        let place1 = Place(name: "Madrid", latitude: 40.4168, longitude: -3.7038, fav: true)
        let place2 = Place(name: "Barcelona", latitude: 41.3874, longitude: 2.1686, fav: false)

        container.mainContext.insert(place1)
        container.mainContext.insert(place2)

        return container
    }

    func body(content: Content, context: ModelContainer) -> some View {
        content.modelContainer(context)
    }
}

#Preview(traits: .modifier(SampleData())) {
    ContentView()
}
````
