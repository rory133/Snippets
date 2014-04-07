Cache
===
* First-level cache. Represented by Persistence Context. 
* Second-level cache. Hasn't been described in JPA specification.

@Cacha
Binding parameters.
===
There are two ways to pass parameters to query:
* Positional parameters are designated by the question mark (?) followed by an integer (e.g., ?1). When the query is executed, the parameter numbers that should be replaced need to be specified.
```
SELECT c
FROM Customer c
WHERE c.firstName = ?1 AND c.address.country = ?2
```
* Named parameters can also be used and are designated by a String identifier that is prefixed by the colon (:) symbol. When the query is executed, the parameter names that should be replaced need to be specified.
```
SELECT c
FROM Customer c
WHERE c.firstName = :fname AND c.address.country = :country
```

Queries.
===
JPA 2.1 has five different types of queries that can be used in code, each with a different purpose:
* Dynamic queries: This is the simplest form of query, consisting of nothing more than a JPQL query string dynamically specified at runtime.
* Named queries: Named queries are static and unchangeable. There is a restriction in that the name of the query is scoped to the persistence unit and must be unique within that scope, meaning that only one findAll method can exist. A findAll query for customers and a findAll query for addresses should be named differently. A common practice is to prefix the query name with the entity name. For example, the findAll query for the Customer entity would be named Customer.findAll.
* Criteria API: JPA 2.0 introduced the concept of object-oriented query API.
* Native queries: This type of query is useful to execute a native SQL statement instead of a JPQL statement.
* Stored procedure queries: JPA 2.1 brings a new API to call stored procedures.

JPA 2.1 uses two different locking mechanisms (JPA 1.0 only had support for optimistic locking).
* Optimistic locking is based on the assumption that most database transactions don’t conflict with other transactions, allowing concurrency to be as permissive as possible when allowing transactions to execute. The decision to acquire a lock on the entity is actually made at the end of the transaction.The use of a dedicated @Version annotation on an entity allows the EntityManager to perform optimistic locking simply by comparing the value of the version attribute in the entity instance with the value of the column in the database. Without an attribute annotated with @Version, the entity manager will not be able to do optimistic locking automatically (implicitly).
* Pessimistic locking is based on the opposite assumption, so a lock will be obtained on the resource before operating on it. Pessimistic locking is based on the opposite assumption to optimistic locking, because a lock is eagerly obtained on the entity before operating on it. This is very resource restrictive and results in significant performance degradation, as a database lock is held using a SELECT ... FOR UPDATE SQL statement to read data.

Queries Examples.
===
* Since JPA 2.0, an attribute can be retrieved depending on a condition (using a CASE WHEN ... THEN ... ELSE ... END expression). For example, instead of retrieving the price of a book, a statement can return a computation of the price (e.g., 50% discount) depending on the publisher (e.g., 50% discount on the Apress books, 20% discount for all the other books).
```
SELECT CASE b.editor WHEN 'Apress'
THEN b.price * 0.5
ELSE b.price * 0.8
END
FROM Book b
```
* A constructor may be used in the SELECT expression to return an instance of a Java class initialized with the result of the query. The class doesn’t have to be an entity, but the constructor must be fully qualified and match the attributes.
```
SELECT NEW org.agoncal.javaee7.CustomerDTO(c.firstName, c.lastName, c.address.street1)
FROM Customer c
```

* Executing these queries will return either a single value or a collection of zero or more entities (or attributes) including duplicates. To remove the duplicates, the DISTINCT operator must be used.
```
SELECT DISTINCT c FROM Customer c
```

* Subqueries are supported:
```
SELECT c
FROM Customer c
WHERE c.age = (SELECT MIN(cust. age) FROM Customer cust))
```

* Order by
```
SELECT c
FROM Customer c
WHERE c.age > 18
ORDER BY c.age DESC, c.address.country ASC
```

* Group By and Having
```
SELECT c.address.country, count(c) FROM Customer c
GROUP BY c.address.country
HAVING c.address.country <> 'UK'
```
* Bulk Delete
```
DELETE FROM Customer c WHERE c.age < 18
```
* Bulk Update
```
UPDATE Customer c SET c.firstName = 'TOO YOUNG' WHERE c.age < 18
```

Callbacks
===
Types:
* @PrePersist. Marks a method to be invoked before EntityManager.persist() is executed.
* @PostPersist. Marks a method to be invoked after the entity has been persisted. If the entity autogenerates its
primary key (with @GeneratedValue), the value is available in the method.
* @PreUpdate. Marks a method to be invoked before a database update operation is performed (calling the
entity setters or the EntityManager.merge() method).
* @PostUpdate. Marks a method to be invoked after a database update operation is performed.
* @PreRemove. Marks a method to be invoked before EntityManager.remove() is executed.
* @PostRemove. Marks a method to be invoked after the entity has been removed.
* @PostLoad. Marks a method to be invoked after an entity is loaded (with a JPQL query or an EntityManager.find()) or refreshed from the underlying database. There is no @PreLoad annotation, as it doesn’t make sense to preload data on an entity that is not built yet.
Method requirements:
  * Methods can have public, private, protected, or package-level access but must not be static or
final.
* A method may be annotated with multiple life-cycle event annotations. However, only one life-cycle annotation of a given type may be present in an entity class (e.g., you can’t have two @PrePersist annotations in the same entity).
* A method can throw unchecked (runtime) exceptions but not checked exceptions. Throwing a
runtime exception will roll back the transaction if one exists.
* A method can invoke JNDI, JDBC, JMS, and EJBs but cannot invoke any EntityManager or
Query operations.
* With inheritance, if a method is specified on the superclass, it will get invoked before the method on the child class. For example, if in Listing 6-38 Customer was inheriting from a Person entity, the Person @PrePersist method would be invoked before the Customer @PrePersist method.
* If event cascading is used in the relationships, the callback method will also be called in a cascaded way. For example, let’s say a Customer has a collection of addresses, and a cascade remove is set on the relation. When you delete the customer, the Address @PreRemove method would be invoked as well as the Customer @PreRemove method.


### Listeners.
* Requirements:
..* The first is that the class must have a public no-arg constructor.
..* The methods must have a parameter of a type that is compatible with the entity type, as the entity related to the event is being passed into the callback.
..* Only unchecked exceptions can be thrown. This causes the remaining listeners and callback methods to not be invoked and the transaction to be roll backed if one exists.

* Invokation order:
1. @EntityListeners for a given entity or superclass in the array order,
2. Entity listeners for the superclasses (highest first),
3. Entity listeners for the entity,
4. Callbacks of the superclasses (highest first), and
5. Callbacks of the entity.

#### Examples:
```
@EntityListeners({DataValidationListener.class, AgeCalculationListener.class})
@Entity
public class Customer {
  @Id @GeneratedValue
  private Long id;
  private String firstName;
  private String lastName;
  private String email;
  private String phoneNumber;
  @Temporal(TemporalType.DATE)
  private Date dateOfBirth;
  @Transient
  private Integer age;
  @Temporal(TemporalType.TIMESTAMP)
  private Date creationDate;
  // Constructors, getters, setters
}
```
When several listeners are defined and the life-cycle event occurs, the persistence provider iterates through each listener in the order in which they are listed and will invoke the callback method, passing a reference of the entity to which the event applies.
￼

#### Default Listener.
￼You can define default listener in persistance.xml:
```
<?xml version="1.0" encoding="UTF-8"?>
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence/orm http://xmlns.jcp.org/xml/ns/persistence/orm_2_1.xsd" version="2.1">
   <persistence-unit-metadata>
      <persistence-unit-defaults>
<entity-listeners>
<entity-listener class="org.agoncal.book.javaee7.chapter06.DebugListener"/>
         </entity-listeners>
      </persistence-unit-defaults>
   </persistence-unit-metadata>
</entity-mappings>
```
In this file, the <persistence-unit-metadata> tag defines all the metadata that don’t have any annotation equivalent. The <persistence-unit-defaults> tag defines all the defaults of the persistence unit, and the <entity-listener> tag defines the default listener. This file needs to be referred in the persistence.xml and deployed with the application. The DebugListener will then be automatically invoked for every single entity.
When you declare a list of default entity listeners, each listener gets called in the order in which it is listed in
the XML mapping file. Default entity listeners always get invoked before any of the entity listeners listed in the @EntityListeners annotation. If an entity doesn’t want to have the default entity listeners applied to it, it can use the @ExcludeDefaultListeners annotation.
Example:
```

@ExcludeDefaultListeners
@Entity
public class Customer {
  @Id @GeneratedValue
  private Long id;
  private String firstName;
  private String lastName;
  private String email;
  private String phoneNumber;
  @Temporal(TemporalType.DATE)
  private Date dateOfBirth;
  @Transient
  private Integer age;
  @Temporal(TemporalType.TIMESTAMP)
  private Date creationDate;
  // Constructors, getters, setters
}
```
