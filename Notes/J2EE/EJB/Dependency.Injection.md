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
