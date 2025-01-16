

##  SWIFT BASADO EN `PROTOCOLOS`
se basa siempre en `contratos` que tu le dices al sistema que tiene que `cumplir` e incluso puede llegar a incorporar ciertas funcionalidades concretas.

## Swift UI funciona con el concepto de `MVVM`
 Se cogen los `eventos` y se traspasaban de un lado a otro, es decir, por un lado tengo la parte del código que me da la `funcionalidad` de la aplicación (lo que seria la `lógica de negocio`) y por otro lado seria la `vista` que es lo que ve el `usuario`. Entonces la `conexión` entre la lógica de usuario y la vista estaba realizada.

DIFERENCIA ENTRE `UIKIT` CON `MVC` Y `SWIFT UI` CON `MVVM`.

## PASO PARCIAL ACTUAL DE LIBRERÍA DE `COMBINE` A LA LIBRERÍA DE `Swift Macros`

## `Clearn Architecture` que es un tipo de `arquitectura`

## Struct, extensiones, propiedad calculada, protocolos Codable, Identifiable, Hashable (EN EL MODELO)
- Es una extensión de `struct StarCardDTO` (por lo que se podria poner en un fichero aparte). Todo lo que se ponga aquí es algo que se va a incluir a lo que se tiene en el `struct StarCardDTO`
- toCard: es una propiedad calculada, que no almacena valores sino que cada vez que se la llama devuelve algo que calcula o transforma en tiempo real (como un patrón setter - getter en Java)
- Protocolo 'Codable' para que pueda cargarse la información del json y transformarse en instancias.
- Protocolo 'Identifiable': va a tener un campo 'id' que sera igualable, es decir único.
- Protocolo 'Hashable': permite tener un valor de hash, un valor de comprobación único. Cuando en todas sus estancias sean iguales es cuando se va a poder comparar por igualdad. Es decir pueden ser dos instancias diferentes pero si tienen todos los mismos valores los compararía como iguales.

## Creación de protocolos, extensión a protocolo, struct con dicho protocolo (EN EL REPOSITORIO)
- Creamos un protocolo para aislar a un variable la URL, para que se pueda inyectar un valor distinto de URL y poder itercambiar de forma sencilla entre el json de producción y de testing
El struct tiene el protocolo personalizado el cual tiene los métodos 'url' y 'getData()' (la extensión), la variable 'url' como sucede en los protocolos obliga a implementarla  en el struct al que se llama
- ¿SI EN VEZ D EUN JSONFUERA UNA API SERIA IGUAL?

## Observable, final class, variable con protocolo personalizado, init (EN EL VISTA-MODELO)
- Se anota la clase de tipo observable, que es lo que me va a permitir que cualquier propiedad variable que haya en la clase cuando se actualice la clase al ponerla en una vista se refresque la vista
- Protocolo DataRepository, que lo curioso es que tambien es un tipo de dato. Y dentro de la variable va a estar tanto la instancia del Struct de 'Repository' como de 'RepositoryTest', las cuales tienen el protocolo DataRepository. Basicamente la viarable va apoder acceder a todos los datos de que tiene dicho protocolo.
- Creamos el inicializador que ponemos que por defecto reciba Repository() (producción) pero se va a poder cambiar en la vista donde se invoca (ContentView.swift) por el de RepositoryTest (desarrollo) de manera muy sencilla y escalable.
- final class: ¿Poruae no puede ser un actor o struct?

## Llamada a struct (como tipo de dato, que herera a las extensiones) (EN LA VISTA SECUNDARIA)
- Cargamos ya una instancia de una variable llamada 'test' (definida antes en 'Model.swift') donde la variable es una una extensión StarCard (struct). Para la Preview

##  @State, #Preview (EN LA VISTA)
- @State: indica al sistema que cuando cambie ciertas propiedades provocara que la vista se actualice. En este caso el observable StarCardVM() el cual tiene la variable cards en el momento que esta cambie automaticamente refrescará la pantalla y lo cambiará
- #Preview, esto lo que hace es dar un contexto en el que creo la instancia de 'ContentView' para que se vea en el simulador de la derecha de Xcode.

## @testable, Suite Testing, .tags (EN EL TEST)
-  es mi aplicación StarWarsDemo (import StarWarsDemo), pero añadimos el @testable, para que acceda a todos los componentes de nuestra aplicación y se pueda invocar dichos componentes
-  No son Struct sino Suite los test
- puede haber diferentes elementos como para deshabilitar que este en el ipad, etc. Además del elemento etiqueta que ponemos para decir que es un Suite del repositorio nuestro aplicación el test que estamos realizando

### ¿SE DEBERIA DE IMPLEMENTAR ACTORES EN ALGUN MOMENTO?
