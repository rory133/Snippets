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
