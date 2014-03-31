Mappings
====
Up to now, I have assumed that an entity gets mapped to a single table, also known as a primary table. But sometimes
when you have an existing data model, you need to spread the data across multiple tables, or secondary tables. To do
this, you need to use the annotation @SecondaryTable to associate a secondary table to an entity or @SecondaryTables
(with an “s”) for several secondary tables.
```
@Entity
@SecondaryTables({
@SecondaryTable(name = "city"),
@SecondaryTable(name = "country")
})
public class Address {
@Id
private Long id;
private String street1;
private String street2;
@Column(table = "city")
private String city;
@Column(table = "city")
private String state;
@Column(table = "city")
private String zipcode;
@Column(table = "country")
private String country;
// Constructors, getters, setters
}
```

Instead of using the default access type, you can explicitly
specify the type by means of the @javax.persistence.Access annotation.
This annotation takes two possible values, FIELD or PROPERTY, and can be used on the entity itself and/or on
each attribute or getter. For example, when an @Access(AccessType.FIELD) is applied to the entity, only mapping
annotations placed on the attributes will be taken into account by the persistence provider. It is then possible to
selectively designate individual getters for property access with @Access(AccessType.PROPERTY).

Composite primary key
===
Composite primary key can be implemented by two variants: @IdClass and @EmbeddedId

* @EmbeddedId
Key definition:
```
@Embeddable
public class NewsId {
private String title;
private String language;
// Constructors, getters, setters, equals, and hashcode
}
```
Apply composite key:
```
@Entity
public class News {
@EmbeddedId
private NewsId id;
private String content;
// Constructors, getters, setters
}
```
* @idClass

```
public class NewsId {
private String title;
private String language;
// Constructors, getters, setters, equals, and hashcode
}
```
Apply composite key:
```
@Entity
@IdClass(NewsId.class)
public class News {
@Id private String title;
@Id private String language;
private String content;
// Constructors, getters, setters
}
```
The @IdClass approach is more prone to error, as you need to define each primary key attribute in both the
@IdClass and the entity, taking care to use the same name and Java type. The advantage is that you don’t need to
change the code of the primary key class (no annotation is needed). For example, you could use a legacy class that,
for legal reasons, you are not allowed to change but that you can reuse.
One visible difference is in the way you reference the entity in JPQL. In the case of @IdClass, you would do
something like the following:
```
select n.title from News n // @IdClass
select n.newsId.title from News n // @EmbeddedId
```

