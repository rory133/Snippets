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

Queries
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
