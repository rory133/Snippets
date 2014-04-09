### Session Context
Session beans are business components that live in a container. Usually, they don’t access the container or use the
container services directly (transaction, security, dependency injection, etc.). These services are meant to be handled
transparently by the container on the bean’s behalf (this is called inversion of control). However, it is sometimes
necessary for the bean to explicitly use container services in code (such as explicitly marking a transaction to be
rolled back). And this can be done through the javax.ejb.SessionContext interface. The SessionContext allows
programmatic access to the runtime context provided for a session bean instance. SessionContext extends the
javax.ejb.EJBContext interface.

getCallerPrincipal  | Returns the java.security.Principal associated with the invocation
getRollbackOnly | Tests whether the current transaction has been marked for rollback.
getTimerService | Returns the javax.ejb.TimerService interface. Only stateless beans and singletons can use this method. Stateful session beans cannot be timed objects.
getUserTransaction | Returns the javax.transaction.UserTransaction interface to demarcate transactions. Only session beans with bean-managed transaction (BMT) can use this method.
isCallerInRole | Tests whether the caller has a given security role.
lookup | Enables the session bean to look up its environment entries in the JNDI naming context.
setRollbackOnly | Allows the bean to mark the current transaction for rollback.
wasCancelCalled | Checks whether a client invoked the cancel() method on the client Future object corresponding to the currently executing asynchronous business method.

Example:
```
@Stateless
public class ItemEJB {

  @PersistenceContext(unitName = "chapter07PU")
  private EntityManager em;
  
  @Resource
  private SessionContext context;
  
  public Book createBook(Book book) {
    if (!context.isCallerInRole("admin"))
      throw new SecurityException("Only admins can create books");

    em.persist(book);
    if (inventoryLevel(book) == TOO_MANY_BOOKS)
      context.setRollbackOnly();
    return book;
  }
}
```

### Asynchronous Calls
By default, session bean invocations through remote, local, and no-interface views are synchronous. Since EJB 3.1, you can call methods asynchronously simply by annotating a session bean method with
@javax.ejb.Asynchronous:
```
@Stateless
public class OrderEJB {
@Asynchronous
  public void sendEmailOrderComplete(Order order, Customer customer) {
  // Very Long task
  }
  
  @Asynchronous
    public void printOrder(Order order) {
  // Very Long task
  }
}
```

### Environment Naming Context
Environment entries are specified in the deployment descriptor and are accessible via dependency injection. They support the following Java types: String, Character, Byte, Short, Integer, Long, Boolean, Double,
and Float. Listing below shows the ejb-jar.xml file of ItemEJB defining two entries: currencyEntry of type String
with the value Euros and a changeRateEntry of type Float with the value 0.80:
```
<ejb-jar xmlns="http://xmlns.jcp.org/xml/ns/javaee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
http://xmlns.jcp.org/xml/ns/javaee/ejb-jar_3_2.xsd"
version="3.2">
  <enterprise-beans>
    <session>
      <ejb-name>ItemEJB</ejb-name>
      <env-entry>
        <env-entry-name>currencyEntry</env-entry-name>
        <env-entry-type>java.lang.String</env-entry-type>
        <env-entry-value>Euros</env-entry-value>
      </env-entry>
      <env-entry>
        <env-entry-name>changeRateEntry</env-entry-name>
        <env-entry-type>java.lang.Float</env-entry-type>
        <env-entry-value>0.80</env-entry-value>
      </env-entry>
    </session>
  </enterprise-beans>
</ejb-jar>
```
After defining environment entries, they can be injected:
```
@Stateless
public class ItemEJB {
  
  @Resource(name = "currencyEntry")
  private String currency;

  @Resource(name = "changeRateEntry")
  private Float changeRate;
  
  public Item convertPrice(Item item) {
    item.setPrice(item.getPrice() * changeRate);
    item.setCurrency(currency);
  return item;
  }
}
```
