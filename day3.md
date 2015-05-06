# Day 3

- [CDI Cont'd: Using Qualifiers](#cdi-contd-using-qualifiers)
- [Dealing With Data](#dealing-with-data)
- [Websockets](#websockets)

## CDI Cont'd: Using Qualifiers
To create a qualifier, first create a new annotation.  For example, if we have a dependency called ActionRepository.  Then, our new annotation would be:
```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE,ElementType.METHOD,ElementType.FIELD,ELementType.PARAMETER})
public @interface Action {}
```

The process of matching a bean to an injection point is called _typesafe resolution._  If you don't use a qualifier annotation like above, then the dependency/bean will automatically use @Default annotation. 

###Using CDI with Servlets
The `@Model` annotation allows us to say an object is going to be mapped to the frontend (controller/servlet/JSP).  Similarily inorder for a bean to hold state during a client request we must define its scope.  We do this using annotations such as: `@RequestScoped` (used for a single HTTP request) and `@ApplicationScoped` (used when you want to share state of the bean across all requests, ie you can use this on a DAO).  We use `@Produces` for fields in a `@Model` object that we want to be mapped to the request scope.  [More on it here.](http://docs.oracle.com/javaee/6/tutorial/doc/gjbbk.html)

##Dealing With Data
###JDBC
The Java Database Connectivity (JDBC) API allows us to interface with data sources (not limited to relational db).  As far as db connectivity goes, a pool of connections is created at container startup that will provide commands (close, get, use) to db connection requests from your code.  This allows for efficient resourse reuse (pooling).  

To setup a datasource, simply define it as a resource in your context.xml.  This basically makes it accessible to us in our code via JNDI like so:
```java
@Resource(name = 'jdbc/WebShopDB')
private DataSource ds;

// later in your code, you would do:
ds.getConnection();
```

###Java Persistance API (JPA)
If you've used Hibernate, this is a generacized version of it.

##Websockets
This isn't new but was covered in the course.  Websockets allow for full-duplex (as opposed to HTTP which is half-duplex) async messaging.  To setup websockets, you need to setup both endpoints (server and client).  In Java, use the `@ServerEndpoint('/chat')` annotation to setup the server endpoint.  Then in that class, create `void onOpen(Session s)`, `onMessage`, `onClose` methods.  Since Websockets is a standard, you can also have your clientside endpoint using Javascript via `var webSocket = new WebSocket("ws://localhost:8080/project/chat")`.

You can also define custom configurations for your endpoints such as having a decoders/encoders (for when you are sending more than a simple primitive type).
```java
@ServerEndpoint(value="/auction", decoders=AuctionMessageDecoder.class)
```

[Index](readme.md) | [Go to Day 4 >>](/day4.md)