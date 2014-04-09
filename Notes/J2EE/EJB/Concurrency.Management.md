
* @ConcurrencyManagement annotation for sigleton session bean. It allows to configure concurrence control for bean and has two diffirent ways:
1.Container-managed concurrency (CMC): The container controls concurrent access to the bean
instance based on metadata (annotation or the XML equivalent). Used if no concurrency management configured.
2.Bean-managed concurrency (BMC): The container allows full concurrent access and defers the
synchronization responsibility to the bean.

### Container-Managed Concurrency
You can then use the @Lock annotation to specify how the container must manage concurrency when
a client invokes a method. The annotation can take the value READ (shared) or WRITE (exclusive). @Lock annotation can be specified on the class, the methods, or both. Specifying on the class means that it
applies to all methods. If you do not specify the concurrency locking attribute, it is assumed to be @Lock(WRITE) by
default.
* @Lock(LockType.WRITE): A method associated with an exclusive lock will not allow
concurrent invocations until the method’s processing is completed. For example, if a client C1
invokes a method with an exclusive lock, client C2 will not be able to invoke the method until
C1 has finished.
* @Lock(LockType.READ): A method associated with a shared lock will allow any number of
other concurrent invocations to the bean’s instance. For example, two clients, C1 and C2, can
access simultaneously a method with a shared lock.

#### Example
```
@Singleton
@Lock(LockType.WRITE)
@AccessTimeout(value = 20, unit = TimeUnit.SECONDS)
public class CacheEJB {
  private Map<Long, Object> cache = new HashMap<>();
  
  public void addToCache(Long id, Object object) {
    if (!cache.containsKey(id))
    cache.put(id, object);
  }

  public void removeFromCache(Long id) {
    if (cache.containsKey(id))
    cache.remove(id);
  }
  
  @Lock(LockType.READ)
  public Object getFromCache(Long id) {
    if (cache.containsKey(id))
      return cache.get(id);
    else
      return null;
  }
}
```
