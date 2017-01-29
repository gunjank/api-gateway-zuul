# api-gateway-zuul - Based on Spring Zuul example

### Create  Gateway micro-service 

Spring Cloud Netflix includes an embedded Zuul proxy, which we can enable with the @EnableZuulProxy annotation. This will turn the Gateway application into a reverse proxy that forwards relevant calls to other services---such as our Book service.

Open GatewayApplication class and add the annotation, like so:
```
gateway/src/main/java/hello/GatewayApplication.java

package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@SpringBootApplication
public class GatewayApplication {

  public static void main(String[] args) {
    SpringApplication.run(GatewayApplication.class, args);
  }

}
```

To forward requests from the Gateway application, we need to tell Zuul the routes that it should watch and the services to which to forward requests to those routes. We specify routes using properties under zuul.routes. Each of our microservices can have an entry under zuul.routes.NAME, where NAME is the application name (as stored in the spring.application.name property).

Add the application.properties file to a new directory, src/main/resources, in the Gateway application. It should look like this:
```
gateway/src/main/resources/application.yml
```
[src/main/resources/application.yml](src/main/resources/application.yml)

Spring Cloud Zuul will automatically set the path to the application name. In this sample because we set zuul.routes.books.url, so Zuul will proxy requests to /books to this URL.

Notice the second-to-last property in our file: Spring Cloud Netflix Zuul uses Netflix’s Ribbon to perform client-side load balancing, and by default, Ribbon would use Netflix Eureka for service discovery. For this simple example, we’re skipping service discovery, so we’ve set ribbon.eureka.enabled to false. Since Ribbon now can’t use Eureka to look up services, we must specify a url for the Book service.

### Add a filter

Now let’s see how we can filter requests through our proxy service. Zuul has four standard filter types:

pre filters are executed before the request is routed,

routing filters can handle the actual routing of the request,

post filters are executed after the request has been routed, and

error filters execute if an error occurs in the course of handling the request.

We’re going to write a pre filter. Spring Cloud Netflix picks up, as a filter, any @Bean which extends com.netflix.zuul.ZuulFilter and is available in the application context. Create a new directory, src/main/java/hello/filters/pre, and within it, create the filter file, SimpleFilter.java:
```
gateway/src/main/java/hello/filters/pre/SimpleFilter.java
```
link:complete/gateway/src/main/java/hello/filters/pre/SimpleFilter.java[]
Filter classes implement four methods:

filterType() returns a String that stands for the type of the filter---in this case, pre, or it could be route for a routing filter.

filterOrder() gives the order in which this filter will be executed, relative to other filters.

shouldFilter() contains the logic that determines when to execute this filter (this particular filter will always be executed).

run() contains the functionality of the filter.

Zuul filters store request and state information in (and share it by means of) the RequestContext. We’re using that to get at the HttpServletRequest, and then we log the HTTP method and URL of the request before it is sent on its way.

The GatewayApplication class is annotated with @SpringBootApplication, which is equivalent to (among others) the @Configuration annotation that tells Spring to look in a given class for @Bean definitions. Add one for our SimpleFilter here:
```
gateway/src/main/java/hello/GatewayApplication.java
```
[src/main/java/hello/GatewayApplication.java](src/main/java/hello/GatewayApplication.java)

### Trying it out


Visit one of the sample json service or github user service, as http://localhost:8080/1/posts/ or http://localhost:8080/2/gujank, and you should see your request’s method logged by the Gateway application before it’s handed on to the routed application:

2017-01-28 22:58:50.857  INFO 51076 --- [nio-8080-exec-5] hello.filters.pre.SimpleFilter           : GET request to http://localhost:8080/2/gujank
2017-01-28 22:58:54.532  INFO 51076 --- [nio-8080-exec-6] hello.filters.pre.SimpleFilter           : GET request to http://localhost:8080/1/posts/

Summary

Congratulations! You’ve just used Spring to develop a api-gateway service application that can proxy and filter requests for your microservices.
