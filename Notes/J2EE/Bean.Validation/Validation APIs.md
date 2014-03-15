Validator
====
If you do it programmatically you need to start with the Validation class which bootstraps the Bean Validation provider. Its buildDefaultValidatorFactory method builds and returns a ValidatorFactory which in turn is used to build a Validator. The code looks like the following:
```
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
Validator validator = factory.getValidator();
///You then need to manage the life cycle of the ValidatorFactory and programmatically close it. 
factory.close();
```
If your application runs in a Java EE container, then the container must make the following instances available under JNDI:
* ValidatorFactory under java:comp/ValidatorFactory
* Validator under java:comp/Validator
Then you can look up these JNDI names and get an instance of Validator. But instead of looking the instances
up via JNDI, you can request them to be injected via the @Resource annotation.
If your container is using CDI (which by default it is in Java EE 7), the container must allow the injection via @Inject.

Validate bean:
```
CD cd = new CD("Kind of Blue", 12.5f); Set<ConstraintViolation<CD>> violations = validator.validate(cd); assertEquals(0, violations.size());
```
Validate Properties:
```
CD cd = new CD();
cd.setNumberOfCDs(2);
Set<ConstraintViolation<CD>> violations = validator.validateProperty(cd, "numberOfCDs"); assertEquals(0, violations.size());
```
Validate Values:
```
Set<ConstraintViolation<CD>> constr = validator.validateValue(CD.class, "numberOfCDs", 2); assertEquals(0, constr.size());
Set<ConstraintViolation<CD>> constr = validator.validateValue(CD.class, "numberOfCDs", 7); assertEquals(1, constr.size());
```
Validate Methods:
```
CD cd = new CD("Kind of Blue", 12.5f);
Method method = CD.class.getMethod("calculatePrice", Float.class);
ExecutableValidator methodValidator = validator. forExecutables(); Set<ConstraintViolation<CD>> violations = methodValidator.validateParameters(cd, method,
assertEquals(1, violations.size());
```
Validate Groups:
```
CD cd = new CD();
cd.setDescription("Best Jazz CD ever");
Set<ConstraintViolation<CD>> violations = validator.validate(cd, Default.class); assertEquals(2, violations.size());
```
