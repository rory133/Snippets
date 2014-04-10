### Startup Initialization
* @Startup garanties bean instatiation during EJB-container startup process.
```
@Singleton
@Startup
public class CacheEJB {
 // ...
}
```
* @javax.ejb.DependsOn allows to build startup order.
```
@Singleton
public class CountryCodeEJB {
  //...
}

@DependsOn("CountryCodeEJB")
@Singleton
public class CacheEJB {
  //...
}

@DependsOn("business.jar#CountryCodeEJB")
@Singleton
public class CacheEJB {
  // ...
}
```
#### Stateless and Singleton
1. The life cycle of a stateless or singleton session bean starts when a client requests a reference to the bean (using either dependency injection or JNDI lookup). In the case of a singleton, it can also start when the container is bootstrapped (using the @Startup annotation). The container creates a new session bean instance.
2. If the newly created instance uses dependency injection through annotations (@Inject, @Resource, @EJB, @PersistenceContext, etc.) or deployment descriptors, the container injects all the needed resources.
3. If the instance has a method annotated with @PostContruct, the container invokes it.
4. The bean instance processes the call invoked by the client and stays in ready mode to process future calls. Stateless beans stay in ready mode until the container frees some space in the pool. Singletons stay in ready mode until the container is shut down.
5. The container does not need the instance any more. It invokes the method annotated with @PreDestroy, if any, and ends the life of the bean instance.

#### Stateful bean life cycle
1. The life cycle of a stateful bean starts when a client requests a reference to the bean (either using dependency injection or JNDI lookup). The container creates a new session bean instance and stores it in memory.
2. If the newly created instance uses dependency injection through annotations (@Inject, @Resource, @EJB, @PersistenceContext, etc.) or deployment descriptors, the container injects all the needed resources.
3. If the instance has a method annotated with @PostContruct, the container invokes it.
4. The bean executes the requested call and stays in memory, waiting for subsequent client requests.
5. If the client remains idle for a period of time, the container invokes the method annotated
with @PrePassivate, if any, and passivates the bean instance into a permanent storage. Passivation process can be disabled by annotation @Stateful(passivationCapable=false).
6. If the client invokes a passivated bean, the container activates it back to memory and
invokes the method annotated with @PostActivate, if any.
7. If the client does not invoke a passivated bean instance for the session timeout period, the container destroys it.
8. Alternatively to step 7, if the client calls a method annotated by @Remove, the container then invokes the method annotated with @PreDestroy, if any, and ends the life of the bean instance.

#### Life-Cycle Callback Annotations
* @PostConstruct. Marks a method to be invoked immediately after you create a bean instance and the container
does dependency injection. This annotation is often used to perform any initialization.
* @PreDestroy. Marks a method to be invoked immediately before the container destroys the bean instance.The method annotated with @PreDestroy is often used to release resources that had been previously initialized. With stateful beans, this happens after timeout or when a method annotated with @Remove has been completed.
* @PrePassivate. Stateful beans only. Marks a method to be invoked before the container passivates the instance. It usually gives the bean the time to prepare for serialization and to release resources that cannot be serialized (e.g., it closes connections to a database, a message broker, a network socket, etc.).
* @PostActivate. Stateful beans only. Marks a method to be invoked immediately after the container reactivates the
instance. Gives the bean a chance to reinitialize resources that had been closed during passivation.

#### Rules apply to a callback method
* The method must not have any parameters and must return void.
* The method must not throw a checked exception but can throw runtime exceptions. Throwing a runtime exception will roll back the transaction if one exists.
* The method can have public, private, protected, or package-level access but must not be static or final.
* A method may be annotated with multiple annotations (the init() methodcan be annotated with @PostConstruct and @PostActivate). However, only one annotation of a given type may be present on a bean (you can’t have two @PostConstruct annotations in the same session bean, for example).
* A callback method can access the beans’ environment entries.
