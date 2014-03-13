Interceptors
===
Interceptors fall into four types:
* Constructor-level interceptors: Interceptor associated with a constructor of the target class 
(@AroundConstruct),
* Method-level interceptors: Interceptor associated with a specific business method (@AroundInvoke)
* Timeout method interceptors: Interceptor that interposes on timeout methods with @AroundTimeout (only used with EJB timer service
* Life-cycle callback interceptors: Interceptor that interposes on the target instance life-cycle 
event callbacks (@PostConstruct and @PreDestroy).

Target Class Interceptors
===
There are several ways of defining interception. The simplest is to add interceptors (method-level, timeout, or 
life-cycle interceptors) to the bean itself as shown below. CustomerService annotates logMethod() with 
@AroundInvoke. logMethod() is used to log a message when a method is entered and exited. Once this Managed 
Bean is deployed, any client invocation to createCustomer() or findCustomerById() will be intercepted, and the 
logMethod() will be applied. Note that the scope of this interceptor is limited to the bean itself (the target class).

Method pattern:
```
@AroundInvoke
Object <METHOD>(InvocationContext ic) throws Exception;
```

The following rules apply to an around-invoke method (as well constructor, timeout, or life-cycle interceptors):
* The method can have public, private, protected, or package-level access but must not be 
static or final.
* The method must have a javax.interceptor.InvocationContext parameter and must return 
Object, which is the result of the invoked target method.
* The method can throw a checked exception

Example:
```
@Transactional
public class CustomerService {
 
 @Inject
 private EntityManager em;
 @Inject
 private Logger logger;
 
 public void createCustomer(Customer customer) {
 em.persist(customer);
 }
 
 public Customer findCustomerById(Long id) {
 return em.find(Customer.class, id);
 }
 
 @AroundInvoke
 private Object logMethod(InvocationContext ic) throws Exception {
 logger.entering(ic.getTarget().toString(), ic.getMethod().getName());
 try {
 return ic.proceed();
 } finally {
 logger.exiting(ic.getTarget().toString(), ic.getMethod().getName());
 }
 }
}
```
Class Interceptors
====

Define Interceptor in a separate class:
```
public class LoggingInterceptor {
 
  @Inject
  private Logger logger;
 
  @AroundConstruct
  private void init(InvocationContext ic) throws Exception {
    logger.fine("Entering constructor");
    try {
      ic.proceed();
    } finally {
      logger.fine("Exiting constructor");
    }
 }
}
```

and apply
```
@Transactional
public class CustomerService {
 
 @Inject
 private EntityManager em;
 
 @Interceptors(LoggingInterceptor.class)
 public void createCustomer(Customer customer) {
 em.persist(customer);
 }
 
 public Customer findCustomerById(Long id) {
 return em.find(Customer.class, id);
 }
}
```
or
```
@Transactional
@Interceptors(LoggingInterceptor.class)
public class CustomerService {
  
  public void createCustomer(Customer customer) {...}
  
  public Customer findCustomerById(Long id) {...}
 
  @ExcludeClassInterceptors
  public Customer updateCustomer(Customer customer) { ... }
} 
```