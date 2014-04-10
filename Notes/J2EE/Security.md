#### Main points.
There are two types of implementations: declarative(throught annotations) and programatic(SessionContext API). 

#### Programmatic Authorization.
The SessionContext interface defines the following methods related to security:
* isCallerInRole(): This method returns a boolean and tests whether the caller has a given security role.
* getCallerPrincipal(): This method returns the java.security.Principal that identifies the caller.

#### Declarative 

| Annotation        |Bean |Method           | Description  |
| ------------- |:---:|:-------------:| :-----|
| @PermitAll      |X| X | Indicates that the given method (or the entire bean) is accessible by everyone (all roles are permitted).|
| @DenyAll      |X| X      |   Indicates that no role is permitted to execute the specified method or all methods of the bean (all roles are denied). This can be useful if you want to deny access to a method in a certain environment
(e.g., the method launchNuclearWar() should only be allowed in production but not in a test environment). |
| @RolesAllowed |X| X      |    Indicates that a list of roles is allowed to execute the given method (or the entire bean).|
| @DeclareRoles |X|-      |  Defines roles for security checking. |
| @RunAs |X| -     |  Temporarily assigns a new role to a principal. |

#### Examples
```
@Stateless
@RolesAllowed({"user", "employee", "admin"})
public class ItemEJB {
  @PersistenceContext(unitName = "chapter08PU")
  private EntityManager em;
  
  @PermitAll
  public Book findBookById(Long id) {
    return em.find(Book.class, id);
  }
  
  public Book createBook(Book book) {
    em.persist(book);
  return book;
  }
  
  @RolesAllowed("admin")
  public void deleteBook(Book book) {
    em.remove(em.merge(book));
  }
  
  @DenyAll
  public Book findConfidentialBook(Long secureId) {
    return em.find(Book.class, secureId);
  }
}

```
ItemEJB in listing below authorizes access to the user, employee, and admin role. When one of
these roles accesses a method, the method is run with the temporary inventoryDpt role (@RunAs("inventoryDpt")).
This means that when you invoke the createBook() method, the InventoryEJB.addItem() method will be invoked
with an inventoryDpt role.
```
@Stateless
@RolesAllowed({"user", "employee", "admin"})
@RunAs("inventoryDpt")
public class ItemEJB {
  @PersistenceContext(unitName = "chapter08PU")
  private EntityManager em;
  
  @EJB
  private InventoryEJB inventory;
  
  public List<Book> findBooks() {
    TypedQuery<Book> query = em.createNamedQuery("findAllBooks", Book.class);
    return query.getResultList();
  }
  
  public Book createBook(Book book) {
    em.persist(book);
    inventory.addItem(book);
    return book;
  }
}
```
