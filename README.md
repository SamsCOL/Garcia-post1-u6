# Post-contenido — Unidad 6: JSP con MVC

## Descripción
Repositorio del laboratorio de la Unidad 6 de Programación Web — Séptimo
Semestre. Contiene un único proyecto Maven Web (`gestor-tareas-mvc`) que
formaliza el patrón MVC con un Front Controller y el patrón Comando,
extendido con autenticación por sesión con roles, validación por campo
e internacionalización.

## Prerrequisitos
- JDK 17 o superior
- Apache Tomcat 10.x (puerto 8080)
- Maven 3.8+ (o el integrado en IntelliJ IDEA / Eclipse)
- Git

## Estructura (plana)
Todos los archivos del proyecto están en la raíz del repositorio, sin
subcarpetas. Cada clase Java conserva su `package` (`model`, `service`,
`controller`, `controller.comando`), y el `pom.xml` indica a Maven en qué
ruta del WAR debe quedar cada archivo:

| Archivo(s) en la raíz | Ruta dentro del WAR |
|---|---|
| `*.java` | `WEB-INF/classes/com/ejemplo/mvc/...` (compiladas) |
| `messages.properties`, `messages_es.properties` | `WEB-INF/classes/` |
| `web.xml` | `WEB-INF/web.xml` |
| `index.jsp` | `/index.jsp` |
| `lista.jsp`, `formulario.jsp`, `login.jsp` | `WEB-INF/views/` (no accesibles por URL directa) |
| `estilos.css` | `css/estilos.css` |

| Capa | Archivos |
|---|---|
| Modelo | `Tarea.java`, `TareaDAO.java`, `Usuario.java`, `UsuarioDAO.java` |
| Servicio | `TareaService.java`, `AutenticacionService.java` |
| Controlador | `FrontControllerServlet.java` |
| Comandos | `Comando.java`, `ListarComando.java`, `FormularioComando.java`, `GuardarComando.java`, `EliminarComando.java`, `CompletarComando.java`, `LoginComando.java`, `LogoutComando.java`, `IdiomaComando.java` |
| Vistas | `index.jsp`, `lista.jsp`, `formulario.jsp`, `login.jsp`, `estilos.css` |
| Configuración / i18n | `pom.xml`, `web.xml`, `messages.properties`, `messages_es.properties` |

## Parte 1 — Front Controller y patrón Comando
`FrontControllerServlet` es el único punto de entrada (`/app`) y delega en
objetos Comando (`ListarComando`, `FormularioComando`, `GuardarComando`,
`EliminarComando`, `CompletarComando`). `TareaService` y `TareaDAO` separan la
lógica de negocio y el acceso a datos del Controlador. Las vistas usan
JSTL y Expression Language, sin scriptlets (incluso `index.jsp` redirige
con `<c:redirect>`). Las operaciones que modifican datos terminan con
`sendRedirect` (patrón Post/Redirect/Get).

## Parte 2 — Sesión con roles, validación por campo e i18n
`FrontControllerServlet` centraliza la verificación de sesión antes de
resolver cualquier comando protegido. `LoginComando`/`LogoutComando`
gestionan `HttpSession` con roles ADMIN/USER; `EliminarComando` solo
permite el rol ADMIN. `GuardarComando` valida cada campo del formulario
por separado, usando el límite de longitud del título leído del
contexto de aplicación (`web.xml`), y repuebla el formulario con lo
ingresado. `IdiomaComando` guarda la preferencia de idioma en una Cookie,
leída directamente en las vistas con el objeto EL implícito `cookie` y
`ResourceBundle` (`messages.properties` / `messages_es.properties`).

## Funcionalidades implementadas
- Listado de tareas con 3 tareas precargadas (`/app` sin parámetros).
- Registro de tareas con redirect POST→GET a `/app?comando=listar`.
- Completar tarea (se muestra tachada).
- Eliminar tarea con diálogo de confirmación, solo para rol ADMIN.
- Login/logout con `HttpSession` (timeout de 30 min) y saludo con nombre y rol.
- Redirección a login para cualquier comando protegido sin sesión activa.
- Validación en servidor con mensaje por campo (título obligatorio y con
  longitud máxima, categoría obligatoria, prioridad válida, fecha con
  formato `yyyy-MM-dd` y no pasada) y repoblado del formulario.
- Selector Español/English persistido en cookie (30 días), aplicado también
  en la pantalla de login.

## Usuarios de prueba
| Usuario | Clave | Rol |
|---|---|---|
| admin | Admin123! | ADMIN |
| maria | Maria2026! | USER |

## Decisiones de diseño
- **El Comando devuelve la vista lógica** (o `null` si ya hizo un redirect)
  en vez de invocar el forward directamente, para que
  `FrontControllerServlet` concentre esa llamada en un solo lugar y pueda
  aplicar lógica común a todos los comandos.
- **Front Controller en vez de un Servlet por acción**: la verificación de
  sesión de la Parte 2 se agregó una sola vez, en `procesar()`, sin
  copiarla al inicio de cada Servlet. Cualquier comando nuevo queda
  protegido automáticamente salvo que se agregue al conjunto `PUBLICOS`.
- **Sesión en `HttpSession`, idioma en Cookie**: el usuario y el rol viven
  en `HttpSession` porque deben expirar con la sesión; el idioma vive en
  una Cookie porque debe sobrevivir al cierre de sesión (Guía, tabla
  comparativa 5.4), de modo que la pantalla de login ya aparece en el
  idioma elegido.
- **Validación por campo**: `GuardarComando` acumula los errores en un
  `LinkedHashMap` (campo → mensaje) en lugar de detenerse en el primero,
  para mostrar todos los problemas a la vez junto a su campo y conservar
  los valores ya ingresados.
- **Longitud máxima del título en el contexto de aplicación**: se lee del
  `context-param` `app.maxLongitudTitulo` de `web.xml` en vez de
  codificarse como literal en `GuardarComando`, para poder cambiar la
  regla sin recompilar.
- **Estructura plana**: por requisito de entrega, todos los archivos están
  en un mismo nivel; el `pom.xml` (`sourceDirectory`, `resources`,
  `webResources` del `maven-war-plugin`) reconstruye la estructura estándar
  del WAR al empaquetar, manteniendo las JSP protegidas en `WEB-INF/views`.

## Cómo compilar y desplegar
1. Clonar el repositorio: `git clone [URL-del-repo]`
2. Abrir la carpeta como proyecto Maven en IntelliJ IDEA.
3. Ejecutar `mvn clean package` → genera `target/gestor-tareas-mvc.war`.
4. Opción A: copiar el WAR a `webapps/` de Tomcat 10.
   Opción B: en IntelliJ, *Run → Edit Configurations → + → Tomcat Server →
   Local*, pestaña *Deployment*, agregar `gestor-tareas-mvc:war exploded`.
5. Abrir `http://localhost:8080/gestor-tareas-mvc/app`.

## Capturas de pantalla
![Listado de tareas](lista-tareas.png)
![Formulario con validaciones por campo](formulario-errores.png)
![Login](login.png)
![Restricción de rol al eliminar](restriccion-rol.png)
![Selector de idioma](idioma.png)
