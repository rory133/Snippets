Main
=====
The Bean Validation specification recommends as good practice to consider null as valid.



Constraint Annotation
=====

Annotation
```
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = NotNullValidator.class)
public @interface NotNull {
 
 String message() default "{javax.validation.constraints.NotNull.message}"; // mandatory
 
 Class<?>[] groups() default {}; // mandatory
 
 Class<? extends Payload>[] payload() default {};
}
```
Custom validator.
```
public class NotNullValidator implements ConstraintValidator<NotNull, Object> {
 
 public void initialize(NotNull parameters) {
 }
 
 public boolean isValid(Object object, ConstraintValidatorContext context) {
 return object != null;
 }
}
```

The ConstraintValidator interface defines two methods that need to be implemented by the concrete classes.
* initialize: This method is called by the Bean Validation provider prior to any use of the constraint. This is where you usually initialize the constraint parameters if any.
* isValid: This is where the validation algorithm is implemented. This method is evaluated by the Bean Validation provider each time a given value is validated. It returns false if the value is not valid, true otherwise. The ConstraintValidatorContext object carries information and operations available in the context the constraint is validated to (as you’ll see later).
* 

Constraints can be applied to a class, field, method, constructor:

```
public class CardValidator {
  
  @NotNull
  private Person person;
  
  private ValidationAlgorithm validationAlgorithm;
public CardValidator(@NotNull ValidationAlgorithm validationAlgorithm) { this.validationAlgorithm = validationAlgorithm;
}
@AssertTrue
public Boolean validate(@NotNull CreditCard creditCard) {
    return validationAlgorithm.validate(creditCard.getNumber(), creditCard.getCtrlNumber());
  }
@AssertTrue
public Boolean validate(@NotNull String number, @Future Date expiryDate,
                          Integer controlNumber, String type) {
    return validationAlgorithm.validate(number, controlNumber);
  }
}
```
All constraints in inheritence path will be validated.
```
public class Item {
@NotNull
  protected Long id;
  @NotNull @Size(min = 4, max = 50)
  protected String title;
  protected Float price;
  protected String description;
@NotNull
  public Float calculateVAT() {
    return price * 0.196f;
}
@NotNull
public Float calculatePrice(@DecimalMin("1.2") Float rate) {
    return price * rate;
  }
}
```
```
public class CD extends Item {
  @Pattern(regexp = "[A-Z][a-z]{1,}")
  private String musicCompany;
  @Max(value = 5)
private Integer numberOfCDs; private Float totalDuration; @MusicGenre
private String genre;
// ConstraintDeclarationException : not allowed when method overriding public Float calculatePrice(@DecimalMin("1.4") Float rate) {
    return price * rate;
  }
}
```
Messages
====
The value of the default message can be hard coded, but it is recommended to use a resource bundle key to allow internationalization. By convention the resource bundle key should be the fully qualified class name of the constraint annotation concatenated to .message.
```
// Hard coded error message
String message() default "Email address doesn't look good";
// Resource bundle key
String message() default "{org.agoncal.book.javaee7.Email.message}";
```
By default the resource bundle file is named ValidationMessages.properties and must be in the class path of the application.
Thanks to message interpolation (javax.validation.MessageInterpolator interface), the error message can contain placeholders.
```
javax.validation.constraints.Size.message = size must be between {min} and {max}
```

ConstraintValidator Context
====
The ConstraintValidatorContext interface allows redefinition of the default constraint message. The buildConstraintViolationWithTemplate method returns a ConstraintViolationBuilder, based on the fluent API pattern, to allow building custom violation reports. The code that follows adds a new constraint violation to the report:
```
context.buildConstraintViolationWithTemplate("Invalid protocol")
       .addConstraintViolation();
```

Groups
===
Groups are binded to phase of validation proccess. It means that you can use one or more groups during certain validation procces.
```
@ChronologicalDates(groups = Delivery.class)
public class Order {
@NotNull
private Long id;
@NotNull @Past
private Date creationDate; private Double totalAmount;
@NotNull(groups = Payment.class) @Past(groups = Payment.class) private Date paymentDate;
@NotNull(groups = Delivery.class) @Past(groups = Delivery.class) private Date deliveryDate;
  private List<OrderLine> orderLines;
  // Constructors, getters, setters
}
```

Deployment Descriptors
====
If can't use annotation for appling constraints you use deployment descriptor for Bean Validation (validation.xml) for it. The first one, validation.xml, can be used by applications to refine some of the Bean Validation behavior (such as the default Bean Validation provider, the message interpolator, or specific properties).
```
<?xml version="1.0" encoding="UTF-8"?> <validation-config
xmlns="http://jboss.org/xml/ns/javax/validation/configuration" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://jboss.org/xml/ns/javax/validation/configuration 
                            validation-configuration-1.1.xsd"
version="1.1">
   <constraint-mapping>META-INF/constraints.xml</constraint-mapping>
</validation-config>
```
Defining Constraints on a Bean:
```
<?xml version="1.0" encoding="UTF-8"?>
<constraint-mappings
xmlns="http://jboss.org/xml/ns/javax/validation/mapping" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://jboss.org/xml/ns/javax/validation/mapping 
                            validation-mapping-1.1.xsd"
        version="1.1">
<bean class="org.agoncal.book.javaee7.chapter03.Book" ignore-annotations="false"> <field name="title">
<constraint annotation="javax.validation.constraints.NotNull"> <message>Title should not be null</message>
</constraint> </field>
<field name="price">
<constraint annotation="javax.validation.constraints.NotNull"/> <constraint annotation="javax.validation.constraints.Min">
<element name="value">2</element> </constraint>
</field>
<field name="description">
<constraint annotation="javax.validation.constraints.Size"> <element name="max">2000</element>
      </constraint>
    </field>
  </bean>
</constraint-mappings>
```
