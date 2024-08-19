Sección 1: Introducción a Servicios en Quarkus
•	¿Qué es @ApplicationScoped en Quarkus?
Es una anotación que indica que un bean debe tener un único ciclo de vida en toda la aplicación. Esto significa que se crea una sola instancia del bean durante la vida de la aplicación y se reutiliza en todas las solicitudes. Este ámbito es útil para servicios o componentes que deben mantener estado o datos compartidos durante toda la ejecución de la aplicación.
•	¿Cómo funciona la inyección de dependencias en Quarkus?
En Quarkus, la inyección de dependencias se basa en CDI (Contexts and Dependency Injection) funciona mediante la anotación de clases con @Inject, que permite al contenedor de Quarkus gestionar la creación y el ciclo de vida de las dependencias. Cuando una clase requiere una dependencia, Quarkus inyecta automáticamente la instancia adecuada en el punto solicitado, ya sea en campos, métodos o constructores. Esto facilita la gestión de dependencias y promueve un diseño desacoplado y modular.

•	¿Cuál es la diferencia entre @ApplicationScoped, @RequestScoped, y @Singleton en Quarkus?
• @ApplicationScoped: El bean tiene una única instancia durante la vida de toda la aplicación, es útil para compartir datos o mantener estado global.
• @RequestScoped: El bean se crea y se destruye para cada solicitud HTTP, su instancia es específica de una solicitud, ideal para datos que solo deben persistir durante la vida de una solicitud.
• @Singleton: Similar a @ApplicationScoped, pero es una anotación estándar de Java EE que también asegura que solo haya una instancia del bean durante toda la vida de la aplicación, en Quarkus, @ApplicationScoped y @Singleton son intercambiables.
•	¿Cómo se define un servicio en Quarkus utilizando @ApplicationScoped?
Para definir un servicio en Quarkus utilizando @ApplicationScoped, simplemente anota la clase del servicio con @ApplicationScoped, esto asegura que se cree una única instancia del servicio para toda la aplicación. Un ejemplo básico sería:
import javax.enterprise.context.ApplicationScoped; 
@ApplicationScoped 
public class MyService { 
// Métodos y lógica del servicio 
}

Luego, se puede inyectar este servicio en otras clases usando @Inject:

import javax.inject.Inject; 
public class MyResource { 
@Inject MyService myService;
// Uso de myService 
}
•	¿Por qué es importante manejar correctamente los alcances (scopes) en Quarkus al crear servicios?
Manejar correctamente los alcances (scopes) en Quarkus es crucial porque afecta el ciclo de vida y el estado de los beans:
•	Rendimiento: @ApplicationScoped usa una sola instancia, lo que puede mejorar el rendimiento y reducir el uso de memoria en comparación con @RequestScoped, que crea instancias nuevas para cada solicitud.
•	Consistencia: @RequestScoped asegura que los datos se mantengan solo durante una solicitud, evitando problemas de concurrencia o datos compartidos entre solicitudes que pueden ocurrir con @ApplicationScoped.
•	Recursos: Escoger el alcance adecuado ayuda a gestionar el uso eficiente de recursos y evita problemas relacionados con la creación y destrucción de instancias.
Usando el alcance correcto garantiza un diseño eficiente, seguro y adecuado para el comportamiento esperado de tu aplicación.

Sección 2: Creación de un ApiResponse Genérico
•	¿Qué es un ApiResponse genérico y cuál es su propósito en un servicio REST?
Un ApiResponse genérico en un servicio REST es un objeto que encapsula la respuesta de una API de manera estándar, proporcionando información adicional como códigos de estado, mensajes de error y datos. Su propósito es:
•	Uniformidad: Ofrecer una estructura consistente para las respuestas de la API, facilitando el manejo y la interpretación de respuestas en el cliente.
•	Claridad: Proporcionar información clara sobre el resultado de una solicitud, incluyendo detalles sobre el éxito o el fallo y mensajes de error específicos.
•	Extensibilidad: Permitir la inclusión de metadatos y detalles adicionales sin cambiar la firma de la API.
Esto ayuda a mantener la comunicación entre el cliente y el servidor organizada y predecible.
•	¿Cómo se implementa una clase ApiResponse genérica en Quarkus?
Para implementar una clase ApiResponse genérica en Quarkus, se debe seguir los siguientes pasos:
1.	Define la clase genérica:
public class ApiResponse<T> {
    private int status;
    private String message;
    private T data;

    // Constructores, getters y setters
    public ApiResponse(int status, String message, T data) {
        this.status = status;
        this.message = message;
        this.data = data;
    }

    public int getStatus() {
        return status;
    }

    public void setStatus(int status) {
        this.status = status;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }
}

2.	Utiliza ApiResponse en un recurso REST:
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/example")
public class ExampleResource {

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public ApiResponse<String> getExample() {
        String exampleData = "Hello, World!";
        return new ApiResponse<>(200, "Success", exampleData);
    }
}

Esta implementación permite que la respuesta de la API sea flexible y uniforme, manejando diferentes tipos de datos en la respuesta.







•	¿Cómo se modifica un recurso REST en Quarkus para que utilice un ApiResponse genérico?
•	Se debe definir la clase ApiResponse genérica (como el paso demostrado anteriormente).
•	Modificar el recurso REST para usar ApiResponse:
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

@Path("/example")
public class ExampleResource {

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public Response getExample() {
        String exampleData = "Hello, World!";
        ApiResponse<String> apiResponse = new ApiResponse<>(200, "Success", exampleData);
        return Response.ok(apiResponse).build();
    }
}

Se debe cambiar el tipo de retorno del método del recurso REST para que sea Response y envuelva el ApiResponse genérico en el cuerpo de la respuesta HTTP, esto estandarizará las respuestas y proporcionará un formato uniforme para la comunicación con el cliente.

•	¿Qué beneficios ofrece el uso de un ApiResponse genérico en términos de mantenimiento y consistencia de código?
El uso de un ApiResponse genérico ofrece los siguientes beneficios en términos de mantenimiento y consistencia de código:
•	Uniformidad: Proporciona una estructura estándar para las respuestas de la API, haciendo que el manejo de respuestas sea consistente en toda la aplicación.
•	Facilidad de mantenimiento: Permite realizar cambios en la estructura de la respuesta en un solo lugar (la clase ApiResponse), lo que simplifica el mantenimiento y evita la duplicación de código.
•	Claridad: Facilita la comprensión del formato de respuesta y los posibles estados de error, mejorando la legibilidad y el manejo de errores en el cliente y en el servidor.
•	Extensibilidad: Permite agregar campos adicionales (como metadatos o detalles de error) sin modificar la interfaz de la API ni cambiar los métodos de los recursos REST.
•	¿Cómo manejarías diferentes tipos de respuestas (éxito, error, etc.) utilizando la clase ApiResponse?
Para manejar diferentes tipos de respuestas (éxito, error, etc.) utilizando la clase ApiResponse, se puede hacer lo siguiente:
1-	Para respuestas exitosas:
ApiResponse<T> successResponse = new ApiResponse<>(200, "Operation successful", data);
2-	Para respuestas de error:
ApiResponse<null> errorResponse = new ApiResponse<>(500, "Internal server error", null);

3-	Para respuestas con mensajes de advertencia:
ApiResponse<T> warningResponse = new ApiResponse<>(400, "Warning message", data);
Resumiendo: se debe utilizar el código de estado y el mensaje para diferenciar entre éxito, error y advertencias, y ajustar el campo data según sea necesario. Esto proporcionará claridad en la respuesta y permitirá manejar diferentes casos de manera estandarizada.
Sección 3: Integración y Buenas Prácticas
•	¿Qué consideraciones se deben tener al inyectar servicios en un recurso REST en Quarkus?
Al inyectar servicios en un recurso REST en Quarkus, se debe considerar lo siguiente:
1.	Ámbito del servicio: Asegúrarse de que el servicio tenga un ámbito adecuado (@ApplicationScoped, @RequestScoped, etc.) para evitar problemas de estado compartido o instancias innecesarias.
2.	Manejo de errores: Los servicios deben manejar errores adecuadamente para que las respuestas del recurso REST sean robustas y gestionen correctamente los fallos.
3.	Dependencias: Verificar que todas las dependencias del servicio estén correctamente configuradas e inyectadas para evitar errores de inicialización o problemas de dependencia.
4.	Ciclo de vida: Considerar el ciclo de vida del servicio para asegurarte de que se gestione correctamente la creación y destrucción de instancias, especialmente si el servicio mantiene estado.
•	¿Cómo se pueden manejar excepciones en un servicio REST utilizando ApiResponse?
Para manejar excepciones en un servicio REST utilizando ApiResponse, se pueden seguir estos pasos:
1-	Define una respuesta de error en ApiResponse:
ApiResponse<null> errorResponse = new ApiResponse<>(500, "Error message", null);
2-	Utiliza un manejador global de excepciones (con @ExceptionMapper):
import javax.ws.rs.ext.ExceptionMapper;
import javax.ws.rs.ext.Provider;

@Provider
public class GenericExceptionMapper implements ExceptionMapper<Exception> {
    @Override
    public Response toResponse(Exception exception) {
        ApiResponse<null> apiResponse = new ApiResponse<>(500, exception.getMessage(), null);
        return Response.status(500).entity(apiResponse).build();
    }
}
3-	En el recurso REST, captura excepciones específicas y retorna la respuesta adecuada:
@GET
@Produces(MediaType.APPLICATION_JSON)
public Response getExample() {
    try {
        // Lógica del servicio
        String exampleData = "Hello, World!";
        ApiResponse<String> apiResponse = new ApiResponse<>(200, "Success", exampleData);
        return Response.ok(apiResponse).build();
    } catch (SomeSpecificException e) {
        ApiResponse<null> errorResponse = new ApiResponse<>(400, "Specific error message", null);
        return Response.status(400).entity(errorResponse).build();
    }
}

Resumiendo: Se debe definir una estructura de error en ApiResponse, usar un manejador global para excepciones no capturadas, y manejar excepciones específicas en el recurso REST para proporcionar respuestas consistentes y manejables.
