
# MS-Vehiculos — Pipeline de Despliegue (EP1 - Ingeniería DevOps)

## Integrantes
- **Marcelo Acevedo** — Mar.acevedoa@duocuc.cl
- **Daniel Rios** — dan.riosv@duocuc.cl

## Descripción del microservicio

**MS-Vehiculos** es un microservicio de catálogo de vehículos que expone una API REST con operaciones CRUD completas (Crear, Leer, Actualizar y Eliminar). Fue construido con **Spring Boot 3.5** y **Java 21**, utilizando **Spring Data JPA** para la persistencia en una base de datos **MySQL** alojada en **AWS RDS**. La documentación interactiva de la API se genera automáticamente mediante **Swagger UI (SpringDoc OpenAPI)**.

El microservicio proviene de un proyecto desarrollado en ramos anteriores de la carrera y fue seleccionado como base para implementar el pipeline DevOps de esta evaluación. Se despliega en dos instancias **AWS EC2** con Ubuntu, gestionado como servicio del sistema operativo mediante `systemd`.

**Tecnologías principales:**
- Java 21 + Spring Boot 3.5
- Spring Data JPA + MySQL (AWS RDS)
- Swagger UI (SpringDoc OpenAPI 2.8)
- Lombok
- Maven (wrapper incluido)

---

## 1. Estrategia de ramificación

Elegimos **Trunk-based Development** para este proyecto.

**Justificación:**

Tras analizar los diferentes modelos de ramificación disponibles (GitFlow, GitHub Flow, Trunk-based Development), optamos por **Trunk-based Development** considerando los siguientes factores:

- **Tamaño del equipo (2 personas):** Al ser un equipo reducido, la complejidad de mantener múltiples ramas de larga duración (como requiere GitFlow con `develop`, `release`, `feature/*`, etc.) genera un overhead innecesario. Trunk-based simplifica el flujo al trabajar directamente sobre `main`, lo que reduce la fricción y los conflictos de merge.

- **Frecuencia de despliegues:** Este modelo favorece la **integración continua rápida**, ya que cada commit en `main` puede ser potencialmente desplegable. Esto se alinea con la filosofía DevOps de entregar valor de forma frecuente y con bajo riesgo.

- **Contexto de entorno cloud simulado:** En el marco de esta evaluación, trabajamos con instancias EC2 en AWS donde el despliegue es directo. No necesitamos releases planificados ni ramas de staging, por lo que la simplicidad de trunk-based es la opción más eficiente.

- **Ventajas reconocidas:**
  - Integración continua real: los cambios se integran rápidamente, evitando la acumulación de deuda técnica.
  - Menor cantidad de conflictos de merge.
  - Historial de commits lineal y fácil de auditar.
  - Detección temprana de errores gracias a la ejecución de CI en cada push.

- **Desventajas reconocidas:**
  - Requiere disciplina en la calidad de los commits (cada commit debe ser funcional).
  - En equipos más grandes, podría generar problemas si no se complementa con feature flags o ramas de corta duración.
  - Menor aislamiento de funcionalidades en desarrollo.

**Comparación con otros modelos:**

| Modelo | Ventaja principal | Desventaja para nuestro caso |
|--------|-------------------|------------------------------|
| **GitFlow** | Releases planificados, separación clara de etapas | Excesivo overhead para un equipo de 2, ramas de larga duración innecesarias |
| **GitHub Flow** | Simplicidad con PRs | Añade pasos intermedios que no necesitamos con solo 2 integrantes |
| **Trunk-based** ✅ | Integración continua rápida, mínima fricción | Requiere commits de calidad (mitigado con CI) |

**Estructura de ramas utilizada:**
- `main` → rama principal, código estable y listo para producción. Todos los commits se integran directamente aquí.

> **Nota:** En caso de escalar el equipo, se adoptarían ramas de corta duración (`feature/<nombre>`, `hotfix/<nombre>`) que se integran rápidamente a `main`, manteniendo el espíritu de trunk-based.

---

## 2. Convenciones de commits

Usamos el formato **Conventional Commits**:

```
<tipo>: <descripción breve en minúsculas>
```

Tipos permitidos:
| Tipo       | Uso                                               |
|------------|----------------------------------------------------|
| `feat`     | Nueva funcionalidad                                 |
| `fix`      | Corrección de errores                               |
| `docs`     | Cambios solo de documentación                       |
| `refactor` | Cambio de código que no agrega función ni corrige bug |
| `test`     | Agregar o modificar pruebas                         |
| `chore`    | Tareas de mantenimiento (configuración, dependencias) |
| `ci`       | Cambios en la configuración de CI/CD                |

Ejemplos:
```
feat: agregar endpoint de consulta de vehiculos por patente
fix: corregir validacion de patente duplicada en actualizacion
docs: actualizar guia de instalacion en README
ci: configurar workflow de GitHub Actions para compilacion Maven
```

**Reglas adicionales:**
- Los mensajes se escriben en español e infinitivo (ej: "agregar", "corregir", no "agregado" o "se agregó").
- Máximo 72 caracteres en la primera línea.
- Se puede agregar un cuerpo descriptivo separado por una línea en blanco si el cambio lo amerita.

---

## 3. Naming de ramas

Aunque trabajamos con trunk-based development directamente en `main`, documentamos las convenciones de naming para ramas de corta duración que se usarían en caso de escalar el equipo:

| Prefijo      | Ejemplo                        | Uso |
|--------------|--------------------------------|-----|
| `feature/`   | `feature/endpoint-vehiculos`   | Nueva funcionalidad |
| `hotfix/`    | `hotfix/patente-duplicada`     | Corrección urgente |

**Reglas:**
- Nombres en minúsculas, separados por guiones (`-`).
- Sin espacios ni caracteres especiales (acentos, ñ, etc.).
- Nombre corto pero descriptivo del cambio.
- Máximo 3-4 palabras después del prefijo.

---

## 4. Flujo de trabajo

### En Trunk-based Development (modelo actual):
1. Se realizan los cambios directamente en `main`.
2. Se realizan commits siguiendo la convención de Conventional Commits.
3. Se sube el cambio con `git push origin main`.
4. El workflow de **GitHub Actions** se ejecuta automáticamente, compilando y validando el código.
5. Si el workflow falla, se corrige inmediatamente con un nuevo commit.

### Flujo de merge (para ramas de corta duración, si se escala):
1. Se crea la rama (`feature/...` o `hotfix/...`) desde `main`.
2. Se realizan los commits siguiendo la convención anterior.
3. Se sube la rama (`git push`) y se abre un **Pull Request** hacia `main`.
4. Se usa **Squash and merge** como estrategia para mantener un historial limpio y legible en `main`, condensando todos los commits de la rama en uno solo con un mensaje descriptivo.
5. Una vez aprobado el PR, se elimina la rama origen.

---

## 5. Estrategia de revisión de Pull Requests

- Cada PR debe ser revisado por **al menos 1 integrante distinto del autor** antes de mergear.
- Checklist mínima de revisión:
  - [ ] El código compila/ejecuta sin errores.
  - [ ] Los commits siguen la convención definida.
  - [ ] El PR incluye una descripción clara del cambio.
  - [ ] El workflow de GitHub Actions pasó correctamente.
  - [ ] Se actualizó documentación si corresponde.

> Se utiliza una [plantilla de Pull Request](.github/PULL_REQUEST_TEMPLATE.md) que estandariza la descripción de cada PR.

---

## 6. Automatización con GitHub Actions

Se configuró un workflow ([`.github/workflows/ci.yml`](.github/workflows/ci.yml)) que se ejecuta automáticamente en cada **push a `main`**.

### ¿Qué hace el workflow?

1. **Descarga el código fuente** del repositorio usando `actions/checkout`.
2. **Configura el entorno Java 21** (distribución Temurin) con `actions/setup-java`, incluyendo caché automático de dependencias Maven.
3. **Compila el proyecto** ejecutando `./mvnw clean package -DskipTests`, lo que:
   - Descarga todas las dependencias definidas en `pom.xml`.
   - Compila las clases Java.
   - Empaqueta el microservicio como un archivo `.jar` ejecutable.

### ¿Por qué es importante?

Este workflow implementa la etapa de **Integración Continua (CI)** del pipeline DevOps:

- **Detección temprana de errores:** Cada push activa automáticamente la compilación. Si un commit introduce un error de compilación, se detecta inmediatamente antes de que afecte al equipo o al entorno de producción.
- **Validación automática:** No depende de que un integrante recuerde compilar manualmente; el proceso es automático y consistente.
- **Feedback rápido:** Los resultados del workflow se visualizan directamente en GitHub, proporcionando retroalimentación inmediata al desarrollador.
- **Base para CD:** Este workflow sienta las bases para agregar etapas de Continuous Deployment (CD) en el futuro, como el despliegue automático a las instancias EC2 vía SSH.

---

## 7. Estructura del proyecto

```
ms-vehiculos/
├── .github/
│   ├── workflows/
│   │   └── ci.yml                    # Workflow de GitHub Actions (CI)
│   └── PULL_REQUEST_TEMPLATE.md      # Plantilla de Pull Requests
├── src/
│   └── main/
│       ├── java/cl/matiivilla/vehiculos/
│       │   ├── BibliotecaVehiculosApplication.java   # Clase principal
│       │   ├── controller/
│       │   │   ├── HomeController.java               # Endpoint raíz (/)
│       │   │   └── VehiculoController.java           # CRUD REST (/api/v1/vehiculos)
│       │   ├── dto/
│       │   │   ├── VehiculoRequest.java              # DTO de entrada con validaciones
│       │   │   └── VehiculoResponse.java             # DTO de salida
│       │   ├── model/
│       │   │   └── Vehiculo.java                     # Entidad JPA
│       │   ├── repository/
│       │   │   └── VehiculoRepository.java           # Repositorio JPA
│       │   └── service/
│       │       └── VehiculoService.java              # Lógica de negocio
│       └── resources/
│           └── application.yml                       # Configuración de la aplicación
├── pom.xml                                           # Configuración Maven
├── mvnw / mvnw.cmd                                   # Maven Wrapper
├── .gitignore                                        # Archivos ignorados por Git
└── README.md                                         # Este archivo
```

---

## 8. Endpoints de la API

| Método   | Ruta                    | Descripción                    |
|----------|-------------------------|--------------------------------|
| `GET`    | `/`                     | Información de la API          |
| `GET`    | `/api/v1/vehiculos`     | Listar todos los vehículos     |
| `GET`    | `/api/v1/vehiculos/{id}`| Buscar vehículo por ID         |
| `POST`   | `/api/v1/vehiculos`     | Crear un vehículo nuevo        |
| `PUT`    | `/api/v1/vehiculos/{id}`| Actualizar un vehículo         |
| `DELETE` | `/api/v1/vehiculos/{id}`| Eliminar un vehículo           |

**Documentación Swagger UI:** `http://<IP>:9004/swagger-ui.html`

---

## 9. Despliegue

El microservicio se despliega en **2 instancias AWS EC2** con Ubuntu.

| Instancia | IP Pública | Puerto | Rol |
|-----------|-----------|--------|-----|
| EVA_1 | `54.157.118.185` | 9004 | Servidor principal |
| EVA_1_ACTION | `34.229.73.22` | 9004 | Servidor secundario |

**Base de datos:** MySQL en AWS RDS (`database-1.cj6ww6io2ki7.us-east-1.rds.amazonaws.com:3306/vehiculos`)

---

## Uso de Inteligencia Artificial

Declaramos el uso de IA de la siguiente forma:
- **Herramienta utilizada:** Google Antigravity (asistente de IA para desarrollo)
- **Uso:** Apoyo en la redacción y estructuración del README, generación del workflow base de GitHub Actions, y organización de la guía de despliegue.
- Las justificaciones técnicas, decisiones de diseño y reflexiones fueron elaboradas y validadas por el equipo.

---

## Conclusiones y reflexiones individuales

> Cada integrante debe redactar su propia reflexión SIN apoyo de IA, explicando su aprendizaje y contribución al proyecto.

**Marcelo Acevedo:**

<!-- ⚠️ COMPLETAR: Escribir reflexión personal sin uso de IA -->

Como reflexion final, quiero recalcar que el trabajo fue realizado en equipo, y que no se me da bien el uso de github, pero con ayuda de la IA pude solventar esa falencia, la cual deberia de mejorar, de momento el uso de AWS y el trabajo con base de datos me ha llamado la atencion, y espero poder seguir aprendiendo de ello. Me encargue de que el microservicio pudiera operar de la manera en que nosotros queriamos.
**Daniel Rios:**

<!-- ⚠️ COMPLETAR: Escribir reflexión personal sin uso de IA -->

