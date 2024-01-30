### 7 Microservices

>This chapters covers
* Microservices overview and characteristics
* Developing microservices with Camel
* WildFly Swarm and Camel
* Spring Boot and Camel
* Designing for failures
* Netflix Hystrix circuit breakers

The mantra of a microservice is said to be small, nimble, do one thing only, and do that well

Robert C. Martin’s definition of the single responsibility principle, which states: “Gather together those things that change for the same reason, and separate those things that change for different reasons.
A good practice is to provide health checks in your microservices, accessible from known endpoints that the runtime platform uses for constant monitoring. 

Camel is just a library you include in the JVM runtime, and it runs anywhere.

Standalone—Running just Camel
CDI—Running Camel with CDI
WildFly Swarm—We’s see how Camel runs with the lightweight Java EE server
Spring Boot—Running Camel with Spring Boot

Summary of using standalone Camel for microservices
Running Camel standalone is the simplest and smallest runtime for running Camel—it’s just Camel. You can quickly get started, but it has its limits. For example, you need to figure out how to package your application code with the needed JARs from Camel and third-party dependencies, and how to run that as a Java application. You could try to build a fat JAR, but that creates problems if duplicate files need to be merged together. But if you’re using Docker containers, you can package your application together in a Docker image, with all the JARs and your application code separated, and still make it easy to run your application.

CDI Camel as microservice
What is CDI? Contexts and Dependency Injection (CDI) is a Java specification for dependency injection that includes a set of key features:

Dependency injection—Dependency injection of beans.
POJOs—Any kind of Java bean can be used with CDI.
Lifecycle management—Perform custom action on bean creation and destruction.
Events—Send and receive events in a loosely coupled fashion.
Extensibility—Pluggable extensions can be installed in any CDI runtime to customize behavior.
Camel provides support for CDI by the camel-cdi component, which contains a set of CDI extensions that make it easy to set up and bootstrap Camel with CDI.

```
@Singleton     
public class HelloRoute extends RouteBuilder {

    public void configure() throws Exception {
        from("jetty:http://localhost:8080/hello")
            .transform().simple("Hello from Camel");
    }
}
```

The only code from CDI is the @Singleton (javax.inject.Singleton) annotation ❶.

Configuring CDI applications

In CDI, you use @Produces and @Named annotations to create and configure beans. You use this to create and configure an instance of PropertiesComponent, as shown in the following listing.

```
@Singleton
public class HelloConfiguration {

@Produces     
@Named("properties")     
PropertiesComponent propertiesComponent() {     
PropertiesComponent component = new PropertiesComponent();
component.setLocation("classpath:hello.properties");
return component;
}
}
```

Using @Uri to inject Camel endpoint

```
@Singleton
public class HelloRoute extends RouteBuilder {

    @Inject
    private HelloBean hello;

    @Inject @Uri("jetty:http://localhost:8080/hello")     
private Endpoint jetty;

    public void configure() throws Exception {
        from(jetty)     
.bean(hello, "sayHello");
}
}
```

Listening to Camel events using CDI

```
@Singleton
public class HelloConfiguration {

void onContextStarted(@Observes CamelContextStartedEvent event) {     
System.out.println("***************************************");
System.out.println("* Camel started " + event.getContext().getName());
System.out.println("***************************************");
}
}
```

This concludes our coverage of using Camel with CDI, including coverage of three of the most common, key features of CDI and camel-cdi:

Dependency injection using @Inject and @Produces
Injecting a Camel endpoint using @Uri
Bean lifecycle using scopes such as @Singleton
Event listening using @Observes

Spring Boot with Camel as microservice

Spring Boot is an opinionated framework for building microservices. Spring Boot is designed with convention over configuration and allows developers to get started quickly developing microservices with reduced boilerplate, configuration, and fuss. Spring Boot does this via the following:

Autoconfiguration and reduced configuration needed
Curated list of starter dependencies
Simplified application packaging as a standalone fat JAR
Optional application information, insights, and metrics

One of the best ways to use Camel from an existing Java class such as a Spring controller is to inject a Camel ProducerTemplate or FluentProducerTemplate

Using Camel routes with Spring Boot

When creating Camel routes in Java DSL with Spring Boot, you should annotate the class with @Component, which ensures that the route is autodetected by Spring and automatically added to Camel when the application starts up ❶. Because we want to expose a rest service from a Camel route, we need to use an HTTP server component. Spring Boot comes with a servlet engine out of the box, therefore we need to configure a servlet to be used. You can do this easily with Camel by adding the camel-servlet-starter dependency to the Maven pom.xml file. Camel will then automatically detect the CamelServlet and integrate that with the servlet engine within Spring Boot. Camel will by default use the context-path /camel/*, which can be reconfigured in the application.properties file.

In Camel, you can define REST services using a DSL known as Rest DSL ❷, which will use the servlet component that’s just been configured. The Camel route ❸ comes next. Notice that the Rest DSL calls the route by using the direct endpoint, which is how you can link them together.

- Monitoring Camel with Spring Boot
 ` <dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-actuator</artifactId>
  </dependency>`

The REST service is using Camel’s jetty component ❶ as the HTTP server. To work with JSON from Java POJO classes, we turned on JSON binding ❷, which will use Jackson under the covers. Jackson is instructed to not fail if binding from an empty list/bean ❸. This is needed in case the shopping cart is empty and the GET service ❻ is called. Swagger is turned on for API documentation ❹. The Rest DSL then exposes three REST services ❺ ❻ ❼, each calling the CartService bean ❽. Notice that each REST service documents its input and output types and parameters, which becomes part of the Swagger API documentation.

The REST service uses JSON, so you turn on automatic JSON binding ❶, which allows Camel to bind between JSON data format and POJO classes. 

`
restConfiguration().bindingMode(RestBindingMode.json);     
rest("/ratings/{ids}").produces("application/json")     
.get().to("bean:ratingService");
`

Comments about microservices characteristics
Characteristic	Comment
Small in size	                    Each microservice is surely small in size and does one thing and one thing only.
Observable	                        Microservices should be observable from centralized monitoring and management. This is covered in chapter 16.
Design for failure	                An important topic that we haven’t covered sufficiently in this chapter. But there’s more to come in chapter 11, covering error handling.
Highly configurable	                Spring Boot is highly configurable. WildFly Swarm doesn’t yet offer configuration on the same level as Spring Boot. But you can use the Camel properties component for Camel-only configuration.
Smart endpoints and dumb routes	    Can easily be done using Camel in smaller Camel routes.
Testable	                        Camel has great support for testing, which is covered in chapte

Designing for failures
Retry
Circuit Breaker
Bulkhead

Using the Retry pattern to handle failures
The Retry pattern is intended for handling transient failures, such as temporary network outages, by retrying the operation with the expectation that it’ll then succeed. The Camel error handler uses this pattern as its primary functionality. This section only briefly touches on the Retry pattern and Camel’s error handler because chapter 11 is entirely devoted to this topic. But you can use the Retry pattern in the rules prototype containing the following Camel route that calls the inventory service:

public class InventoryRoute extends RouteBuilder {

`public void configure() throws Exception {
from("direct:inventory")
.to("jms:queue:inventory")     
.unmarshal().jaxb("camelinaction");
}
}`

- To handle failures when calling the inventory microservice ❶, we can configure Camel’s error handler to retry the operation:

public void configure() throws Exception {
errorHandler(defaultErrorHandler()
.maximumRedeliveries(5)     
.redeliveryDelay(2000));

    from("direct:inventory")
      .to("jms:queue:inventory")
      .unmarshal().jaxb("camelinaction");
}

Camel’s error handler to retry ❶ up to five times, with a two-second delay between each attempt. For example, if the first two attempts fail, the operation succeeds at the third attempt and can continue to route the message. Only if all attempts fail does the entire operation fail, and an exception is propagated back to the caller from the Camel route.

Which failures
Not all failures are candidates for retrying. For example, network operations such as HTTP calls or database operations are good candidates. But in every case, it depends. An HTTP server may be unresponsive, and retrying the operation in a bit may succeed. Likewise, a database operation may fail because of a locking error due to concurrent access to a table, which may succeed on the next retry. But if the database fails because of wrong credentials, retrying the operation will keep failing, and therefore it isn’t a candidate for retries.

Camel’s error handler supports configuring different strategies for different exceptions. Therefore, you can retry only network issues, and not exceptions caused by invalid credentials. Chapter 11 covers Camel’s error handler.

How often to retry
You also have to consider how frequently you may retry an operation, as well as the total duration. For example, a real-time service may be allowed to retry an operation only a few times, with short delays, before the service must respond to its client. In contrast, a batch service may be allowed many retries with longer delays.

The retry strategy should also consider other factors, such as the SLAs of the service provider. For example, if you call the service too aggressively, the provider may throttle and degrade your requests, or even blacklist the service consumer for a period of time. In a distributed system, a service provider must build in such mechanisms—otherwise, consumers of the services can overload their systems or degrade downstream services. Therefore, the service provider often provides information about the remaining request count allowed. The retry strategy can read the returned request count and attempt only within the permitted parameters.

Idempotency
Another factor to consider is whether calling a service is idempotent. A service is idempotent if calling the service again with the same input parameters yields the same result. A classic example is a banking service: calling the bank balance service is idempotent, whereas calling the withdrawal service isn’t idempotent.

Distributed systems are inherently more complex. Because of remote networks, a request may have been processed by the remote service but not received by the client, which then assumes the operation failed, and retries the same operation, which then may cause unexpected side effects.

Monitoring
Monitoring and tracking retries are important. For example, if some operations have too many retries before they either fail or succeed, it indicates a problem area that needs to be fixed. Without proper monitoring in place, retries may remain unnoticed and affect the overall stability of the syste

- **Camel uses the term redelivery when a message is retried.**