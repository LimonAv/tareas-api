# CLAUDE.md

Este archivo proporciona orientación a Claude Code (claude.ai/code) cuando trabaja con código en este repositorio.

## Descripción del Proyecto

**tareas-api**: Gestor de tareas pendientes en Java 17 para la mentoría de Generation México (CH70 Java, octubre 2026). Almacenamiento en memoria, sin base de datos.

## Requisitos

- Java 17+ (`java -version`)
- Maven 3.8+ (`mvn -v`)

## Comandos

```bash
# Ejecutar todas las pruebas
mvn test

# Ejecutar clase de prueba específica
mvn test -Dtest=TareaRepositorioTest
mvn test -Dtest=TareaServicioTest

# Ejecutar demo de consola (App.java)
mvn -q exec:java

# Limpiar artefactos de compilación
mvn clean

# Solo compilar
mvn compile
```

## Arquitectura

**Diseño por capas:**
```
Tarea (modelo)
  ↓
TareaRepositorio (capa de datos)
  ↓
TareaServicio (lógica de negocio)
  ↓
App (punto de entrada consola)
```

**Patrones clave:**
- **Repository**: Almacenamiento en memoria usando `LinkedHashMap<Integer, Tarea>`. IDs asignados secuencialmente desde 1. Sin persistencia entre ejecuciones.
- **Capa de servicio**: Lógica de negocio y orquestación. Lanza `TareaNoEncontradaException` cuando el ID no existe.
- **Inmutabilidad**: Los campos de Tarea son mayormente `final`. El estado (completada) es el único campo mutable.

**Configuración de pruebas:**
- JUnit 5 (Jupiter)
- `@BeforeEach` crea instancia fresca del servicio por prueba
- Fecha fija `LocalDate.of(2026, 10, 8)` en TareaServicioTest para reproducibilidad

## Bugs Conocidos y TODOs

Este es un **proyecto de aprendizaje** con bugs intencionales para práctica:

1. **TareaRepositorio.buscarPorTitulo()** (línea 44):
   - Javadoc dice "sin distinguir mayúsculas de minúsculas"
   - La implementación usa `.contains()` que SÍ distingue mayúsculas
   - Prueba está `@Disabled` en TareaRepositorioTest:38
   - Corrección: usar `.toLowerCase().contains(texto.toLowerCase())`

2. **TareaServicio.listarPorPrioridadMinima()** (línea 51):
   - Usa `>` en lugar de `>=`, excluyendo la prioridad mínima misma
   - Pedir MEDIA devuelve solo ALTA, no MEDIA+ALTA
   - Corrección: cambiar a `>=`

3. **TareaServicio.diasRestantes()** (línea 67):
   - Parámetros invertidos en `ChronoUnit.DAYS.between(fechaLimite, hoy)`
   - Devuelve negativo cuando debería ser positivo (ver prueba línea 64 espera -3)
   - Corrección: cambiar a `between(hoy, fechaLimite)`

4. **Constructor de Tarea** (línea 19):
   - Comentario TODO: validar que título no esté vacío ni en blanco
   - Actualmente acepta `""`

5. **TareaServicio.completar()** (línea 25):
   - No verifica si `repositorio.buscar()` devuelve null
   - Lanzará NullPointerException en lugar de TareaNoEncontradaException
   - Prueba faltante anotada en TareaServicioTest:39

6. **Cobertura de pruebas faltante** (TareaServicioTest:102):
   - No hay pruebas para `eliminar(int)`

## Estructura de Paquetes

Todo el código en: `mx.generation.tareas`

**Modelo:**
- `Tarea` - entidad con id, titulo, descripcion, prioridad, fechaLimite, completada
- `Prioridad` - enum: BAJA, MEDIA, ALTA

**Datos:**
- `TareaRepositorio` - operaciones CRUD en memoria

**Negocio:**
- `TareaServicio` - casos de uso (crear, completar, eliminar, listar, reportes)
- `TareaNoEncontradaException` - RuntimeException personalizada

**Entrada:**
- `App` - demo de consola con datos de ejemplo