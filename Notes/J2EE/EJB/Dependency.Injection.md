### Overview
The containers can inject various types of resources into session beans using different annotations:
* @EJB: Injects a reference of the local, remote, or no-interface view of an EJB into the annotated
variable.
* @PersistenceContext and @PersistenceUnit: Expresses a dependency on an EntityManager
and on an EntityManagerFactory, respectively.
* @Resource: Injects several resources such as JDBC data sources, session context, user
transactions, JMS connection factories and destinations, environment entries, the timer
service, and so on.
* @Inject: Injects nearly everything with @Inject and @Produces.

Example:
```
@Stateless
public class ItemEJB {
  @PersistenceContext(unitName = "chapter07PU")
  private EntityManager em;
  
  @EJB
  private CustomerEJB customerEJB;
  
  @Inject
  private NumberGenerator generator;
  
  @WebServiceRef
  private ArtistWebService artistWebService;
  
  private SessionContext ctx;
  
  @Resource
  public void setCtx(SessionContext ctx) {
    this.ctx = ctx;
  }
//...
}
```
### Inject a EJB into managed beans.
Listing below demonstrate how to inject a EJB into other managed beans.
```
@Stateless
@Remote(ItemRemote.class)
@Local(ItemLocal.class)
@LocalBean
public class ItemEJB implements ItemLocal, ItemRemote {...}
```
```
// Client code injecting several references to the EJB or interfaces
@EJB ItemEJB itemEJB;
@EJB ItemLocal itemEJBLocal;
@EJB ItemRemote itemEJBRemote;
``
Also EJB can be found throught  JNDI
``
@EJB(lookup = "java:global/classes/ItemEJB") ItemRemote itemEJBRemote;
``
Most of the time you can just replace @EJB by @Inject and your client code will work. For remote EJBs, as you just saw, you might need to use a JNDI string to look it up. The @Inject annotation cannot have a String parameter, so in this case, you need to produce the remote EJB to be able to inject it:
```
// Code producing a remote EJB
@Produces @EJB(lookup = "java:global/classes/ItemEJB") ItemRemote itemEJBRemote;

// Client code injecting the produced remote EJB
@Inject ItemRemote itemEJBRemote;
```

Depending on your client environment, you might not be able to use injection (if the component is not managed
by a container). In this case, you can use JNDI to look up session beans through their portable JNDI name.
