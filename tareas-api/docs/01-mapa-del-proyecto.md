# Mapa del Proyecto: tareas-api

## 1. Qué hace el proyecto

Gestor de tareas pendientes en Java 17 para la mentoría de Generation México (CH70 Java). Permite crear, completar, eliminar y consultar tareas con almacenamiento en memoria sin base de datos.

## 2. Tabla de clases

| Clase | Responsabilidad | Depende de |
|-------|-----------------|------------|
| **Prioridad** | Enum que define los niveles de prioridad de las tareas: BAJA, MEDIA, ALTA | Ninguna |
| **Tarea** | Modelo de dominio que representa una tarea con id, título, descripción, prioridad, fecha límite y estado de completado | Prioridad |
| **TareaNoEncontradaException** | Excepción personalizada que se lanza cuando se busca una tarea con un id inexistente | Ninguna |
| **TareaRepositorio** | Capa de datos que maneja el almacenamiento en memoria con LinkedHashMap, asigna ids consecutivos y proporciona operaciones CRUD | Tarea |
| **TareaServicio** | Capa de lógica de negocio que orquesta casos de uso: crear, completar, eliminar, listar tareas pendientes, filtrar por prioridad, calcular días restantes y generar reportes | TareaRepositorio, Tarea, Prioridad, TareaNoEncontradaException |
| **App** | Punto de entrada de consola que ejecuta una demo creando tareas de ejemplo y mostrando el reporte | TareaServicio, TareaRepositorio, Prioridad |
| **TareaRepositorioTest** | Suite de pruebas unitarias para TareaRepositorio usando JUnit 5 | TareaRepositorio, Tarea, Prioridad |
| **TareaServicioTest** | Suite de pruebas unitarias para TareaServicio usando JUnit 5 con fecha fija para reproducibilidad | TareaServicio, TareaRepositorio, Tarea, Prioridad |

## 3. Cómo se compila y se prueba

```bash
# Compilar el proyecto
mvn compile

# Ejecutar todas las pruebas
mvn test

# Ejecutar una clase de prueba específica
mvn test -Dtest=TareaRepositorioTest
mvn test -Dtest=TareaServicioTest

# Ejecutar la demo de consola (App.java)
mvn -q exec:java

# Limpiar artefactos de compilación
mvn clean
```

## 4. Tres cosas sospechosas o incompletas

### 1. TareaRepositorio.java:44 - Búsqueda distingue mayúsculas cuando no debería

El método `buscarPorTitulo()` dice en su Javadoc (línea 38) que busca "sin distinguir mayúsculas de minúsculas", pero la implementación en la línea 44 usa `.contains(texto)` que SÍ las distingue. Por eso buscar "informe" no encuentra "INFORME". Hay una prueba deshabilitada con `@Disabled` en TareaRepositorioTest:38 que documenta este fallo.

### 2. TareaServicio.java:51 - Filtro de prioridad excluye el nivel mínimo solicitado

El método `listarPorPrioridadMinima()` usa el operador `>` en lugar de `>=` al comparar prioridades. Esto hace que cuando pides tareas con prioridad MEDIA o superior, solo devuelve las de prioridad ALTA, excluyendo incorrectamente las MEDIA. El Javadoc en líneas 45-46 dice explícitamente que con MEDIA debería devolver MEDIA y ALTA.

### 3. TareaServicio.java:67 - Cálculo de días restantes devuelve valores invertidos

El método `diasRestantes()` tiene los parámetros de `ChronoUnit.DAYS.between()` en orden incorrecto. Usa `between(fechaLimite, hoy)` cuando debería ser `between(hoy, fechaLimite)`, lo que provoca que devuelva valores negativos cuando deberían ser positivos y viceversa. La prueba en TareaServicioTest:64 espera `-3` cuando el comentario en línea 63 dice "Vence en 3 días", confirmando que el valor esperado está adaptado al bug en lugar de ser correcto.