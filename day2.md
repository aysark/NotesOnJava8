# Day 2

- [New Collector Class](#new-collector-class)
- [Revamped Date/Time API](#revamped-datetime-api)
- [Reflections Design Pattern using Proxy Class](#reflections-design-pattern-using-proxy-class)
- [Write Javascript With Java](#write-javascript-with-java)
- [CDI: Java Standard for Dependency Injection & Interception](#cdi-java-standard-for-dependency-injection-interception)
- [Other New Features](#other-new-features)

## New Collector Class
This further expands on the Stream API functionality.
Contains a `groupingBy` method similar to a GROUP BY clause in a SQL query.  
```java
public Map<String, Long> getDVDsInEachGenre() {
	Map<String, Long> collect = DVDDao.getAllDVDs().collect(Collectors.groupingBy(DVD::getGenre, Collectors.counting()));
	return collect;
}
```
Contains neat summarizing methods (though, don't expect it to print it out pretty).
```java
public DoubleSummaryStatistics getPricingStats() {
	return DVDDao.getAllDVDs().collect(Collectors.summarizingDouble(DVD::getPrice));
}
```

##Revamped Date/Time API
Thread safe classes, domain-driven design.  Manipulations can be chained (Fluent API).
LocalDate (date only), LocalTime (time only) - note these are immutable and when manipulations are made, new instances are created.  Example:
```
LocalDate ld = LocalDate.of(2015, Month.FEBRUARY, 29);
ld.plusDays(1);
System.out.println(ld) // will print 2015-02-29 not 2015-03-01
```

TemporalAdjuster allows for more complex date manipulations.

```java
public static Appointment createFirstDayOfNextMonthAppointment(String description) {
	return new Appointment(description, LocalDateTime.of(LocalDate.now(), LocalTime.NOON).with(TemporalAdjusters.firstDayOfNextMonth()));
}
```

## Reflections Design Pattern using Proxy Class
Imagine you had a Business object interface and its impl.  And you had a TransactionManager that you want decoupled from Business obj, but want it to manage the transactional methods that are called by bus obj.

Build an invoker as so (note: this code is rough, not compiled):
```java
public class MyInvoker impelements InvocationHandler }
	BusinessObj target;
	TransactionManager mgr;

	public MyInvoker(BusinessObject target, TransactionMgr mgr) {
		this.target = target;
		this.mgr = mgr;
	}

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
		Object result = null;
		mgr.startTransaction();
		try {
			result = method.invoke(target, args);
			mgr.commit();
		} catch( Exception e) {
			mgr.rollback();
		}
		return result;
	}
}
```

And to make the proxy handle it:
```java
public class BusinessObjectFactory {
	public static BusinessObject getBusinessObject() {
		BusinessObject t = new BusinessObjectImpl();
		TransactionManager mgr = new TransactionMgr();
		InvocationHandler inv = new MyInvoker(t, mgr);
		try {
			// proxy creates a class that impls BusinessObject interface
			proxy = (BusinessObject) Proxy.newProxyInstance(BusinessObject.class.getClassLoader(), new Class[] {BusinessObject.class}, inv);
		} catch( Exception e) {

		}
		return proxy
}
}
```
And to use it:
```java
public static void main(String[] args) {
	BusinessObject o = BusinessObjectFactoru.getBusinessObject()
	o.doStuff();
	System.out.println(o);
	System.out.println(o.getClass().getName()); // will be proxy
}
```

[More on it here.](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Proxy.html)

##Write Javascript With Java
Java 8 comes with the Nashorn Javascript Engine which allows you to write and eval Javascript code inside your Java code. There's an icecube joke somewhere here...

Can access Javascript variables/methods from Java.  Javascript scripts can implement Java interfaces. 

_Some_ will ask why?

Suppose we have an email validation javascript function in a file (it implements our Java Validator interface).  Here is an example of how we can invoke that Javascript function from Java.
```java
public static boolean validateEmail(String email) throws ScriptException, NoSuchMethodException {

	// Get an instance of the ScriptEngineManager
	ScriptEngineManager sem = new ScriptEngineManager();
	// Get the Nashorn scriptengine
	ScriptEngine se = sem.getEngineByName("Nashorn");

	InputStream resource = ValidationService.class
			.getResourceAsStream("/Validation.js");

	// evaluate the Validation resource
	se.eval(new InputStreamReader(resource));

	// Get the Validator for validation of emails
	Validator v = (Validator)  se.get("emailValidator");

	// validate the email and return the result
	return (boolean) v.validate(email);
}
```

##CDI: Java Standard for Dependency Injection & Interception
Context and Dependency Injection (CDI) allows us to ensure loosely coupled modules for our app.  Following the principles of inversion of control/hollywood principle.  Now Java ships with CDI, a DI framework, allowing us to use `@Inject` annotation to inject a dependency.  For example:
```java
	@Inject
	private BlurayRepository repository;
```

This assumes we have a BlurayRepository interface and atleast one impl (otherwise we will get an ExceptionInInitializerError).    

##Other New Features
`Arrays.parallelSort`` assigns sorting tasks to multiple threads.  More performant for larger arrays.

Improved diamond operator usage, can do: `print(new Shop<>)`

[Index](readme.md) | [Go to Day 3 >>](/day3.md)
