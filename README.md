Qualifiers with Members
========

```
@Qualifier
@Retention(RUNTIME)
@Target({FIELD, TYPE, METHOD})
public @interface NumberOfDigits {
 
 Digits value();
 boolean odd();
}
 
public enum Digits {
 TWO,
 EIGHT,
 TEN,
 THIRTEEN
}
```

The manner in which you would use this qualifier with members doesn’t change from what you’ve seen so far. 
The injection point will qualify the needed implementation by setting the annotation members as follows:

```
@Inject @NumberOfDigits(value = Digits.THIRTEEN, odd = false)
private NumberGenerator numberGenerator;
```

And the concerned implementation will do the same.
```
@NumberOfDigits(value = Digits.THIRTEEN, odd = false)
public class IsbnEvenGenerator implements NumberGenerator {...} 
```


