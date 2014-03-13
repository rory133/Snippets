Qualifiers with Members
========

```
@Qualifier
@Retention(RUNTIME)
@Target({FIELD, TYPE, METHOD})
public @interface NumberOfDigits {
 
 Digits value();
 boolean odd();
}
 
public enum Digits {
 TWO,
 EIGHT,
 TEN,
 THIRTEEN
}
```

The manner in which you would use this qualifier with members doesn’t change from what you’ve seen so far. 
The injection point will qualify the needed implementation by setting the annotation members as follows:

```
@Inject @NumberOfDigits(value = Digits.THIRTEEN, odd = false)
private NumberGenerator numberGenerator;
```

And the concerned implementation will do the same.
```
@NumberOfDigits(value = Digits.THIRTEEN, odd = false)
public class IsbnEvenGenerator implements NumberGenerator {...} 
```

Producers
=========
I’ve shown you how to inject CDI Beans into other CDI Beans. But you can also inject primitives (e.g., int, long, 
float . . .), array types and any POJO that is not CDI enabled, thanks to producers. By CDI enabled I mean any class 
packaged into an archive containing a beans.xml file.
By default, you cannot inject classes such as a java.util.Date or java.lang.String. That’s because all these 
classes are packaged in the rt.jar file (the Java runtime environment classes) and this archive does not contain a 
beans.xml deployment descriptor. If an archive does not have a beans.xml under the META-INF directory, CDI will not 
trigger bean discovery and POJOs will not be able to be treated as beans and, thus, be injectable. The only way to be 
able to inject POJOs is to use producer fields or producer methods as shown:
```
public class NumberProducer {
 
 @Produces @ThirteenDigits
 private String prefix13digits = "13-";
 
 @Produces @ThirteenDigits
 private int editorNumber = 84356;
 
 @Produces @Random
 public double random() {
 return Math.abs(new Random().nextInt());
 }
}
```
Usage example:
```
@ThirteenDigits
public class IsbnGenerator implements NumberGenerator {
 
 @Inject @ThirteenDigits
 private String prefix;
 
 @Inject @ThirteenDigits
 private int editorNumber;
 
 @Inject @Random
 private double postfix;
 
 public String generateNumber() {
 return prefix + editorNumber + postfix;
 }
}
```

InjectionPoint AP
====
In some cases objects need to know something about the injection point into which 
they are injected. This can be a way of configuring or changing behavior depending on the injection point.
Let’s take for example the creation of a logger. In the JDK, to create a java.util.logging.Logger you need to set 
the category of the class that owns it. For example, if you want a logger for the BookService you will write:
``` 
Logger log = Logger.getLogger(BookService.class.getName());
```
How would you produce a Logger that needs to know the class name of the injection point? CDI has an 
InjectionPoint API that provides access to metadata about an injection point (see http://docs.jboss.org/cdi/api/1.0/javax/enterprise/inject/spi/InjectionPoint.html). Thus you need to 
create a producer method that uses the InjectionPoint API to configure the right logger. Listing below shows how the createLogger method gets the injection point class name.

```
public class LoggingProducer {
 
 @Produces
 private Logger createLogger(InjectionPoint injectionPoint) {
 return Logger.getLogger(injectionPoint.getMember().getDeclaringClass().getName());
 }
}
```
And usage:
```
public class BookServiceImpl {
..
@Inject Logger log;
..
}
...
```


