### Overview
* EJBs are server-side components that encapsulate business logic and take care of transactions and security.
* A session bean may have the following traits:
..* Stateless: The session bean contains no conversational state between methods, and any
instance can be used for any client. It is used to handle tasks that can be concluded with
a single method call. For stateless session beans, the persistence context is transactional.
..* Stateful: The session bean contains conversational state, which must be retained across
methods for a single user. It is useful for tasks that have to be done in several steps.
..* Singleton: A single session bean is shared between clients and supports concurrent access.
The container will make sure that only one instance exists for the entire application.
* EJB container provides core services common to many enterprise applications such as the following:
..* Remote client communication: Without writing any complex code, an EJB client (another EJB, a user interface, a batch process, etc.) can invoke methods remotely via standard protocols.
..* Dependency injection: The container can inject several resources into an EJB (JMS destinations
and factories, datasources, other EJBs, environment variables, etc.) as well as any POJO thanks to CDI.
..* State management: For stateful session beans, the container manages their state transparently. You can maintain state for a particular client, as if you were developing a desktop application.
..* Pooling: For stateless beans and MDBs, the container creates a pool of instances that can be shared by multiple clients. Once invoked, an EJB returns to the pool to be reused instead of being destroyed.
..* Component life cycle: The container is responsible for managing the life cycle of each component.
..* Messaging: The container allows MDBs to listen to destinations and consume messages without too much JMS plumbing.
..* Transaction management: With declarative transaction management, an EJB can use annotations to inform the container about the transaction policy it should use. The container takes care of the commit or the rollback.
..* Security: Class or method-level access control can be specified on EJBs to enforce user and role authorization
..* Concurrency support: Except for singletons, where some concurrency declaration is needed, all the other types of EJB are thread-safe by nature. You can develop high-performance applications without worrying about thread issues.
..* Interceptors: Cross-cutting concerns can be put into interceptors, which will be invoked automatically by the container.
..* Asynchronous method invocation: Since EJB 3.1, it’s now possible to have asynchronous calls without involving messaging.
* Containers give EJBs a set of service. On the other hand, EJBs cannot create or manage threads, access files using
java.io, create a ServerSocket, load a native library, or use the AWT (Abstract Window Toolkit)or Swing APIs to
interact with the user.
*  Comparison Between EJB Lite and Full EJB
Feature|EJB Lite|Full EJB 3.2
Session beans (stateless, stateful, singleton)| Yes|Yes
No-interface view |Yes| Yes
Local interface| Yes |Yes
Interceptors| Yes |Yes
Transaction support Yes |Yes
Security |Yes |Yes
Embeddable API| Yes| Yes
Asynchronous calls |No| Yes
MDBs| No| Yes
Remote interface| No |Yes
JAX-WS web services |No| Yes
JAX-RS web services |No |Yes
Timer service |No| Yes
RMI/IIOP interoperability |No |Yes

* A session bean class is any standard Java class that implements business logic. The requirements to develop a session bean class are as follows:
..* The class must be annotated with @Stateless, @Stateful, @Singleton, or the XML equivalent
in a deployment descriptor.
..* It must implement the methods of its interfaces, if any.
..* The class must be defined as public, and must not be final or abstract.
..* The class must have a public no-arg constructor that the container will use to create instances.
..* The class must not define the finalize() method.
..* Business method names must not start with ejb, and they cannot be final or static.
..* The argument and return value of a remote method must be legal RMI types.

* Simple EJB:
```
@Stateless
public class BookEJB {
  @PersistenceContext(unitName = "chapter07PU")
  private EntityManager em;
  
  public Book findBookById(Long id) {
    return em.find(Book.class, id);
  }
  
  public Book createBook(Book book) {
    em.persist(book);
  return book;
  }
}
```

* A session bean can implement several interfaces or none. A business interface is a standard Java interface that
does not extend any EJB-specific interfaces. Like any Java interface, business interfaces define a list of methods that will be available for the client application. They can use the following annotations:
..* @Remote: Denotes a remote business interface. Method parameters are passed by value and
need to be serializable as part of the RMI protocol.
..* @Local: Denotes a local business interface. Method parameters are passed by reference from
the client to the bean.

* You cannot mark the same interface with more than one annotation. The no-interface view is a variation of the local view that exposes all public business methods of the bean class locally without the use of a separate business interface.

* Listing shows a local interface (ItemLocal) and a remote interface (ItemRemote) implemented by the ItemEJB
stateless session bean. With this code, clients will be able to invoke the findCDs() method locally or remotely as it is defined in both interfaces. The createCd() will only be accessible remotely through RMI.
```
@Local
public interface ItemLocal {
  List<Book> findBooks();
  List<CD> findCDs();
}

@Remote
public interface ItemRemote {
  List<Book> findBooks();
  List<CD> findCDs();
  Book createBook(Book book);
  CD createCD(CD cd);
}

@Stateless
public class ItemEJB implements ItemLocal, ItemRemote {
  // ...
}
```
Alternatively to the code in Listing, you might specify the interface in the bean’s class. In this case, you would
have to include the name of the interface in the @Local and @Remote annotations as shown in Listing below. This is
handy when you have legacy interfaces where you can’t add annotations and need to use them in your session bean.
```
public interface ItemLocal {
  List<Book> findBooks();
  List<CD> findCDs();
}
public interface ItemRemote {
  List<Book> findBooks();
  List<CD> findCDs();
  Book createBook(Book book);
  CD createCD(CD cd);
}
@Stateless
@Remote(ItemRemote.class)
@Local(ItemLocal.class)
@LocalBean
public class ItemEJB implements ItemLocal, ItemRemote {
  // ...
}
```
If the bean exposes at least one interface (local or remote) it automatically loses the no-interface view. It then
needs to explicitly specify that it exposes a no-interface view by using the @LocalBean annotation on the bean class.
As you can see in Listing above the ItemEJB now has a local, remote, and no interface.

* In addition to remote invocation through RMI, stateless beans can also be invoked remotely as SOAP web services
or RESTful web services. Listing shows a stateless bean with a local interface, a SOAP web services endpoint (@WebService), and a RESTful web service endpoint (@Path). Note that these annotations come, respectively, from JAX-WS and JAX-RS and are not part of EJB.
```
@Local
public interface ItemLocal {
List<Book> findBooks();
List<CD> findCDs();
}
@WebService
public interface ItemSOAP {
List<Book> findBooks();
List<CD> findCDs();
Book createBook(Book book);
CD createCD(CD cd);
}
@Path(/items)
public interface ItemRest {
List<Book> findBooks();
}
@Stateless
public class ItemEJB implements ItemLocal, ItemSOAP, ItemRest {
// ...
}
```

#### Portable JNDI
* The Java EE specification defines portable JNDI names with the following syntax:
```
java:<scope>[/<app-name>]/<module-name>/<bean-name>[!<fully-qualified-interface-name>]
```
Each portion of the JNDI name has the following meaning:
..* <scope> defines a series of standard namespaces that map to the various scopes of a Java EE application:
...* global: The java:global prefix allows a component executing outside a Java EE application to access a global namespace.
...* app: The java:app prefix allows a component executing within a Java EE application to access an application-specific namespace.
...* module: The java:module prefix allows a component executing within a Java EE application to access a module-specific namespace.
...* comp: The java:comp prefix is a private component-specific namespace and is not accessible by other components.
..* <app-name> is only required if the session bean is packaged within an ear or war file. If this is
the case, the <app-name> defaults to the name of the ear or war file (without the .ear or .war
file extension).
..* <module-name> is the name of the module in which the session bean is packaged. It can be an EJB module in a stand-alone jar file or a web module in a war file. The <module-name> defaults to the base name of the archive with no file extension.
..* <bean-name> is the name of the session bean.
..* <fully-qualified-interface-name> is the fully qualified name of each defined business interface. For the no-interface view, the name can be the fully qualified bean class name.

For example we have EJB:
```
package org.agoncal.book.javaee7;
@Stateless
@Remote(ItemRemote.class)
@Local(ItemLocal.class)
@LocalBean
public class ItemEJB implements ItemLocal, ItemRemote {
// ...
}
```
Once deployed, the container will create three JNDI names so an external component will be able to access the
ItemEJB using the following global JNDI names
```
java:global/cdbookstore/ItemEJB!org.agoncal.book.javaee7.ItemRemote
java:global/cdbookstore/ItemEJB!org.agoncal.book.javaee7.ItemLocal
java:global/cdbookstore/ItemEJB!org.agoncal.book.javaee7.ItemEJB
```

#### Aassivation and activation.
The one-to-one correlation comes at a price because, as you might have guessed, if you have one million clients, you
will get one million stateful beans in memory. To avoid such a big memory footprint, the container temporarily clears stateful beans from memory before the next request from the client brings them back. This technique is called passivation and activation. Passivation is the process of removing an instance from memory and saving it to a persistent location (a file on a disk, a database, etc.). It helps you to free memory and release resources (a database or JMS connections, etc.). Activation is the inverse process of restoring the state and applying it to an instance. Passivation and activation are done automatically by the container; you shouldn’t worry about doing it yourself, as it’s a container service. What you should worry about is freeing any resource (e.g., database connection, JMS factories connection, etc.) before the bean is passivated. Since EJB 3.2, you can also disable passivation as you’ll see in the next chapter with life-cycle and callback annotations.


### Statefull session beans.
#### Examples
```
@Stateful
@StatefulTimeout(value = 20, unit = TimeUnit.SECONDS)
public class ShoppingCartEJB {

  private List<Item> cartItems = new ArrayList<>();
  
  public void addItem(Item item) {
    if (!cartItems.contains(item))
    cartItems.add(item);
  }

  public void removeItem(Item item) {
    if (cartItems.contains(item))
    cartItems.remove(item);
  }
  
  public Integer getNumberOfItems() {
    if (cartItems == null || cartItems.isEmpty())}
      return 0;
    }
    return cartItems.size();
  }

  public Float getTotal() {
    if (cartItems == null || cartItems.isEmpty())}
      return 0f;
    }
    Float total = 0f;
      for (Item cartItem : cartItems) {
        total += (cartItem.getPrice());
      }
    return total;
  }
    
  public void empty() {
    cartItems.clear();
  }
  
  // the bean instance to be permanently removed from memory after you invoke the checkout() method
  @Remove
  public void checkout() {
    // Do some business logic
    cartItems.clear();
  }
}
```
