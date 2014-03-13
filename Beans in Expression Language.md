Beans in Expression Language
===
By default, CDI Beans are not assigned any name and are not resolvable via EL binding. To assign a bean a name, 
it must be annotated with the @javax.inject.Named built-in qualifier as shown:
```
@Named
public class BookService {
 
 private String title, description;
 private Float price;
 private Book book;
 
 @Inject @ThirteenDigits
 private NumberGenerator numberGenerator;
 
 public String createBook() {
 book = new Book(title, price, description);
 book.setIsbn(numberGenerator.generateNumber());
 return "customer.xhtml";
 }
}
```

The @Named qualifier allows you to access the BookService bean through its name (which by default is the class 
name in camel case with the first letter in lowercase). The following code shows a JSF button invoking the createBook 
method:

``` 
<h:commandButton value="Send email" action="#{bookService.createBook}"/>
```

You can also override the name of the bean by adding a different name to the qualifier.
``` 
@Named("myService")
public class BookService {...}
```
Then you can use this new name on your JSF page.
``` 
<h:commandButton value="Send email" action="#{myService.createBook}"/>
```
