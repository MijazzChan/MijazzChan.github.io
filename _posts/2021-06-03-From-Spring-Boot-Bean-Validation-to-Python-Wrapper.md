---
title: From Spring Boot Bean Validation to Python Wrapper
author: MijazzChan
date: 2021-06-03 12:59:54 +0800
categories: [Java, Design]
tags: [python, java, annotation, wrapper]
---

Keywords in this post are as follows.

+ Spring Boot Bean Validation
+ `JSR-303/349/380`: Bean Validation Related Topic JSR
+ Java Annotation Processing
+ Custom Annotation
+ Java `Reflection` API (`java.lang.reflect`)
+ Python Function Wrapper

Prerequisite.

+ Basic Spring Boot 2 knowledge(IoC)

## Bean Validation Scenario 

### Spring boot code example

If you have encountered with Java Bean Validation or Field Validation, then you may be familiar with the following pattern.

`pom.xml(Maven3)`

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

This is our Controller’s incoming query payload class. (Query Object)

```java
// Lombok enabled.
@Data
public class MyQueryObject{
    @Min(0)
    @Max(100)
    private int queryNumberInstance;

    @Pattern(regexp = "^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$")
    private String hexColorValue;

    // getter, setter, ... handled by Lombok.
}
```

Now the controller itself.

```java
@RestController
public class MyControllerDemo{
    @RequestMapping(value = "/some/path", method = RequestMethod.POST, produces = "application/json")
    public ResponseEntity<String> someQuery(@Valid @RequestBody MyQueryObject queryObj){
        // ...
        return ResponseEntity.ok("Here u go.");
    }
}
```

If the query object itself fails the validation check, a `MethodArgumentNotValidException` will be generated, and by default, `spring` will translate that exception into `HTTP STATUS - 400(BAD REQUEST)`.

### Spring boot test example

Provided by `spring-test` itself, `MockMvc` can be instanized  to perform mock POST request easily. Let’s write up a simple spring boot test, shall we?

```java
@SpringBootTest
@AutoConfigureMockMvc
@Slf4j
public class QueryObjectValidationTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    void passValidQueryObject_to_MyControllerDemo() throws Exception {
        MediaType mediaType = MediaType.APPLICATION_JSON;

        // This payload will not cause any exception.
        String validPayload = "{\"queryNumberInstance\": 69, \"hexColorValue\" : \"#282a36\"}";
        mockMvc.perform(MockMvcRequestBuilders.post("/some/path")
                .content(validPayload)
                .contentType(mediaType))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().contentType(mediaType))
                .andDo(mvcResult -> log.info(mvcResult.getResponse().getContentAsString()));

        // This payload will definitely trigger exception,
        // and spring will handle it with BAD_REQUEST.
        String invalidPayload = "{\"queryNumberInstance\": 96, \"hexColorValue\" : \"#Z10RYP\"}";
        mockMvc.perform(MockMvcRequestBuilders.post("/some/path")
                .content(invalidPayload)
                .contentType(mediaType))
                .andExpect(MockMvcResultMatchers.status().isBadRequest())
                .andDo(mvcResult -> log.info(String.valueOf(HttpStatus.valueOf(mvcResult.getResponse().getStatus()))));
    }
```

> Test provided above passed without exception.

```
2021-06-03 15:35:56.324  INFO 15821 --- [           main] i.m.t.QueryObjectValidationTest : Here u go.
2021-06-03 15:35:56.344  WARN 15821 --- [           main] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public org.springframework.http.ResponseEntity<java.lang.String> icu.mijazz.temporaryspringbootdemo.controller.Demo2Controller.someQuery(icu.mijazz.temporaryspringbootdemo.qo.MyQueryObject): [Field error in object 'myQueryObject' on field 'hexColorValue': rejected value [#Z10RYP]; codes [Pattern.myQueryObject.hexColorValue,Pattern.hexColorValue,Pattern.java.lang.String,Pattern]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [myQueryObject.hexColorValue,hexColorValue]; arguments []; default message [hexColorValue],[Ljavax.validation.constraints.Pattern$Flag;@7ac058a0,^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$]; default message [must match "^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$"]] ]
2021-06-03 15:35:56.347  INFO 15821 --- [           main] i.m.t.QueryObjectValidationTest  : 400 BAD_REQUEST
```

### Exception Handling(Additional)

> A digression into irrelevant details.

`@ExceptionHandler` annotation 

```java
@ExceptionHandler(value = MethodArgumentNotValidException.class)
public ResponseEntity<String> invalidQueryHandler() {
    // Do Whatever.
    return ResponseEntity.status(HttpStatus.I_AM_A_TEAPOT).body("Handled.");
}
```

### Other Scenario

You may also find it common for developers to have validation-annotation in Persistence Layer Entity. By default, `Spring Data` uses `Hibernate` underneath, which supports Bean Validation. However, Validations done in persistence layer act only as a anti data-corruption method. They can effectively stop invalid data from being written to DB, yet have little effect on the protection of parameter Injection based attack. 

## Custom Validator

**You may wonder why I wrote a complete example just to illustrate how the spring boot validator’ s approach on parameter validation. **

**Because besides that, spring boot also offers a approach to customize and realize your data validator just by minor implementation.**

### Construct a Custom Annotation

Remember how we validate the `MyQueryObject.hexColorValue` above? We use a `@Pattern` with a `regex` . What if we can create our own annotation `@HexColorValue` to validate every occurrence of Hex Color Value inside our project. It will improve our code readability significantly.

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = HexColorValueValidator.class)
public @interface HexColorValue {
    
    String message() default "{HexColorValue.inValid}";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
    
}
```

  Note that `@Constraint` is provided by `javax.validation.Constraint`. **Unlike the common scenario of creating custom annotation from stretch, this annotation with corresponding validator class will save us some trouble creating our own Annotation Processor using  `Java Reflection API`. **

**We only need to use `@Constraint` to explicitly point out which class is our validation handler, spring will automatically instantiate a instace. Validator discussed in this scope should have an implementation of  `interface ConstraintValidator<A extends Annotation, T>`, in this case, which is `ConstraintValidator<HexColorValue, String>`.**

> `message()` => `ValidationMessages.properties`
>
> The `ValidationMessages` resource bundle and the locale variants of this resource bundle contain strings that override the default validation messages. The `ValidationMessages` resource bundle is typically a properties file, `ValidationMessages.properties`, in the default package of an application.

  ### Implement ConstraintValidator

After creating our own `@HexColorValue` and pointing validator class using `@Constraint`. We will implement `HexColorValueValidator` as follows.

```java
public class HexColorValueValidator implements ConstraintValidator<HexColorValue, String> {

    private static final Pattern hexColorValuePattern = Pattern.compile("^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$");

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        return hexColorValuePattern.matcher(value).matches();
    }

}
```

### Substitute `@Pattern` with `@HexColorValue`

```java
@Data
public class MyQueryObject{
    // ......
    // No longer need @Pattern
    // @Pattern(regexp = "^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$") 
    
    @HexColorValue
    private String hexColorValue;

}
```

Everything will work smoothly as it used to be.

### Invoke Validation Manually

Since we have already created a custom validator class for HexColorValue, what if we need to re-use that validator code to validate String on the fly. 

You may think of using `HexColorValueValidator.isValid(String hexValue) -> boolean`. However, Raw usage of this method is not recommended. ~~(I just cannot find the way to explicitly invoke that exact method without providing a self ConstraintValidatorContext context...)~~. **Spring got us covered by using it perfect dependency injection technique.** 

#### Bean Tweak

> implements `Serializable`(Optional).

```java
public class MyQueryObject implements Serializable
```

#### Service-ify the manual validator with Generic Class Support

> `Validation`, `ValidatorFactory`, `Validator` ...
>
> `validator.validate()` will automatically locate the field that needs to be validated.(Annotated with registered with @Constraint), then  invoke the corresponding method inside the registered validation handler class to proceed.

```java
@Service
public class ValidationService<A extends Serializable> {

    private final ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();

    private final Validator validator = validatorFactory.getValidator();

    public boolean isValid(A objectPendingValidate) {
        Set<ConstraintViolation<A>> violations = validator.validate(objectPendingValidate);
        return violations.isEmpty();
    }
}
```

with the validation class being `@Service`-ify, now we can use `@Autowired` to inject `ValidationService` project-wisely.

#### Test-Drive our `ValidationService`

```java
    @Autowired
    ValidationService<MyQueryObject> qoValidationService;

    @Test
    void customValidationService_on_MyQueryObject() {
        MyQueryObject validQueryObject = new MyQueryObject();
        validQueryObject.setQueryNumberInstance(69);
        validQueryObject.setHexColorValue("#2B2C2D");

        MyQueryObject invalidQueryObject = new MyQueryObject();
        invalidQueryObject.setQueryNumberInstance(96);
        invalidQueryObject.setHexColorValue("#ZSDZXF");

        assertTrue(qoValidationService.isValid(validQueryObject));
        assertFalse(qoValidationService.isValid(invalidQueryObject));
        // Test passed.
    }
```

#### Strict Validation

Here’s an interesting point. When we call the constructor of `MyQueryObject` to initialize an instance, instead of letting spring handle the bean via dependency injection, the validation strategy will not kick in.

Look at `invalidQueryObject.setHexColorValue("#ZSDZXF");`. We just throw a invalid value to a setter. What if we are in a situation when all the bean existence should be **“strictly valid”**? 

Well, this perhaps goes a little bit off the track of this post original purpose. You can always make some validation implementations in the `setter` or `getter`.

## Python Wrapper Approach

### Introduction

If you are not familiar with python wrapper pattern or how does it work in python, it will not matter. If you ever coded in python, it wouldn’t be hard for you to understanding the following pattern. Remember how `%timeit` works in Jupyter Notebook?

```python
import time

def timeit(function_instance):
    def wrapper(*args, **kargs):
        time_start = time.time()
        result = function_instance(*args, **kargs)
        time_stop = time.time()
        print('{} second(s) taken up to execute function {}' \
            .format(time_stop - time_start, function_instance.__name__))
        return result
    return wrapper

@timeit
def yell_out_shit(slogan: str) -> None:
    time.sleep(1.5)
    print(slogan)

if __name__ == "__main__":
    yell_out_shit("PHP is the best language!")
```

```
❯ python /home/mijazz/Dev/pyworkspace/temp.py
PHP is the best language!
1.5018470287322998 second(s) taken up to execute function yell_out_shit
```

**But how come this ever get related in Java Bean Validation?**

### Scene Re-appearance

```python
def math_func_need_positiveNumber(input: int):
    if input <= 0:
        raise AssertionError('Invalid input in {} with {}'\
            .format(math_func_need_positiveNumber.__name__, input))
    # bla, bla, bla
    pass

def math_func_need_negativeNumber(input: float):
    if input > 0:
        raise AssertionError('Invalid input in {} with {}'\
            .format(math_func_need_negativeNumber.__name__, input))
    pass

def func_need_num_in_range(input: float):
    if input < 69 or input > 96:
        raise AssertionError('Invalid input in {} with {}'\
            .format(func_need_num_in_range.__name__, input))
    pass
```

For coder just need python for some simple math calculation, it is not uncommon to see this type of code...However, to state my point, I am not saying this code style is bad or something, a quick and dirty fix can simply be realized by python wrapper.

###  Implement our own `@Range()` annotation

> In our `MyQueryObject.queryNumberInstance`, I use `@Min(1)`, `@Max(100)` annotations to specify a range. But just so you know, `org.hibernate.validator.constraints.Range` is also out of the box. `@Range(min=1, max=100)` will be like the exact same.

Let’s start with the `Range` method first.

a little heads up.

+ `*annotion_args` is positional args given in decorator. They are **decorator/wrapper** params.
+ `**annotation_kargs` is keyword args given in decorator. They are **decorator/wrapper** params.
+ `*func_args` is the positional args given into the function itself. They are **function** params.
+ `**func_kargs` is the keyword args given into the function itself. They are **function** params.

```python
def Range(*annotation_args, **annotation_kargs):
    def wrapper(func):
        def on_invoke(*func_args, **func_kargs):
            # Keyword args processing
            for (arg_key, (low, high)) in annotation_kargs.items():
                if low is not None and func_kargs[arg_key] < low:
                    raise AssertionError('{} - Limit exceeded.'.format(func_kargs[arg_position]))
                if high is not None and func_kargs[arg_key] >= high:
                    raise AssertionError('{} + Bound exceeded.'.format(func_kargs[arg_position]))

            # Positional args processing
            for (arg_position, low, high) in annotation_args:
                if low is not None and func_args[arg_position] < low:
                    raise AssertionError('{} - Limit exceeded.'.format(func_args[arg_position]))
                if high is not None and func_args[arg_position] >= high:
                    raise AssertionError('{} + Bound exceeded.'.format(func_args[arg_position]))
            return func(*func_args, **func_kargs)
        return on_invoke
    return wrapper
```

### Usage

Let’s see how this puppy works. Derive a function of common usage.

```python
@Range((0, None, 0), (1, 0, 1), positive=(1, None))
def some_func(negative, ZERO, **kwargs):
    pass
```

+ `(0, None, 0)` and `(1, 0, 1)` are `*annotion_args`, which represents that `function_arg[0] <=> negative` should have `None` as low limit, `0` as upper bound; `Function_args[1] <=> ZERO` should have `0` as low limit, `1` as upper bound. 

In general.

+ `(0, None, 0)` => `0th` positional arg should be in `[-inf, 0)`.
+ `(1, 0, 1)` => `1st` positional arg should be in `[0, 1)`.
+ `positive=(1, None)` => keyword arg `positive` should be in `[1, +inf)`.

```python
some_func(-9, 0, positive=9) # Smooth.
some_func(-9, 0, 9)          # positive not given.
some_func(10, 0, positive=9) # 10 + Upper bound exceeded.
```

### Thoughts

You can even use the same technique to do a argument Type Check, instead of using a regular one. (Back in python2 style ??? or pythonic???). 

```python
def ForceType(**annotation_kargs):
    def wrapper(func):
        def on_invoke(**function_kargs):
            for (argname, type) in annotation_kargs.items():
                if not isinstance(function_kargs[argname], type):
                    raise TypeError()
            return func(**function_kargs)
        return on_invoke
    return wrapper
```

```python
@ForceType(Integer=int, String=str)
def another_func(*, Integer, String):
    pass

another_func(Integer=1, String="2") # Smooth
another_func(Integer=1, String=2)   # TypeError
```

Once you figure out the underlying pattern of the example provided above, it’s easy to implement other feature like `str` length check.

