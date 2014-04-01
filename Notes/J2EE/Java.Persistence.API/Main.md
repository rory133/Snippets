Main
===
* In previous versions of Java ee, the persistent component model was called entity Bean, or to be more precise, entity Bean CMp (Container-Managed persistence), and was associated with enterprise JavaBeans. this model of persistence lasted from J2ee 1.3 until 1.4, but was heavyweight and finally replaced by Jpa since Java ee 5. Jpa uses the term “entity” instead of “entity Bean.”
* How does JPA map objects to a database? Through metadata.
* XML deployment descriptors are an alternative to using annotations. However, although each annotation has
an equivalent XML tag and vice versa, there is a difference in that the XML overrides annotations. If you annotate an
attribute or an entity with a certain value, and at the same time you deploy an XML descriptor with a different value,
the XML will take precedence.
* The question is, when should you use annotations over XML and why? First of all, it’s a matter of taste, as the
behavior of both is exactly the same. When the metadata are really coupled to the code (e.g., a primary key), it does
make sense to use annotations, since the metadata are just another aspect of the program. Other kinds of metadata,
such as the column length or other schema details, can be changed depending on the deployment environment
(e.g., the database schema is different in the development, test, or production environment). A similar situation
arises when a JPA-based product needs to support several different database vendors. Certain id generation, column
options, and so on may need to be adjusted depending on the database type in use. This may be better expressed in
external XML deployment descriptors (one per environment) so the code doesn’t have to be modified.
