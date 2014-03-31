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

