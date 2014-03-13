Decorators
=====
Let’s take the example of an ISSN number generator. ISSN is an 8-digit number that has been replaced by ISBN 
(13-digit number). Instead of having two separate number generators (such as the one in Listing 2-9 and Listing 2-10) 
you can decorate the ISSN generator to add an extra algorithm that turns an 8-digit number into a 13-digit number. 
Listing 2-34 implements such an algorithm as a decorator. The FromEightToThirteenDigitsDecorator class is 
annotated with javax.decorator.Decorator, implements business interfaces (the NumberGenerator defined in 
Figure 2-3), and overrides the generateNumber method (a decorator can be declared as an abstract class so that it does 
not have to implement all the business methods of the interfaces if there are many). The generateNumber() method 
invokes the target bean to generate an ISSN, adds some business logic to transform such a number, and returns an 
ISBN number.

```
@Decorator
public class FromEightToThirteenDigitsDecorator implements NumberGenerator {
 
 @Inject @Delegate
 private NumberGenerator numberGenerator;
 
 public String generateNumber() {
 String issn = numberGenerator.generateNumber();
 String isbn = "13-84356" + issn.substring(1);
 return isbn;
 }
}
```
Decorators must have a delegate injection point (annotated with @Delegate), with the same type as the beans 
they decorate (here the NumberGenerator interface). It allows the decorator to invoke the delegate object (i.e., the 
target bean IssnNumberGenerator) and therefore invoke any business method on it (such as 
numberGenerator.generateNumber().
By default, all decorators are disabled like alternatives and interceptors. You need to enable decorators in the 
beans.xml as shown.
```
<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
 http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
 version="1.1" bean-discovery-mode="all">
 
 <decorators>
 <class>org.agoncal.book.javaee7.chapter02.FromEightToThirteenDigitsDecorator</class>
 </decorators>
 
</beans>
```
If an application has both interceptors and decorators, the interceptors are invoked first.
