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
