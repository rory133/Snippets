Events
===
* Events in CDI are not treated asynchronously.
* Event producers fire events using the javax.enterprise.event.Event interface. A producer raises events 
by calling the fire() method, passes the event object, and is not dependent on the observer.

Producer:
```
public class BookService {
 @Inject
 private NumberGenerator numberGenerator;
 @Inject @Added
 private Event<Book> bookAddedEvent;
 @Inject @Removed
 private Event<Book> bookRemovedEvent;
 public Book createBook(String title, Float price, String description) {
 Book book = new Book(title, price, description);
 book.setIsbn(numberGenerator.generateNumber());
 bookAddedEvent.fire(book);
 return book;
 }
 public void deleteBook(Book book) {
 bookRemovedEvent.fire(book);
 }
}
```
Consumer:
```
public class InventoryService {
 
 @Inject
 private Logger logger;
 List<Book> inventory = new ArrayList<>();
 
 public void addBook(@Observes @Added Book book) {
 logger.warning("Adding book " + book.getTitle() + " to inventory");
 inventory.add(book);
 }
 
 public void removeBook(@Observes @Removed Book book) {
 logger.warning("Removing book " + book.getTitle() + " to inventory");
 inventory.remove(book);
 }
}
```

Like most of CDI, event production and subscription are typesafe and allow qualifiers to determine which 
events observers will be observing. @Add and @Remove are qualifiers.
