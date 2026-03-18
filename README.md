# CI/CD con Maven y GitHub Actions

## 1. Estructura del proyecto Maven

Un proyecto Maven bien organizado es la base para que el pipeline de CI funcione sin problemas:

```
proyecto/
├── src/
│   ├── main/java/     # Código de producción
│   └── test/java/     # Tests unitarios
└── pom.xml            # Dependencias, plugins y versión de Java
```

> **Regla clave:** Si `mvn clean install` funciona en local → debe funcionar en CI.

---

## 2. Dependencias y plugins esenciales

### Testing
- **`spring-boot-starter-test`** (proyectos Spring Boot): incluye JUnit 5, AssertJ y Mockito.

### Plugins Maven importantes

| Plugin | Función |
|--------|---------|
| `maven-compiler-plugin` | Compila el proyecto |
| `maven-surefire-plugin` | Ejecuta tests unitarios |
| `maven-failsafe-plugin` | Ejecuta tests de integración (opcional) |

---

## 3. Tests

Buenas prácticas para que los tests pasen en un entorno limpio de CI:

- Deben ejecutarse **sin depender del IDE**.
- Evitar dependencias de rutas locales o bases de datos externas → usar **H2 en memoria** si es necesario.
- Nombres descriptivos con un único escenario por test:
  ```
  metodo_cuandoCondicion_retornaResultado
  ```

---

## 4. Flujo básico con GitHub Actions

Archivo de configuración: `.github/workflows/ci-mvn.yml`

```yaml
name: CI con Maven

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK
        uses: actions/setup-java@v3
        with:
          java-version: '23'
          distribution: 'temurin'

      - name: Cache Maven packages
        uses: actions/cache@v3
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-maven-

      - name: Build with Maven
        run: mvn clean install
```

### ¿Qué hace `mvn clean install`?
1. **clean** → Elimina artefactos anteriores
2. **compile** → Compila el código fuente
3. **test** → Ejecuta los tests unitarios
4. **package** → Genera el `.jar` o `.war`

> Si algún test falla, GitHub Actions marcará el workflow como **Failed** ✗

---

## 5. Buenas prácticas

1. **Mantén el `pom.xml` limpio**: no sobreescribas versiones de dependencias gestionadas por Spring Boot.
2. **Evita dependencias del entorno local**: sin rutas absolutas ni servicios externos sin mock.
3. **Usa perfiles Maven** si necesitas configuraciones distintas para CI (p. ej., base de datos H2 para tests).
4. **Verifica la versión de Java**: el runner de GitHub Actions debe usar la misma que la configurada en el proyecto.
5. **Habilita la caché de Maven**: reduce significativamente el tiempo de build en cada ejecución.

---
Fuentes: [ChatGPT](https://chat.openai.com) + [Claude](https://claude.ai)