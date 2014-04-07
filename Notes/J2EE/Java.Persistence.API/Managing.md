There are two types of environment:
* application managed. Develeper has to manage EM by hand.
* container managed. Servlet-, EJB-, Web-container is responsible for managing EM lifecycle. 

EntityManager Interface Methods to Manipulate Entities:
* void persist(Object entity) - Makes an instance managed and persistent
* <T> T find(Class<T> entityClass, Object primaryKey) - Searches for an entity of the specified class and primary key
* <T> T getReference(Class<T> entityClass, Object primaryKey) - Gets an instance, whose state may be lazily fetched
* void remove(Object entity) - Removes the entity instance from the persistence context and from the underlying database
* <T> T merge(T entity) - Merges the state of the given entity into the current persistence context
* void refresh(Object entity) - Refreshes the state of the instance from the database, overwriting changes made to the entity. A typical case is when you use the EntityManager.refresh() method to undo changes that have been made to the entity in memory only. 
* void flush() - Synchronizes the persistence context to the underlying database
* void clear() - Clears the persistence context, causing all managed entities to become detached
* void detach(Object entity) - Removes the given entity from the persistence context, causing a managed entity to become detached
* boolean contains(Object entity) - Checks whether the instance is a managed entity instance belonging to the current persistence context

Entity operations.
====
* If the data are flushed to the database at one point, and if later in the code the application calls the rollback() method, the flushed data will be taken out of the database.
