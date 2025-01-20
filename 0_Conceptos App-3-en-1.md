
## COMANDOS
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
- `WebImage` (imagenes sacadas de la web)
- `ProgressView` (spinner de carga)
- `Chart` y `SectorMark` (gráfica de circulo) y leyendas.

# APP 3 (Sitios Fav Mapas)
`Mapa`
`Dialogo`
`Formulario`
`Botón superpuesto`
`Sheet`
`Listado horizontal`
