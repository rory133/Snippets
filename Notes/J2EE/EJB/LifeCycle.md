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
