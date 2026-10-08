# CLAUDE.md

## Stack
- **Java 17** (maven.compiler.release)
- **Maven** 3.8+, **JUnit 5.10.2** (Jupiter)
- Almacenamiento en memoria (sin base de datos)

## Comandos
```bash
mvn test              # Ejecutar todas las pruebas
mvn -q exec:java      # Correr la demo (App.java)
mvn compile           # Solo compilar
mvn clean             # Limpiar artefactos
```

## Convenciones Obligatorias
1. **Todo el código en español**: nombres de clases, métodos, variables, comentarios y mensajes
2. **Pruebas en español**: nombres de métodos de prueba descriptivos del comportamiento esperado
3. **JUnit 5**: usar anotaciones `@Test`, `@BeforeEach`, `@DisplayName` cuando mejore legibilidad
4. **Una prueba por comportamiento**: cada método `@Test` verifica un solo comportamiento específico
5. **No agregar dependencias nuevas sin preguntar**: el pom.xml actual solo tiene JUnit 5
6. **Cualquier cambio debe dejar `mvn test` en verde**: nunca hacer commit con pruebas rotas

## Reglas de Trabajo
- Antes de hacer commit, ejecutar `mvn test` y verificar que todas las pruebas pasen
- Si una prueba falla, arreglar el código o la prueba antes de continuar
- Las pruebas deshabilitadas con `@Disabled` están intencionalmente rotas para práctica
- Mantener la arquitectura por capas: Modelo → Repositorio → Servicio → App

## Estructura
```
mx.generation.tareas/
  ├── Tarea.java                      # Modelo
  ├── Prioridad.java                  # Enum BAJA, MEDIA, ALTA
  ├── TareaRepositorio.java           # Capa de datos (CRUD en memoria)
  ├── TareaServicio.java              # Lógica de negocio
  ├── TareaNoEncontradaException.java # Excepción personalizada
  └── App.java                        # Demo de consola
```