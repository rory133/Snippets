Interceptors
===

Interceptors are deployment-specific and are disabled by default. Like alternatives, interceptors have to be 
enabled by using the CDI deployment descriptor beans.xml of the jar or Java EE module as shown:

```
<beans xmlns="http://xmlns.jcp.org/xml/ns/javaee"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
 http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
 version="1.1" bean-discovery-mode="all">
 
 <interceptors>
 <class>org.agoncal.book.javaee7.chapter02.LoggingInterceptor</class>
 </interceptors>
</beans>
```

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

LifeCycle interceptor
===
Define interceptor:
```
public class ProfileInterceptor {
 
  @Inject
  private Logger logger;
 
  @PostConstruct
  public void logMethod(InvocationContext ic) throws Exception {
    logger.fine(ic.getTarget().toString());
    try {
      ic.proceed();
    } finally {
      logger.fine(ic.getTarget().toString());
    }
  }
 
  @AroundInvoke
  public Object profile(InvocationContext ic) throws Exception {
     long initTime = System.currentTimeMillis();
  try {
    return ic.proceed();
  } finally {
    long diffTime = System.currentTimeMillis() - initTime;
    logger.fine(ic.getMethod() + " took " + diffTime + " millis");
  }
 }
}
```

Apply:
```
@Transactional
@Interceptors(ProfileInterceptor.class)
public class CustomerService {
 
  @Inject
  private EntityManager em;
 
  @PostConstruct
  public void init() {
  // ...
  }
 
  public void createCustomer(Customer customer) {
   em.persist(customer);
  }
 
  public Customer findCustomerById(Long id) {
   return em.find(Customer.class, id);
  }
}
```
When the bean is instantiated by the 
container, the logMethod() will be invoked prior to the init() method. Then, if a client calls createCustomer() or 
findCustomerById(), the profile() method will be invoked.

Chaining and Excluding Interceptors
====
```
@Stateless
@Interceptors({I1.class, I2.class})
public class CustomerService {
  
  public void createCustomer(Customer customer) {...}
  
  @Interceptors({I3.class, I4.class})
  public Customer findCustomerById(Long id) {...}
  
  public void removeCustomer(Customer customer) {...}
 
  @ExcludeClassInterceptors
  public Customer updateCustomer(Customer customer) {...}
}
```
When a client calls the updateCustomer() method, no interceptor is invoked because the method is annotated 
with @ExcludeClassInterceptors. When the createCustomer() method is called, interceptor I1 is executed followed 
by interceptor I2. When the findCustomerById() method is invoked, interceptors I1, I2, I3, and I4 get executed in 
this order.


====
Create binding annotation:
```
@InterceptorBinding
@Target({METHOD, TYPE})
@Retention(RUNTIME)
public @interface Loggable { }
```
Define interceptor and annotate it:
```
@Interceptor
@Loggable
public class LoggingInterceptor {
 
 @Inject
 private Logger logger;
 
 @AroundInvoke
 public Object logMethod(InvocationContext ic) throws Exception {
  logger.entering(ic.getTarget().toString(), ic.getMethod().getName());
  try {
    return ic.proceed();
  } finally {
   logger.exiting(ic.getTarget().toString(), ic.getMethod().getName());
  }
 }
}
```
Apply
```
@Transactional
@Loggable
public class CustomerService {
 
 @Inject
 private EntityManager em;
 
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
public class CustomerService {
 @Loggable
 public void createCustomer(Customer customer) {...}
 public Customer findCustomerById(Long id) {...}
}
```
