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
