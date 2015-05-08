# Day 4

- [Asynchronous Servlets](#asynchronous-servlets)
- [Spring MVC and REST Support](#spring-mvc-and-rest-support)
- [GoF Patterns](#gof-patterns)

## Asynchronous Servlets
We use servlets (think actions in controllers if you've used Rails or other MVC) to handle requests from the client.  It used to be that a thread was created per client connection, however since Java 4, threads are allocated per client request.  However this can still be problematic for long running requests.  To solve this we make the servlet asyncronous (so that the request is handled by another thread):
```java
@WebServlet(urlPatterns="/SearchFlight", asyncSupported=true)
public class SearchFlightServlet extends HttpServlet {
	@Inject
	private FlightSearchService fss;

	protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
		AsyncContext atx = req.startAsync();
		searchFlights(atx);
	}

	private void searchFlights(AsyncContext atx) {
		atx.start( () -> {
			ServletRequest r = atx.getRequest();
			List<Flight> flights = fss.searchFlights(r.getParameter("From"), r.getParameter("Destination"), r.getParameter("Date"));
			r.setAttribute("flightsFound", flights);
			atx.dispatch("/SearchResults.jsp");
		});
	}
}
```
___One should only spawn new threads within the AsyncContext or using ManagedExecutorService/ManagedScheduledExecutorService classes___  You can also add a listener to be able to monitor the lifecycle of the thread.

Nowadays we have multiple requests/responses sent from within the same page, we can use Server-Sent Events (SSE) to handle them.  SSE uses HTTP and not Websockets and has been standardized as part of HTML5.

##Spring MVC and REST Support
When a request comes in to a Spring Servlet, a handler mapping tries to find the associated controller for that request (using what you configured in your .xml).  There are also pre and post process objects that get invoked before/after controller executes.  There could also be an AfterResponse object that occurs after our response goes to browser.  The handler mapper manages this chain of events.   

The view resolver figures out which view corresponds to the page that we load up.  In a RESTful approach, we suppress the view resolver logic so that we can send our content right from the controller to the browser.

```java
@RequestMapping(value="/user.htm/{id}", method=RequestMethod.POST)
public @ResponseBody String addUser(@ModelAttribute(value="user") User user, BindingResult r, @PathVariable("id") int id) 
```
The `@ResponseBody` is what suppresses view resolver logic and a message converter takes over.  The `@ModelAttribute` is able to take all the request params and bundle it into a User object that we have defined- any validation errors that may occur from bunding will be in the `BindingResult` var.

Also if we were to return anything other than a primitive type, so like a `List<User>` then it would automatiicaly be returned as a JSON.

## GoF Patterns

### Simplified Factory Pattern
Instead of having seperate factories for each product objects and a main abstract factory class.  You can just have a main factory class:
```java
public class Factory {
	public Product getP(String name) throws Exception {
		String fullName = Class.forName("com.mike.products."+name);
		Product p = (Product) c.newInstance;
		return p;
	}
}
```

And then you can use it by `Product p = Factory.getP("ProductA")` and not have multiple specific factories for each product.

### Real Singletons
As of Java 8 and according to our tests, you can no longer instantiate a new instance of a singleton (using Reflections: `Class.forName("com.s.SingletonInstance").newInstance();`).  You will get this error:
```java
Exception in thread "main" java.lang.IllegalAccessException: Class com.s.Main can not access a member of class com.s.SingletonInstance with modifiers "private"
	at sun.reflect.Reflection.ensureMemberAccess(Unknown Source)
	at java.lang.Class.newInstance(Unknown Source)
	at com.s.Main.main(Main.java:6)
``` 

[Index](readme.md) | [Go to Day 5 >>](/day5.md)