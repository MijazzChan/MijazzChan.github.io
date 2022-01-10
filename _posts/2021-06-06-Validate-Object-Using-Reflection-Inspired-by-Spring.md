---
title: Validate Object Using Reflection Inspired by Spring
author: MijazzChan
date: 2021-06-06 15:15:27 +0800
categories: [Java, Design]
tags: [java, reflection, annotation]
---

Inspired by Spring’s excellent dependency injection way of handling bean validation, after reading some official java documentations and Spring Docs, I decided to find a way of achieving the purpose of validating object, but this time, **without spring**.

Since no Spring will be involved this time, this post will mainly focusing on Java’s own `Reflection` API. And also I will re-use some great pattern inside Spring. Haven’t seen my last post? Check out [HERE](/posts/From-Spring-Boot-Bean-Validation-to-Python-Wrapper/).

> `Effective Java - Third Edition`
>
> `Java Documentation - Reflection`
>
> `Spring Documentation - 5.3.7`

## Intro

Remember how I design my own annotation `@HexColorValue` in my last post? 

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

It is the `@Constraint` annotation that annotated above this exact annotation that gives it magic. It allows Spring to locate the class of our custom validator and inject it on the fly.

Check out out last validator class itself.

```java
public class HexColorValueValidator implements ConstraintValidator<HexColorValue, String> {

    private static final Pattern hexColorValuePattern = Pattern.compile("^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$");

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        return hexColorValuePattern.matcher(value).matches();
    }

}
```

It implements `interface ConstraintValidator<A extends Annotation, T>` to **equip** this interface with our custom validation capability. This technique significantly improves the code extendibility horizontally. (Which means you can add a bunch of custom validator without polluting you code with shit ton of utility classes.)  **For how to turn this into a spring service, I have illustrated it in the last post, again, check out [here](/posts/From-Spring-Boot-Bean-Validation-to-Python-Wrapper/).**

这样写可以使代码在**水平方向上的延展能力**极大增强. 只需要在需要的地方打上注解, 需要校验的地方用自动注入加入一个验证器, 一行解决且无侵入. 接下来就是在无Spring的情况, 仅用Reflection API来复现逻辑, 摆脱框架.

## Custom Annotation

Start by creating out own Annotation. I will use `@Ipv4Address` as a starter this time.

### `@Ipv4Address`

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@ValidateBy(clazz = Ipv4AddressValidatorImpl.class)
public @interface Ipv4Address {
}
```

This annotation requires nothing passing in when it is being annotated. Although pay attention to

+ `@Target`. It means this annotation is annotated above `FIELD` (字段). 
+ `RetentionPolicy.RUNTIME` means this annotation will still be remained in RUNTIME.

+ `@ValidateBy` is also a custom annotation. Inspired by Spring’s `@Constraint(validateBy=?)`

### `@ValidateBy`

When we design our validation logic inside some utils class, it is essential to know what exact Validator Class or Method we will be using and invoking. This piece of information must be grabbed from the constraint annotation. And inspired by Spring. the best place for storing this information is the annotation’s annotation. Because you will never want to add Validator Class everytime you use that annotation.

Here is the simple example of illustrating the difference.

+ No `@ValidateBy` on `@Ipv4Adress`

```java
// If 
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Ipv4Address{
    Class<? extend Validator> validateBy();
}

// You will need to assign validateBy everytime you use this annotation.
public class QO{
    @Ipv4Adrress(validateBy=Ipv4AddressValidator.class)
    String firstIp;
    @Ipv4Adrress(validateBy=Ipv4AddressValidator.class)
    String secondIp;
    @Ipv4Adrress(validateBy=Ipv4AddressValidator.class)
    String thirdIp;
}
// You don't want that. 
```

+ `@ValidateBy` on `@Ipv4Adress`

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@ValidateBy(clazz = Ipv4AddressValidatorImpl.class)
public @interface Ipv4Address {
}

// You NEED NOT TO assign validateBy everytime you use this annotation.
public class QO{
    @Ipv4Adrress
    String firstIp;
    @Ipv4Adrress
    String secondIp;
    @Ipv4Adrress
    String thirdIp;
}
```

**This piece of information should be stored as annotation inside the annotation itself. So, let’s see**  

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE) // it means this annotation should be annotated above annotation.
public @interface ValidateBy {
    Class<? extends Validator<?>> clazz();
}
```

## Validator Implement

This interface itself only has one method, which is `isValid(T value)`

`Validator.java`

```java
public interface Validator<T> {
    boolean isValid(T value);
}
```

We will implement the capability of validation here. Staring with `Ipv4AdressValidatorImpl`

```java
public class Ipv4AddressValidatorImpl implements Validator<String> {

    private static final Pattern v4Pattern = Pattern.compile("^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?).){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$");

    public Ipv4AddressValidatorImpl() {
        // Reflection's getDeclaredConstructor()
    }

    // Implementation is here.
    public boolean isValid(String value) {
        return isValidIpv4Address(value);
    }

    private static boolean isValidIpv4Address(String value) {
        return v4Pattern.matcher(value).matches();
    }
}
```

Now that we have given the capability to interface `Validator`. 

## Service-ify the Validation

Spring uses `ValidationFactory` to provide validator. For me, a `ValidationProvider` is more than enough. `Singleton Pattern` is introduced here.

### `ValidationProvider` - framework

```java
public class ValidationProvider {
    private static final ValidationProvider INSTANCE = new ValidationProvider();
	// NOTE: SINGLETON PATTERN
    private ValidationProvider() {
        // Non-instantiable outside this scope because of this private constructor.
    }

    public static ValidationProvider getInstance() {
        return INSTANCE;
    }

    public <T> Collection<ViolationException> validate(T object) {
        // validation will start here.
    }
```

### `ValidationProvider` - logic

Before I paste up all the code from `validate(T object)`, couple insights of how the validation service are as follows.

+ `ValidationProvider.getInstance()` will allow access from any scope of the project, and ready for validating by calling `validationProvider.validate(object)`.
+ Validation Logic can be simply described in the following steps.
  1. object is passed into `validate(object)`
  2. Fields that declared inside the object will all be collected using `getDeclaredFields()` provided by `java.lang.reflect`.
  3. If the field is still not accessible after `field.trySetAccessible()`, a `ViolationException` will be added to the `violationList` with corresponding reason.
  4. For each field, its annotations will be gathered using `getDeclaredAnnotations()`. If the annotation is related to validation, it will try to dig out the `@ValidateBy` by following up the annotation “Inheritance link”.
  5. If `@ValidateBy` can be found, which means the corresponding annotation is a Constraint-Related annotation. Because annotation stores specific Class by using `@ValidateBy(clazz=XXX)`, `XXX` will be instantiate using `getDeclaredConstructor().newInstance()`.
  6. `"isValid"` method will be invoked by reflection API to gather the validation result.
+ You may wonder why `XXX` will always have the same `“isValid”` method that can be invoked via reflection, that’s exactly the reason why we need `Validator` as a interface and then implement the `isValid` method afterwards.

> For Reflection API usage, [Here](https://docs.oracle.com/javase/tutorial/reflect/) is the documentation you can look into.
>
> `ViolationException`  will be included in the end of this post.

```java
    public <T> Collection<ViolationException> validate(T object) {
        Field[] fields;
        Annotation[] annotations;

        List<ViolationException> violationList = new LinkedList<>();

        fields = object.getClass().getDeclaredFields();
        // For each field.
        for (Field field : fields) {
            if (!field.trySetAccessible()) {
                violationList.add(
                        new ViolationException.Builder()
                                .object(object)
                                .field(field)
                                .reason("Fails to access object's field.")
                );
                continue;
            }

            annotations = field.getDeclaredAnnotations();
            // For each field's annotation
            for (Annotation annotation : annotations) {
                try {
                    // Annotation's annotation. Locate the @ValidateBy annotation declared
                    // on the corresponding annotation. This will lead us to the
                    // Validator Class we assigned.
                    ValidateBy validateBy = annotation.annotationType().getDeclaredAnnotation(ValidateBy.class);
                    if (validateBy == null) {
                        // No Validator Class is assigned.
                        continue;
                    }
                    // Retrieve the validator class assigned from the corresponding annotation by @ValidateBy(clazz=?)
                    Class<? extends Validator<?>> assignedValidator = validateBy.clazz();
                    // Prepare the Method for validation of the exact validator class provided at @ValidateBy
                    Method validationMethod = assignedValidator.getMethod("isValid", field.getType());
                    validationMethod.setAccessible(true);
                    // Construct a instance of the exact Validator using Reflection for method invoke usage.
                    // e.g: Ipv4AdressValidatorImpl::new
                    Validator validator = assignedValidator.getDeclaredConstructor().newInstance();
                    // Invoke method and get the validation result
                    boolean result = (boolean) validationMethod.invoke(validator, field.get(object));
                    // Validation return false
                    if (!result) {
                        violationList.add(
                                new ViolationException.Builder()
                                        .object(object)
                                        .field(field)
                                        .constraintAnnotation(annotation.annotationType())
                                        .build()
                        );
                    }
                } catch (InvocationTargetException | IllegalAccessException | NoSuchMethodException | InstantiationException e) {
                    violationList.add(
                            new ViolationException.Builder()
                                    .object(object).field(field).reason(e.getLocalizedMessage())
                    );
                }
            }
        }

        return violationList;
    }
```

## Simple Test-Drive

> Horizontal extendibility will be described in next chapter.

Let’s write up a simple test case.

```java
public class MyQueryObject {
    @Ipv4Address
    private String ipRangeStart;
    
    @Ipv4Address
    private String ipRangeStop;
    
    @Ipv4Address
    private String proxyPoolAddress;
	// getter and setter
}
```

```java
MyQueryObject validObj = new MyQueryObject();
validObj.setIpRangeStart("10.0.0.1");
validObj.setIpRangeStop("10.7.255.254");
validObj.setProxyPoolAddress("172.17.0.1");

MyQueryObject invalidObj = new MyQueryObject();
invalidObj.setIpRangeStart("255.256.258.999");
invalidObj.setIpRangeStop("172.18.1.1");
invalidObj.setProxyPoolAddress("where 1=1");

ValidationProvider validationProvider = ValidationProvider.getInstance();
        
// Violation Count 
System.out.println(validationProvider.validate(validObj).size());
// Violation Output
System.err.println(validationProvider.validate(invalidObj));
```

```java
0
[===> Violation > xyz.mijazz.springfreevalidation.objects.MyQueryObject.ipRangeStart | @Ipv4Address
, ===> Violation > xyz.mijazz.springfreevalidation.objects.MyQueryObject.proxyPoolAddress | @Ipv4Address
]
```

Smoothly.

## Horizontal Extendibility

> Horizontal Extendibility in the post, specifically in plain words, means you can create your custom validation annotations as many as you want. For instance, I want to create `@Ipv6Adress`, `@FutureDatetime`, `@PastDatetime`, and then annotated them all inside one object.

`ValidationProvider`  is designed to be applicable to various situation. In this case, we only need to create our annotation class annotated by `@ValidateBy(clazz=XXX)`, then a `XXXValidationImpl` implements `Validator`. and **boom** you are good to go. No need to alter code elsewhere.

### `@Ipv6Adress` - Step 1

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@ValidateBy(clazz = Ipv6AddressValidatorImpl.class)
public @interface Ipv6Address {
}
```

### `Ipv6AddressValidatorImpl` - Step 2

```java
public class Ipv6AddressValidatorImpl implements Validator<String> {
    
    private static final Pattern v6Pattern = Pattern.compile("^(([0-9a-fA-F]{1,4}:){7,7}[0-9a-fA-F]{1,4}|([0-9a-fA-F]{1,4}:){1,7}:|([0-9a-fA-F]{1,4}:){1,6}:[0-9a-fA-F]{1,4}|([0-9a-fA-F]{1,4}:){1,5}(:[0-9a-fA-F]{1,4}){1,2}|([0-9a-fA-F]{1,4}:){1,4}(:[0-9a-fA-F]{1,4}){1,3}|([0-9a-fA-F]{1,4}:){1,3}(:[0-9a-fA-F]{1,4}){1,4}|([0-9a-fA-F]{1,4}:){1,2}(:[0-9a-fA-F]{1,4}){1,5}|[0-9a-fA-F]{1,4}:((:[0-9a-fA-F]{1,4}){1,6})|:((:[0-9a-fA-F]{1,4}){1,7}|:)|fe80:(:[0-9a-fA-F]{0,4}){0,4}%[0-9a-zA-Z]{1,}|::(ffff(:0{1,4}){0,1}:){0,1}((25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])\\.){3,3}(25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])|([0-9a-fA-F]{1,4}:){1,4}:((25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9])\\.){3,3}(25[0-5]|(2[0-4]|1{0,1}[0-9]){0,1}[0-9]))$");

    public Ipv6AddressValidatorImpl() {
        // Reflection's getDeclaredConstructor()
    }
	// Implement Validator's method.
    public boolean isValid(String value) {
        return isValidIpv6Address(value);
    }

    private static boolean isValidIpv6Address(String value) {
        return v6Pattern.matcher(value).matches();
    }
}
```

### `MyV6QueryObject` - Step 3

```java
// Mixed with Ipv4, Ipv6.
public class MyV6QueryObject {
    @Ipv6Address
    private String ipRangeStart;
    @Ipv6Address
    private String ipRangeStop;
    @Ipv4Address
    private String proxyPoolIp;
    // getters and setters
}
```

Let’s write up a test case shall we.

```java
MyV6QueryObject myV6QueryObject = new MyV6QueryObject();
myV6QueryObject.setIpRangeStart("fd0a:e481:6bf9:d049:0000:0000:0000:0000");
myV6QueryObject.setIpRangeStop("fd0a:e481:6bf9:d049:?!*#:ff=ff:!*@3:ffff");
myV6QueryObject.setProxyPoolIp("172.17.0.1");
```

use that same `ValidationProvider`

```java
ValidationProvider validationProvider = ValidationProvider.getInstance();
System.out.println(validationProvider.validate(myV6QueryObject));
```

```java
[===> Violation > xyz.mijazz.springfreevalidation.objects.MyV6QueryObject.ipRangeStop | @Ipv6Address
]
```

So the main idea of this chapter is that, you don’t have to bother yourself altering code inside that `ValidationProvider`. For horizontal development in the future, you just have to make your own annotation and implement corresponding `Validator`, and boom, you are good to go.

### Extended Result

I went ahead then finish the logic of  `@FutureDatetime`, `@PastDatetime`. The result came back fine.

```java
public class MixedQueryObject {
    @Ipv4Address
    String validIpv4;
    @Ipv4Address
    String invalidIpv4;
    @Ipv6Address
    String validIpv6;
    @Ipv6Address
    String inValidIpv6;
    @FutureDatetime
    LocalDateTime validFutureDt;
    @FutureDatetime
    LocalDateTime invalidFutureDt;
    @PastDatetime
    LocalDateTime validPastDt;
    @PastDatetime
    LocalDateTime invalidPastDt;
}
```

```java
[===> Violation > xyz.mijazz.springfreevalidation.objects.MixedQueryObject.invalidIpv4 | @Ipv4Address
, ===> Violation > xyz.mijazz.springfreevalidation.objects.MixedQueryObject.inValidIpv6 | @Ipv6Address
, ===> Violation > xyz.mijazz.springfreevalidation.objects.MixedQueryObject.invalidFutureDt | @FutureDatetime
, ===> Violation > xyz.mijazz.springfreevalidation.objects.MixedQueryObject.invalidPastDt | @PastDatetime
]
```

## Conclusion

At the time I wrote up this post, I was thinking that maybe adding a `nullable` feature will be helpful. After all, the whole idea of this post is to offer a way/possibility to validate object without using any Spring or Hibernate feature. 

Although those frameworks do great or even excellent jobs at creating the things we need or use in our dev career, it’s still worth having a look inside to figure out how that actually works and manage to achieve it without using them. At the end of the day, I PREFER NOT use framework only to do the FARMWORK. 

MASSIVE shout out to `Effective Java - Third Edition` by Joshua Bloch. 

 