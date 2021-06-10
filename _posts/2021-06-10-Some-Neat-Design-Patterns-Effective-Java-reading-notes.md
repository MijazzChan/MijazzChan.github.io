---
title: Some Neat Design Patterns - Effective Java reading notes
author: MijazzChan
date: 2021-06-10 00:22:19 +0800
categories: [Personal_Notes, Effective_Java]
tags: [java, notes]
---

+ Builder Pattern

  - Simple Builder Pattern without considering Hierarchy

  - Generic Type Builder Pattern (suits to class hierarchies)

+ Singleton property

+ Non-instantiability while dealing with Utility Class

> These patterns are pretty easy/simple, but it really helps me a lot especially when managing to understanding Java’s underlying design pattern through reading Java source code. Following these patterns also helps producing code which is developer-friendly. 
>
> Code with Simplicity, Unifinity and Tidiness.

## Builder Pattern

> Just seen a post on V2EX not long before. It’s about which pattern you will choose when developing, via properties methods or getters/setters.
>
> properties methods => `obj.name()` to retrieve `name`, `obj.name(String n)` to set `name`.
>
> getters/setters => `obj.getName()` retrieve `name`, `obj.setName()` to set `name`.

Since Java is discussed in this scope, you won’t feel unfamiliar with getters/setters pattern. A typical Java Bean pattern is: 

```java
BeanObject beanObject = new BeanObject(); // No arg Constructor
beanObject.setFirstProperty(arg1);
beanObject.setSecondProperty(arg2);
beanObject.setThirdProperty(arg3);
```

Normally it will not be a problem, until you have a  Java Bean consisting of a whole bunch of attributes/properties. Wherever this Bean is instantiated, you will have a ton of setter code piled up there.

That is the perfect scenario where Builder Pattern comes in handy. **It targets objects with a significant amount of properties and consisting of required properties.** Following this pattern will make code tidy to some degree.

### Simple Builder Pattern without considering Hierarchy

```java
public class ComputerComponents {
    // final properties <=> required properties.
    private final String cpu;
    private final String motherBoard;
    private final String ramStick;
    // private final String psu;
    // Optional properties
    private String hardDrive;
    private String solidStateDrive;
    private String graphicsCard;
    
    // make it private to dis-allow instantiability outside this class. 
    private ComputerComponents(Builder builder) {
        this.cpu = builder.cpu;
        this.motherBoard = builder.motherBoard;
        this.ramStick = builder.ramStick;
        
        this.hardDrive = builder.hardDrive;
        this.solidStateDrive = builder.solidStateDrive;
        this.graphicsCard = builder.graphicsCard;
    }
    
    // Builder Starts
    // make it public to allow outer access.
    public static class Builder {
        // required properties.
        private final String cpu;
        private final String motherBoard;
        private final String ramStick;
        // Optional properties
        private String hardDrive;
        private String solidStateDrive;
        private String graphicsCard;
        // Ensure Builder comes with required properties
        public Builder(String cpu, String motherBoard, String ramStick) {
            this.cpu = builder.cpu;
            this.motherBoard = builder.motherBoard;
            this.ramStick = builder.ramStick;
        }
        // Optional properties
        public Builder hardDrive(String hardDrive) {
            this.hardDrive = hardDrive;
        }
        public Builder solidStateDrive(String solidStateDrive) {
            this.solidStateDrive = solidStateDrive;
        }
        public Builder graphicsCard(String graphicsCard) {
            this.graphicsCard = graphicsCard;
        }
        // call the private constructor inside this scope.
        // provide builder as init param.
        public ComputerComponents build() { 
            return new ComputerComponents(this);
        }
    }
}
```

+ Class Constructor should be **private**. Avoid being invoked and instantiate outside this Class.
+ Builder class Constructor should be public.

The pattern may seem more complicated than the conventional getters/setter pattern.(especially Lombok meh...), though it makes somebody else who uses your code life a bit easier.

```java
new ComputerComponents
    .Builder("Ryzen_5900X", "Asus Prime X570-Pro", "SAMSUNG 16Gx2")
    .solidStateDrive("SAMSUMG 980 PRO")
    .build() 
```

**Neat.** Easy to write, easy to read. You can also implement validity check in Builder. Throws  `IllegalArgumentException` if necessary. 

### Builder Pattern suits to Hierarchies

#### Top-level as abstract class

```java
public abstract class Cellphone {
    // properties that are common in sub-class.
    public enum Feature {
        HEADPHONE_JACK, QUICK_CHARGE,
        DEPTH_CAMERA, OLED_SCREEN
    }
    final Set<Feature> featureSet;
    
    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Feature> featureSet = EnumSet.noneOf(Feature.class);
        // self() method => simulated self-type idiom.
        // Subclass must override.
        protected abstract T self();
        
        public T addFeature(Feature feature) {
            featureSet.add(Objects.requireNonNull(feature));
            return self();
        }
        
        abstract Cellphone build();
    }
    
    Cellphone(Builder<?> builder) {
        // Deep clone.
        this.featureSet = builder.featureSet.clone();
    }
}
```

+ Note that at `Row 8`, `Builder` uses a recursive type parameter, with the `T self()`, it will allow method in subclass work flawlessly without the type problem.
+ Remember to override `T self()` in subclass’s `Builder`.

#### Sub-classes

```java
public class IPhone6S extends Cellphone {
    // properties that are identical to sub-class
    public enum Size {NORMAL, PLUS}
    private final Size size;
    
    public static class Builder extends Cellphone.Builder<Builder> {
        private final Size size;
        
        // Also, builder starts with required properties.
        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }
        
        @Override
        protected Builder self() { return this; }
        
        @Override
        public IPhone6S build() {
            return new IPhone6S(this);
        }
    }
    // Also uses private constructor to limit access.
    private IPhone6S(Builder builder) {
        super(builder);
        this.size = builder.size;
    }
}
```

```java
public class IPhone12 extends Cellphone {
    // properties that are identical to sub-class
    public enum Size { MINI, NORMAL, PRO, PROMAX}
    private final Size size;
    
    public static class Builder extends Cellphone.Builder<Builder> {
        private final Size size;
        
        // Also, builder starts with required properties.
        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }
        
        @Override
        protected Builder self() { return this; }
        
        @Override
        public IPhone12 build() {
            return new IPhone12(this);
        }
    }
    // Also uses private constructor to limit access.
    private IPhone12(Builder builder) {
        super(builder);
        this.size = builder.size;
    }
}
```

The client code will be like:

```java
IPhone6S iPhone6S = new IPhone6S.Builder(IPhone6S.Size.PLUS)
        .addFeature(HEADPHONE_JACK).build();

IPhone12 iPhone12 = new IPhone12.Builder(IPhone12.Size.PROMAX)
        .addFeature(QUICK_CHARGE).addFeature(DEPTH_CAMERA)
        .addFeature(OLED_SCREEN).build();
```

## Enforce Singleton

A Singleton is a class that is instantiated exactly and only once. Usually enforcing a class as singleton is typically because that class is a state-less object or intrinsically unique.

### Singleton with public field and private constructor

```java
public class SystemKernel {
    public static final SystemKernel INSTANCE = new SystemKernel();
    // private constructor
    private SystemKernel() {
        // init method 
    }
    public XX retrieveStuff(){}
}

// Calling it like
SystemKernel.INSTANCE.retrieveStuff();
```

 ### Singleton with private field, private constructor, but public `getInstance()`

> This is probably the most common way to do so???

```java
public class SystemKernel {
    private static final SystemKernel INSTANCE = new SystemKernel();
    // private constructor
    private SystemKernel() {
        // init method 
    }
    public SystemKernel getInstance() { return INSTANCE; }
    public XX retrieveStuff(){}
}

// Calling it like
SystemKernel.getInstance().retrieveStuff();
```

Using either approach with `Serializable`, keep in mind that - **to maintain singleton guarantee, declare all instance fields `transient` and provide a `readResolve()` method.** Otherwise De-Serialization will break that guarantee.

```java
private Object readResolve() {
    // Effective Java Third Edition - Page 360.
    return INSTANCE;
}
```

 ### Enum Approach

> You may encounter problems when Class has to extend a super-class.

```java
public enum SystemKernel {
    INSTANCE;
    
    public XX retrieveStuff(){}
}
```

## Non-instantiability while dealing with Utility Class

> Singleton =>  instantiated exactly and only once.

When dealing with utility class, especially one filled with static methods inside the class, you normally wouldn’t expect this class to be instantiated.   But keep in mind that: 

+ A default constructor is generated if a class contains no explicit constructor.

  You may avoid adding constructor inside a utility class to prevent it from being instantiated. DO NO SUCH THING.

+ Making a class `abstract` does not work. 

  It does prevent a class from being instantiated by making it `abstract`. But its subclasses can be **INSTANTIATED**. Doing so will make user think you may want this class to have hierarchy usage.

The solution is quite simple this time, just **explicitly** declare a **`private` constructor** inside your class that filled with static methods. 

```java
public class MyStringUtils() {
    // explictly declare a private constructor
    private MyStringUtils() {
        
    }
    // filled with static methods.
    public static XX XXX() {}
    public static XX XXXX() {}
    //...
}
```



Both `default constructor` and `abstract for hierarchy usage` can be solved at the same time while keeping it developer-friendly.

