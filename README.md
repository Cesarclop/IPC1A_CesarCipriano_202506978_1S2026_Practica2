# IPC1A_CesarCipriano_202506978_1S2026_Practica2


# Manual Técnico – Visualizador de Algoritmos de Ordenamiento
## 1. Estructura del proyecto


- Visualizador ordenamiento
	- Main.java | Punto de entrada  

- gui - Paneles 
	- VentanaPrincipal.java | JFrame con CardLayout (menú, visualizador, reportes) 
	- PanelMenu.java | Pantalla de bienvenida con navegación  
	- PanelVisualizador.java | Panel principal: controles, gráfica, log, estadísticas  
	- HiloOrdenamiento.java | Hilo que ejecuta los algoritmos en segundo plano  
	- PanelReportes.java | Tabla de historial y generación de HTML  

- Reportes
	- Ejecucion.java | POJO que almacena los datos de una ejecución  
	- DatosSistema.java | Utilidad para obtener perfil del sistema (OSHI + Runtime)  


## 2. Librerías externas utilizadas

| Librería   | Versión | Propósito                                          |
| ---------- | ------- | -------------------------------------------------- |
| JFreeChart | 1.5.5   | Gráfico de barras dinámico.                        |
| OSHI Core  | 6.10.0  | Obtención de modelo y frecuencia del procesador.   |
| JNA        | 5.18.1  | Acceso nativo a APIs del sistema (usado por OSHI). |
| SLF4J API  | 2.0.17  | Logging (usado por OSHI).                          |

## 3. Requisitos técnicos cumplidos

Manual Técnico – Visualizador de Algoritmos de Ordenamiento
- **Lenguaje:** Java 11 (o superior).
- **Interfaz gráfica:** Swing (JFrame, JPanel, JButton, JComboBox, JTextArea, JTable, etc.).
- **Estructuras de datos:** exclusivamente `int[]` (no se usan ArrayList, Vector ni colecciones).
- **Algoritmos implementados manualmente:** Bubble Sort, Shell Sort, Quick Sort (recursivo).
- **Concurrencia:** hilo propio (`extends Thread`) con `Thread.sleep()` para animación. Todas las actualizaciones de UI se realizan mediante `SwingUtilities.invokeLater()`.
- **Visualización gráfica:** JFreeChart con `BarRenderer` personalizado para colores dinámicos.
- **Perfil del sistema:** `Runtime.getRuntime()`, `System.getProperty()` y OSHI.
- **Generación de números aleatorios:** `Math.random()`.

## 4. Lógica general de la aplicación

La aplicación consta de tres pantallas gestionadas con `CardLayout`:

1. **Menú principal (`PanelMenu`)**: Ofrece tres botones para navegar a Visualizador, Reportes y Ayuda.
2. **Visualizador (`PanelVisualizador`)**: Es el núcleo de la práctica. Permite:
   - Cargar datos desde ingreso manual (separados por comas), desde un archivo `.txt` o de forma aleatoria (rango 1-100).
   - Seleccionar el algoritmo (Bubble Sort, Shell Sort, Quick Sort), el orden (ascendente/descendente) y la velocidad de animación (Lento/Medio/Rápido).
   - Visualizar el arreglo mediante un gráfico de barras con colores que indican el estado de cada elemento (azul=normal, amarillo=comparando, rojo=intercambiando, verde=ordenado).
   - Un área de log que muestra cada operación (comparaciones e intercambios).
   - Estadísticas actualizadas en tiempo real (comparaciones, intercambios, iteraciones).
   - Al presionar "Iniciar ordenamiento", se clona el arreglo actual, se guardan los datos para el reporte y se lanza un hilo secundario (`HiloOrdenamiento`) que ejecuta el algoritmo mientras actualiza la interfaz.
3. **Reportes (`PanelReportes`)**: Muestra una tabla con el historial de todas las ejecuciones de la sesión actual. Cada vez que se completa un ordenamiento, se genera un archivo HTML en la carpeta `reportes/` con el resumen de la ejecución (algoritmo, orden, tamaño, comparaciones, intercambios, iteraciones, tiempo, arreglo original, arreglo ordenado, velocidad y perfil del sistema).

El uso de `CardLayout` permite que los paneles no se destruyan al navegar, por lo que el hilo de ordenamiento sigue ejecutándose incluso si el usuario cambia al menú o a reportes. Al regresar al visualizador, se ve el progreso actual.

## 5. Explicación detallada de métodos importantes

### 5.1 Clase `PanelVisualizador`

| Método | Descripción detallada |
|--------|----------------------|
| `crearComponentes()` | Inicializa todos los componentes Swing: campos de texto, botones, combos, área de log, etiquetas de estadísticas y el gráfico JFreeChart. Configura un `BarRenderer` personalizado que sobrescribe `getItemPaint()` para devolver el color de cada barra según el arreglo `estadoColores`. Además, activa las etiquetas de valores sobre las barras (`StandardCategoryItemLabelGenerator`). |
| `construirLayout()` | Organiza la interfaz usando `BorderLayout` en el panel principal. El panel superior (`panelSuperior`) tiene tres regiones: `WEST` (controles), `CENTER` (estadísticas), `EAST` (log). Los controles se colocan con `GridBagLayout` mediante el método auxiliar `agregarA()`. Cada sección se envuelve con `envolverConTitulo()` para añadir un título centrado sin usar `TitledBorder`. |
| `iniciarOrdenamiento()` | Valida que haya un arreglo cargado. Guarda una copia del arreglo original (`arregloOriginal`), el algoritmo seleccionado, el orden, la velocidad y el tiempo de inicio. Deshabilita todos los controles (`habilitarControles(false)`). Clona el arreglo actual (`copiaParaOrdenar`) y crea una instancia de `HiloOrdenamiento` pasándole la referencia del panel, la copia, el algoritmo, el orden y el retardo. Luego inicia el hilo. |
| `actualizarComparacion(int posA, int posB)` | Llama a `setColorBarra()` para marcar las posiciones `posA` y `posB` con estado `1` (amarillo). Luego llama a `refrescarGrafica()` para forzar el repintado y registra el evento en el log. Este método es llamado desde el hilo dentro de `invokeLater()`. |
| `actualizarIntercambio(int posA, int posB)` | Similar a `actualizarComparacion`, pero marca estado `2` (rojo). |
| `limpiarColores(int posA, int posB)` | Restaura a `0` (azul) las barras en las posiciones indicadas, a menos que ya estuvieran en `3` (verde). Luego llama a `refrescarGrafica()`. |
| `marcarOrdenado(int indice)` | Cambia el estado de la barra en `indice` a `3` (verde) y refresca la gráfica. |
| `refrescarGrafica()` | Llama a `actualizarGrafica(arreglo)`. Este método recarga el dataset de JFreeChart con los valores actuales del arreglo, lo que provoca que el renderer personalizado reconsulte los colores. |
| `finalizarOrdenamiento(int[] arregloFinal, int comp, int inter, int iter)` | Calcula el tiempo total transcurrido. Convierte el arreglo original y el final a cadenas legibles. Obtiene el perfil del sistema llamando a `reportes.DatosSistema.obtenerPerfil()`. Obtiene la referencia al `PanelReportes` a través de la ventana principal. Crea un objeto `Ejecucion` con todos los datos y lo envía a `panelReportes.agregarEjecucion()`. |
| `aplicarEstilos()` | Establece colores de fondo personalizados para el panel principal, los subpaneles y la gráfica. Configura el fondo del `ChartPanel`, del gráfico, del área de trazado, de las líneas de cuadrícula y de la leyenda. También ajusta el color de las etiquetas de los ejes. |

### 5.2 Clase `HiloOrdenamiento`

| Método | Descripción detallada |
|--------|----------------------|
| `run()` | Primero, mediante `invokeLater()`, reinicia las estadísticas y el log, y añade el mensaje de inicio. Luego, según el algoritmo seleccionado, llama a `ejecutarBubbleSort()`, `ejecutarShellSort()` o `ejecutarQuickSort()`. Finalmente, en otro `invokeLater()`, marca todas las barras como ordenadas, llama a `finalizarOrdenamiento()` para generar el reporte, añade el mensaje de finalización y vuelve a habilitar los controles. |
| `ejecutarBubbleSort()` | Implementación clásica con dos bucles anidados. En cada comparación incrementa `comparaciones` y llama a `marcarComparacion()`. Si es necesario intercambiar, incrementa `intercambios`, llama a `marcarIntercambio()`, realiza el intercambio, refresca la gráfica y espera. Después de cada comparación o intercambio, limpia los colores y espera. Al final de cada pasada, marca como ordenado el último elemento. |
| `ejecutarShellSort()` | Utiliza una secuencia de gaps empezando por `n/2` y dividiendo entre 2 hasta llegar a 1. Para cada gap, recorre el arreglo desde `i = gap` hasta el final, y realiza una inserción dentro de la subsecuencia de paso `gap`. Compara `arreglo[j-gap]` con `valorActual` y desplaza si corresponde. Cada comparación e intercambio actualiza la UI mediante los métodos auxiliares. No marca elementos durante la ejecución; al final, el hilo principal marca todos como verdes. |
| `ejecutarQuickSort(int inicio, int fin)` | Método recursivo. Si `inicio >= fin` retorna. Incrementa `iteraciones` y actualiza la UI. Llama a `particionar()` para obtener la posición final del pivote, luego marca ese pivote como ordenado. Finalmente, ordena recursivamente la sublista izquierda y derecha. |
| `particionar(int inicio, int fin)` | Elige el pivote como `arreglo[fin]`. Recorre `j` desde `inicio` hasta `fin-1`. Mantiene un índice `i` que delimita la zona izquierda. Si el elemento actual debe ir a la izquierda (según orden), incrementa `i` e intercambia `arreglo[i]` con `arreglo[j]`. Cada comparación e intercambio se visualiza. Al final, coloca el pivote en `i+1` e intercambia con `arreglo[fin]`. Devuelve `i+1`. |
| `marcarComparacion()`, `marcarIntercambio()`, `limpiarColores()`, `refrescarGrafica()`, `actualizarEstadisticasUI()` | Todos estos métodos encapsulan una llamada a `SwingUtilities.invokeLater()` que a su vez llama al método correspondiente del panel. Esto garantiza que las actualizaciones de la interfaz se ejecuten en el EDT. |
| `esperar()` | Llama a `Thread.sleep(retardoMs)` y captura `InterruptedException` para restaurar el estado de interrupción. |

### 5.3 Clase `PanelReportes`

| Método | Descripción detallada |
|--------|----------------------|
| `agregarEjecucion(Ejecucion e)` | Incrementa el contador de ejecuciones, añade una fila al `DefaultTableModel` de la tabla (con los datos de la ejecución) y llama a `generarArchivoHTML(e)`. |
| `generarArchivoHTML(Ejecucion e)` | Crea la carpeta `reportes/` si no existe. Genera un nombre de archivo único con fecha y número de ejecución. Escribe un archivo HTML simple (sin estilos complejos) que incluye: número de ejecución, fecha, tabla de resumen, arreglo original, arreglo ordenado y perfil del sistema. |

### 5.4 Clase `DatosSistema`

| Método | Descripción detallada |
|--------|----------------------|
| `obtenerPerfil()` | Construye una cadena con: `os.name` y `os.arch`; núcleos (`Runtime.availableProcessors()`); RAM total y disponible de la JVM (`Runtime.totalMemory()` y `freeMemory()`). Luego, dentro de un bloque `try-catch`, usa OSHI para obtener el modelo del procesador (`cpu.getProcessorIdentifier().getName()`), la frecuencia (`getVendorFreq() / 1_000_000`) y la RAM física total (`hal.getMemory().getTotal()`). Si ocurre una excepción, añade un mensaje de error. |

## 6. Gestión de colores en la gráfica

- Se mantiene un arreglo `estadoColores` de tipo `int[]` del mismo tamaño que el arreglo visualizado.
- El `BarRenderer` personalizado sobrescribe `getItemPaint(int serie, int indice)`. Este método consulta `estadoColores[indice]` y devuelve:
  - `0` → azul (`new Color(50, 130, 220)`)
  - `1` → amarillo (`new Color(255, 220, 0)`)
  - `2` → rojo (`new Color(220, 50, 50)`)
  - `3` → verde (`new Color(50, 200, 50)`)
- Cada vez que se cambia un estado (comparación, intercambio, ordenado), se modifica `estadoColores` y se llama a `refrescarGrafica()`, que a su vez llama a `actualizarGrafica(arreglo)`. `actualizarGrafica` recarga el dataset de JFreeChart (`datosGrafica.clear()` + `addValue`), lo que provoca un repintado completo y el renderer vuelve a ejecutar `getItemPaint()` para cada barra.

## 7. Flujo de generación de reportes

1. En `PanelVisualizador.iniciarOrdenamiento()` se guardan:
   - `arregloOriginal = arreglo.clone()`
   - `algoritmoUsado`, `esAscendente`, `retardoMs`, `tiempoInicio`.
2. El hilo `HiloOrdenamiento` ejecuta el algoritmo y al finalizar llama a:
   `panel.finalizarOrdenamiento(arreglo, comparaciones, intercambios, iteraciones)`.
3. `finalizarOrdenamiento` calcula el tiempo total (`System.currentTimeMillis() - tiempoInicio`), convierte arreglos a texto, obtiene el perfil del sistema con `DatosSistema.obtenerPerfil()` y crea un objeto `Ejecucion`.
4. Se invoca `panelReportes.agregarEjecucion(ejecucion)`.
5. `agregarEjecucion` añade una fila a la tabla y genera un archivo HTML en la carpeta `reportes/` con el resumen.

## 8. Observaciones de implementación

- **Uso exclusivo de `int[]`:** No se utilizan `ArrayList`, `Vector` ni ninguna otra colección. Incluso en `PanelReportes` se evita `ArrayList` para almacenar el historial; la tabla usa `DefaultTableModel` y el contador `totalEjecuciones` es suficiente.
- **Recursión en Quick Sort:** La implementación es recursiva real, no simulada con pila, como exige el enunciado.
- **Hilos y EDT:** Todas las modificaciones de componentes Swing (cambios de texto, repintados, etc.) se realizan dentro de `SwingUtilities.invokeLater()`. Esto evita bloqueos y errores de concurrencia.
- **Persistencia entre paneles:** Gracias a `CardLayout`, los paneles no se destruyen al navegar. El hilo de ordenamiento sigue ejecutándose incluso si el visualizador no está visible, porque el hilo es independiente del EDT.
- **Formato de archivo de entrada:** Se admiten números separados por comas, espacios o saltos de línea. La función `split("[,\\s]+")` maneja estos separadores.
- **Perfil del sistema:** OSHI proporciona información detallada del hardware; si falla (por ejemplo, en sistemas operativos no soportados), se captura la excepción y se muestra un mensaje genérico. Esto evita que la aplicación se detenga.
- **Nombres de variables:** Se han utilizado nombres cortos pero significativos (`gap`, `pivote`, `indiceMenor`, `retardoMs`), facilitando la comprensión del código.

## 9. Compilación y ejecución

- **Requisito:** JDK 11+, Maven.
- **Compilar:** `mvn clean compile`
- **Ejecutar:** `mvn exec:java` o ejecutar la clase `Main` desde el IDE.

## 10. Posibles mejoras futuras (opcionales)

- Permitir pausar/detener el ordenamiento manualmente.
- Agregar más algoritmos (Selection Sort, Insertion Sort, Merge Sort).
- Guardar el historial entre sesiones (serialización o base de datos).
- Personalizar colores y fuente mediante un archivo de configuración.
- Exportar el historial a CSV o PDF.

---

 *Práctica 2 – Visualizador de algoritmos de ordenamiento.*

---
---

 # Manual de Usuario – Visualizador de Algoritmos de Ordenamiento

## 1. Introducción

El **Visualizador de Algoritmos de Ordenamiento** es una aplicación de escritorio que permite cargar listas de números, ejecutar diferentes algoritmos de ordenamiento (Bubble Sort, Shell Sort, Quick Sort) y ver en tiempo real cómo se reordenan los elementos mediante una gráfica de barras animada. Además, la aplicación genera reportes automáticos con estadísticas y el perfil del sistema.

## 2. Requisitos del sistema

- Sistema operativo: Windows, Linux o macOS.
- Java Runtime Environment (JRE) 11 o superior.
- Resolución de pantalla recomendada: 1280×720 o superior.

## 3. Inicio de la aplicación

![[Pasted image 20260405153451.png]]  

- **Visualizador**: accede a la pantalla de ordenamiento.
- **Reportes**: muestra el historial de ejecuciones y genera reportes HTML.
- **Ayuda / Información**: muestra una breve descripción de los algoritmos y las instrucciones básicas.

## 4. Pantalla del visualizador

Haga clic en **Visualizador** para ingresar. La pantalla se divide en cuatro secciones:

![[Pasted image 20260405154555.png]]


1. **Panel de controles** (izquierda): permite ingresar datos, elegir algoritmo, orden y velocidad.
2. **Panel de estadísticas** (centro): muestra comparaciones, intercambios e iteraciones en tiempo real.
3. **Log de operaciones** (derecha): bitácora con cada paso del algoritmo.
4. **Gráfica de barras** (centro inferior): representa el arreglo; el color de cada barra indica su estado:
   - 🔵 Azul: elemento en reposo (no está siendo comparado ni intercambiado, y aún no está ordenado).
   - 🟡 Amarillo: elemento que se está comparando.
   - 🔴 Rojo: elemento que se está intercambiando.
   - 🟢 Verde: elemento ya colocado en su posición final (ordenado).

## 5. Carga de datos

Puede ingresar números de tres maneras:

### 5.1. Ingreso manual
- Escriba los números en el campo **Ingreso manual**, separados por comas o espacios.  
  Ejemplo: `5, 2, 8, 1, 9`
- Haga clic en el botón **Cargar**. La gráfica mostrará las barras correspondientes.

### 5.2. Carga desde archivo de texto
- Haga clic en **Cargar desde archivo**.
- Seleccione un archivo `.txt` que contenga números separados por comas, espacios o saltos de línea.  
  Ejemplo de contenido del archivo:  
  `10 3 7 1 9`  
  o  
  `10,3,7,1,9`
- Al abrirlo, el arreglo se cargará automáticamente.

### 5.3. Generación aleatoria
- En el campo **Cantidad**, elija cuántos números desea (entre 5 y 30).
- Haga clic en **Aleatorio**. Se generarán números enteros entre 1 y 100.

## 6. Configuración del ordenamiento

- **Algoritmo**: elija entre `Bubble Sort`, `Shell Sort` o `Quick Sort`.
- **Orden**: seleccione `Ascendente` (de menor a mayor) o `Descendente` (de mayor a menor).
- **Velocidad**: puede elegir `Lento` (500 ms entre pasos), `Medio` (100 ms) o `Rápido` (20 ms).

## 7. Iniciar el ordenamiento

Una vez cargado un arreglo, presione el botón **Iniciar ordenamiento**. Durante la ejecución:

- Los controles se deshabilitan para evitar interferencias.
- La gráfica muestra los cambios de color según cada paso.
- Las estadísticas y el log se actualizan automáticamente.
- Puede cambiar de pantalla (por ejemplo, ir al menú o a reportes) y el proceso continúa en segundo plano. Al regresar, verá el progreso actual.

Al terminar, todas las barras se vuelven verdes y se genera un reporte automático (ver sección 8).

## 8. Reportes

Desde el menú principal, haga clic en **Reportes**. Aparecerá una tabla con el historial de todas las ejecuciones de la sesión actual (los datos se pierden al cerrar la aplicación).

![[Pasted image 20260405155815.png]]

Cada fila contiene:
- Número de ejecución (#)
- Algoritmo usado
- Orden (Ascendente/Descendente)
- Tamaño del arreglo (n)
- Comparaciones, intercambios e iteraciones realizadas
- Tiempo total en milisegundos

Además, **por cada ejecución se genera automáticamente un archivo HTML** en la carpeta `reportes/` (dentro de la misma ubicación del programa). El archivo incluye:
- Resumen con todos los datos de la tabla.
- Arreglo original y arreglo ordenado.
- Perfil completo del sistema (sistema operativo, procesador, núcleos, RAM, etc.)

Puede abrir estos archivos con cualquier navegador web.

## 9. Botón Ayuda / Información

Al hacer clic en el botón **Ayuda / Información** del menú principal, se abre una ventana emergente que muestra una breve descripción de los algoritmos de ordenamiento implementados (Bubble Sort, Shell Sort, Quick Sort) y las instrucciones básicas para utilizar la aplicación.

![[Pasted image 20260405160308.png]]

## 10. Botón "Regresar al Menú"

En todas las pantallas (visualizador y reportes) encontrará un botón en la parte inferior derecha para volver al menú principal.

## 11. Ejemplo de uso

**Objetivo:** Ordenar los números `[3, 1, 4, 1, 5, 9, 2]` de forma ascendente con Bubble Sort a velocidad media.

1. En el visualizador, escriba en el campo manual: `3,1,4,1,5,9,2`
2. Presione **Cargar** (las barras se dibujan).
3. Seleccione **Bubble Sort**, **Ascendente** y **Medio**.
4. Presione **Iniciar ordenamiento**.
5. Observe cómo las barras se comparan (amarillo) y se intercambian (rojo) hasta que todas quedan verdes.
6. Revise el log para ver cada paso.
7. Vaya al menú y luego a **Reportes**; debería ver una fila con los datos de esta ejecución.
8. Busque la carpeta `reportes/` y abra el archivo HTML generado (por ejemplo `reporte_20250401_1.html`).

## 12. Solución de problemas

| Problema | Posible solución |
|----------|------------------|
| Al hacer clic en "Iniciar ordenamiento" no pasa nada | Asegúrese de que haya cargado un arreglo (campo no vacío, archivo válido o números aleatorios). |
| La gráfica no se actualiza o se ve extraña | Verifique que las librerías JFreeChart estén correctamente instaladas. |
| No se generan los reportes HTML | Compruebe que la aplicación tenga permisos de escritura en la carpeta donde se ejecuta. La carpeta `reportes/` se creará automáticamente. |
| El programa se congela durante el ordenamiento | Esto no debería ocurrir, pero si sucede, reinicie la aplicación y seleccione una velocidad más lenta. |


