<h1 align="center">C.D.I. (Context Dependency Injection)</h1>
<p>Es una especificación estándar y framework para la inyección de dependencia y manejo de contextos incluido desde Java EE 6 en adelante</p>
<p align="center"><img width="719" alt="image" src="https://github.com/user-attachments/assets/d361a790-efa2-47ce-98ba-8ae7c270fe04"></p>
<h1 align="center">Inyección de dependencia</h1>
<p>Es un patrón de diseño que forma parte de la programación orientada a objetos en la plataforma Java EE, es parte de la especificación JSR-330.</p>
<p>Especifica que se inyectará un componente o variable de contexto en un atributo de otro componente CDI, es decir para inyectar componentes de la aplicación en el componente actual</p>
<p><b>'Inyectar es justamente suministrar a un objeto una referencia de otros que necesite según la relación, tiene que plasmarse mediante la anotación @Inject'</b></p>
<h1 align="center">Características CDI</h1>
<p align="center"><img width="719" alt="image" src="https://github.com/user-attachments/assets/d73fb364-a22b-4b7f-9dc8-1eeb30bab9d6"></p>
<h1 align="center">pom.xml</h1>

```xml
        <dependency>
            <groupId>org.jboss.weld.servlet</groupId>
            <artifactId>weld-servlet-core</artifactId>
            <version>5.1.1.SP1</version>
        </dependency>
```
Al incluir esta dependencia en el `pom.xml`, la aplicación web puede aprovechar las capacidades de CDI para gestionar la inyección de dependencias y el ciclo de vida de los beans. Esto facilita el desarrollo de aplicaciones más modulares, mantenibles y escalables, ya que se puede desacoplar componentes y dejar que el contenedor de CDI gestione sus dependencias.

<h1 align="center">Registrar e inyectar</h1>
<h2>Registrar o publicar un bean:</h2>

- Se crea de forma automática, no hay que hacer nada especial para publicar un bean en el contexto de CDI:
```java
public interface Repositorio { }

public class Repositoriolmpl implements Repositorio { }
```

- Inyectar un bean existente en otro bean:
```java
public class Servicelmpl implements Service {

@Inject
private Repositorio repositorio;

}
```

<h1 align="center">Manejo de contextos</h1>

Podríamos no definer ningún contexto explícitamente y queda como `@Dependent`

```java
public interface Repositorio { }

public class Repositoriolmpl implements Repositorio { }
```
Pero también podríamos definir un contexto explícitamente
```java
@ApplicationScoped
public class Servicelmpl implements Service {

@Inject
private Repositorio repositorio;

}
```

<h1 align="center">Contextos CDI</h1>
<h2>@Dependent</h2>

- <b>Descripción</b>: El contexto `@Dependent` es el alcance predeterminado de un bean CDI si no se especifica otro alcance.
- <b>Características</b>:
  - Un bean con este contexto no tiene ciclo de vida propio; su ciclo de vida está vinculado al objeto que lo inyecta.
  - Se crea una nueva instancia cada vez que se inyecta el bean.
  - No tiene almacenamiento de estado, lo que significa que no se mantiene entre invocaciones.

<h2>@RequestScoped</h2>

- <b>Descripción</b>: El contexto `@RequestScoped` define un ciclo de vida que abarca una sola solicitud HTTP.
- <b>Características</b>:
  - Una instancia del bean se crea al inicio de una solicitud HTTP y se destruye al final de esa solicitud.
  - Es útil para almacenar datos temporales que solo son relevantes durante la ejecución de una solicitud.
  - Se utiliza frecuentemente en aplicaciones web donde se manejan solicitudes HTTP.
 
<h2>@SessionScoped</h2>

- <b>Descripción</b>: El contexto `@SessionScoped` define un ciclo de vida que abarca la sesión de un usuario en una aplicación web.
- <b>Características</b>:
  - Una instancia del bean se crea cuando se inicia una sesión HTTP y se destruye cuando la sesión expira o se cierra.
  - Es adecuado para almacenar datos del usuario que deben persistir mientras dure su sesión.
  - Los beans en este contexto deben implementar la interfaz `Serializable` para asegurar que pueden ser serializados.

<h2>@ConversationScoped</h2>

- <b>Descripción</b>: El contexto `@ConversationScoped` permite definir un ciclo de vida más largo que una solicitud pero más corto que una sesión.
- <b>Características</b>:
  - Permite dividir una interacción de usuario en varias solicitudes HTTP mientras mantiene el estado.
  - Una conversación puede ser transitoria (dura solo una solicitud) o larga (dura varias solicitudes).
  - Requiere control explícito del ciclo de vida de la conversación usando la API de conversación de CDI.
 
<h2>@ApplicationScoped</h2>

- <b>Descripción</b>: El contexto `@ApplicationScoped` define un ciclo de vida que abarca toda la aplicación.
- <b>Características</b>:
  - Una única instancia del bean se crea cuando se inicia la aplicación y se destruye al detenerla.
  - Es ideal para almacenar datos o configuraciones que deben ser compartidos entre todos los usuarios y sesiones.
  - Proporciona un comportamiento similar a un singleton pero gestionado por el contenedor CDI.

<h1 align="center">Anotación @Named</h1>

CDI también nos permite dar nombres a los beans y realizar la inyección mediante el nombre con la anotación `@Named`.

```java
public interface Repositorio { }

@Named("jdbcRepositorio")
public class Repositoriolmpl implements Repositorio { }
```

Luego en el Service lo inyectamos vía nombre del beans
```java
public class Servicelmpl implements Service {

@Inject
@Named("jdbcRepositorio")
private Repositorio repositorio;
```

<h1 align="center">Anotación @Produces</h1>

Otra forma para registrar un bean mediante método, el objeto que devuelve este método (anotado con `@Produces`) queda registrado en el contenedor CDI.
```java
@Produces
public Conexion produceConexion() {
return new Conexion();
```
Opcionalmente también puede tener un nombre y contexto
```java
@Produces
@RequestScoped
@Named("conn")
public Conexion produceConexion() {
return new Conexion();
```

<h1 align="center">Anotación @Qualifier</h1>

Cuando se tiene varias implementaciones de una misma interfaz o clase, y se desea inyectar una de esas implementaciones específicas en un punto particular del código, se utiliza un `@Qualifier` para indicar cuál de las implementaciones debe ser inyectada. Esto ayuda a evitar ambigüedades cuando el contenedor de C.D.I. tiene que decidir qué `bean` inyectar.

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({METHOD, FIELD, PARAMETER, TYPE})
public @interface ProductoServicePrincipal {
}
```

- `@Qualifier`: La anotación que se está creando se utilizará para distinguir entre diferentes implementaciones de un bean a la hora de realizar inyecciones.
- `@Retention(RetentionPolicy.RUNTIME)`: Especifica que la anotación estará disponible en tiempo de ejecución. Esto es necesario para que el contenedor de CDI pueda leer y procesar la anotación cuando se ejecuta la aplicación.
- `@Target({METHOD, FIELD, PARAMETER, TYPE})`: Indica los elementos del código en los que esta anotación puede aplicarse. En este caso, puede ser aplicada a métodos (`METHOD`), campos (`FIELD`), parámetros (`PARAMETER`), y tipos de clase (`TYPE`).

<h1 align="center">Anotación @Stereotype</h1>

- Función de `@Stereotype`
  - <b>Agrupación de Anotaciones</b>: Permite combinar varias anotaciones en una sola, de modo que al aplicar el estereotipo a un `bean`, todas las anotaciones agrupadas se aplican automáticamente. Esto reduce la repetición y simplifica la configuración de beans.
  - <b>Estandarización de Comportamientos</b>: Facilita la aplicación de configuraciones estandarizadas a los beans dentro de la aplicación. Por ejemplo, definir un estereotipo para un componente de servicio que siempre será transaccional y tendrá un ciclo de vida determinado.

```java
@SessionScoped
@Named
@Stereotype
@Retention(RUNTIME)
@Target(ElementType.TYPE)
public @interface CarroCompra {
}
```

- `@SessionScoped`: indica que cualquier bean anotado con `@CarroCompra` tendrá un alcance de sesión. Un bean con alcance de sesión (`@SessionScoped`) se mantiene vivo y su estado se preserva durante toda la duración de la sesión del usuario en una aplicación web. Esto significa que el bean estará disponible mientras dure la sesión HTTP del usuario.
- `@Named`: permite que el bean sea accesible en la capa de presentación, como en las páginas JSF (JavaServer Faces). Esto significa que el bean puede ser referenciado en expresiones EL (Expression Language) dentro de archivos `.xhtml` o .`jsp` usando su nombre de clase en minúsculas, a menos que se le asigne un nombre explícito.
- `@Stereotype`: marca `@CarroCompra` como un estereotipo. Al marcar `@CarroCompra` con `@Stereotype`, se está diciendo que esta anotación se comportará como un estereotipo, permitiendo la reutilización de configuraciones comunes.
- `@Retention(RUNTIME)`: indica que la nueva anotación `@CarroCompra` estará disponible en tiempo de ejecución. Esto es necesario para que el contenedor CDI pueda leer y procesar la anotación mientras la aplicación está en ejecución, permitiendo que se apliquen los comportamientos y configuraciones definidas.
- `@Target(ElementType.TYPE)`: especifica que `@CarroCompra` solo se puede aplicar a tipos, es decir, a clases o interfaces. Esto significa que no se puede aplicar `@CarroCompra` a métodos, campos, o parámetros; solo puede ser utilizada para anotar clases o interfaces.
- `public @interface CarroCompra {}`: Define una nueva anotación personalizada llamada `CarroCompra`. Esta anotación puede ser utilizada para marcar clases o interfaces en el código. Cuando se usa `@CarroCompra` en una clase, automáticamente aplicará las características de las anotaciones `@SessionScoped` y `@Stereotype` a esa clase, así como cualquier otro comportamiento asociado con el estereotipo.

<h1 align="center">Anotación @Disposes</h1>

La anotación `@Disposes` se utiliza en el mecanismo de inyección de dependencias para liberar o limpiar recursos que han sido previamente creados o inyectados mediante un método productor.

- Liberación de Recursos: Cuando un bean necesita liberar un recurso específico, el método que realiza esta acción se marca con `@Disposes`. Este método es llamado automáticamente por el contenedor CDI cuando el contexto del bean termina y el recurso debe ser destruido.
- Emparejamiento con Métodos Productores: La anotación `@Disposes` siempre se usa en combinación con un método productor (`@Produces`). Mientras que el método productor crea o provee una instancia de un recurso (como un objeto de conexión), el método marcado con `@Disposes` realiza la tarea de limpieza o destrucción de dicho recurso.

```java
public class ProducerResources {

    @Resource(name = "jdbc/mysqlDB")
    private DataSource ds;

    @Produces
    @RequestScoped
    @MySQLConn
    private Connection beanConnection() throws NamingException, SQLException {
        return ds.getConnection();
    }

    public void close(@Disposes @MySQLConn Connection connection) throws SQLException {
        connection.close();
    }
}
```
Función del Método `close`
Este método tiene la responsabilidad de cerrar la conexión a la base de datos cuando ya no se necesita.
- `@Disposes`:
  - <b>Función</b>: Indica que este método se utiliza para liberar un recurso que fue previamente producido por un método productor (en este caso, el método `beanConnection`).
  - <b>Contexto</b>: En CDI, cuando un bean de tipo Connection ya no es necesario, el contenedor CDI llama automáticamente a este método para liberar la conexión.

<h1 align="center">Integración con 'EL' (Lenguaje de Expresión)</h1>

También se integra con la librería EL de JSP donde nos permite acceder a métodos y atributos de los beans o componentes CDI mediante el nombre definido con la anotación `@Named`, es decir son asignaciones (o mapping) hacia estos objetos.

```java
@SessionScoped
@Named
public class Carro implements Serializable {
```

Accedemos al carro en las vistas JSP mediante EL:
```jsp
<c:forEach items="${carro.items}" var="item">

</c:forEach>

Total: ${carro.total}
```

<h1 align="center">Anotación @PostConstruct y @PreDestroy</h1>

En el contexto de <b>CDI (Contexts and Dependency Injection)</b> en Java, las anotaciones `@PostConstruct` y `@PreDestroy` se utilizan para definir métodos que deben ejecutarse en momentos específicos del ciclo de vida de un bean.

<h2>@PostConstruct</h2>

- <b>Cuándo se ejecuta</b>: Después de que el contenedor CDI ha inyectado todas las dependencias en el bean, pero antes de que el bean esté disponible para su uso por otros componentes.
- <b>Propósito</b>: Se utiliza para realizar cualquier inicialización necesaria que dependa de las inyecciones de dependencias. Es como un constructor extendido, donde ya se tiene acceso a los recursos inyectados.

```java
@PostConstruct
public void init() {
    // Código de inicialización aquí
}
```

<h2>@PreDestroy</h2>

- <b>Cuándo se ejecuta</b>: Justo antes de que el contenedor CDI destruya el bean.
- <b>Propósito</b>: Se utiliza para realizar cualquier limpieza de recursos, como cerrar conexiones de base de datos o liberar otros recursos que el bean pudo haber adquirido durante su vida útil.

```java
@PreDestroy
public void cleanup() {
    // Código de limpieza aquí
}
```

<h1 align="center">Interceptores C.D.I.</h1>
<p>Los <b>Interceptores CDI (Contexts and Dependency Injection)</b> son una característica en Java que permite interceptar y gestionar la ejecución de métodos de los beans CDI. Un interceptor es una clase que contiene lógica que se ejecuta antes o después de que un método de un bean sea invocado, o incluso alrededor de la ejecución del método.</p>
<h2>¿Para qué sirven los Interceptores CDI?</h2>
<p>Los interceptores se utilizan para agregar comportamiento adicional a los métodos de los beans sin modificar el código del propio método. Esto es particularmente útil para implementar funcionalidades transversales, como:</p>

- <b>Gestión de transacciones</b>: Iniciar y finalizar una transacción antes y después de ejecutar un método.
- <b>Manejo de excepciones</b>: Capturar y gestionar excepciones lanzadas por un método.
- <b>Auditoría y logging</b>: Registrar las llamadas a métodos y sus parámetros o resultados.
- <b>Control de seguridad</b>: Verificar permisos antes de que se ejecute un método.

<h2>¿Cómo funcionan?</h2>
<p>Para usar interceptores, se debe seguir estos pasos:</p>

- <b>Definir un interceptor</b>: Se crea una clase que implemente la interfaz `javax.interceptor.Interceptor`. En esa clase se definen los métodos que contienen la lógica que se ejecutará antes, después, o alrededor del método interceptado.

- <b>Anotar el interceptor</b>: Los métodos del interceptor se anotan con `@AroundInvoke` para indicar que deben interceptar la ejecución de métodos.
```java
@Interceptor
@Logged
public class LoggingInterceptor {

    @AroundInvoke
    public Object logMethod(InvocationContext context) throws Exception {
        System.out.println("Método llamado: " + context.getMethod().getName());
        Object result = context.proceed(); // Ejecuta el método original
        System.out.println("Método finalizado: " + context.getMethod().getName());
        return result;
    }
}
```
- <b>Activar el interceptor</b>: El interceptor se asocia con un bean o un método del bean utilizando una anotación personalizada. La anotación debe estar marcada con `@InterceptorBinding`.
```java
@Inherited
@InterceptorBinding
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface Logged {
}
```
- <b>Declarar el interceptor</b>: Finalmente, el interceptor debe ser declarado en el archivo `beans.xml` para que CDI lo reconozca y lo aplique.
```xml
<beans>
    <interceptors>
        <class>com.ejemplo.LoggingInterceptor</class>
    </interceptors>
</beans>
```
