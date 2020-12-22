---
title: Try Understanding Lombok
author: MijazzChan
date: 2020-12-09 16:49:56 +0800
categories: [NotForBlog, Personal_Notes]
tags: [notes]
---

# Try Understanding Lombok

## What is Lombok


> [Project Lombok](https://projectlombok.org/)
>
> [MVN Repo](https://mvnrepository.com/artifact/org.projectlombok/lombok)

大概一年多以前我接触`Sping Boot`的设计模式时, 了解到`Entity`, `Service`, `Repository`, 等层次设计的时候, `POJO`什么的.

当时的项目用的是`Spring Data JPA`做的持久层, 中间经过几层的数据, 对象传递. 同时也为了方便调试,  产生了很多无用的`field.getter()`, `field.setter()`, `Object.toString()`, `Object.Constructer`函数或字段.

`Lombok`简单来说就是使用注释引入或者说注入所需的生成的字节码.

但是如果知道`Annotation`的运行原理的话, 也比较难理解其实现方式. 因为`Annotation`说白了也只是一个接口. 其下几个相关的

+ `Target` - 规定修饰的类型 
+ `Retention` - 规定策略的类型, 见`RetentionPolicy`

也不直接具有修改注入的能力.

## Go Deep?

查阅了一些资料后

> [Java Annotation - runoob](https://www.runoob.com/w3cnote/java-annotation.html)
>
> [Lombok Github Repo](https://github.com/rzwitserloot/lombok)
>
> [Compilation-Overview - OpenJDK](https://openjdk.java.net/groups/compiler/doc/compilation-overview/index.html)
>
> **JSR 269 Pluggable Annotation Processing API**

所以说到`Annotation`就是一个接口的话, 那么回想被注释的类其实是被`Implement`了.  并且被这些注解注释的类, 在`javac`对其进行编译过程时, 会拉起他们对应的注解 解释?/执行? 器`Annotation Processor`. 在这些注解执行类中, 通过重写`@OverWrite`几个关键的执行方法从而达到编译注入的目的. 

```java
/**
 * An abstract annotation processor designed to be a convenient
 * superclass for most concrete annotation processors.  This class
 * examines annotation values to compute the {@linkplain
 * #getSupportedOptions options}, {@linkplain
 * #getSupportedAnnotationTypes annotation types}, and {@linkplain
 * #getSupportedSourceVersion source version} supported by its
 * subtypes.
 *
 * <p>The getter methods may {@linkplain Messager#printMessage issue
 * warnings} about noteworthy conditions using the facilities available
 * after the processor has been {@linkplain #isInitialized
 * initialized}.
 *
 * <p>Subclasses are free to override the implementation and
 * specification of any of the methods in this class as long as the
 * general {@link javax.annotation.processing.Processor Processor}
 * contract for that method is obeyed.
 *
 * @author Joseph D. Darcy
 * @author Scott Seligman
 * @author Peter von der Ah&eacute;
 * @since 1.6
 */
public abstract class AbstractProcessor implements Processor
```

在Lombok的源码中, 可见其几个Processor的类都是继承自上述这个在`package javax.annotation.processing;`中的类的.

> [core/lombok/javac/handlers/HandleData.java](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/javac/handlers/HandleData.java)
>
> [core/lombok/javac/handlers/HandleSetter.java](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/javac/handlers/HandleSetter.java)
>
> ...

并且采用了一些常用的**反射**和修改AST树来实现功能的. 

这里不细说反射原理, 同时`javac`处理`RetentionPolicy`的策略属`Java Compilation`范畴.

所以回到代码举个例子

> 本次工程是Maven同步的, 所以`javac -cp`引入`lombok`的时候要找到本地maven repo的位置并定位`lombok-xxx.jar`, 如果你没修改过默认应该是`~/.m2`下的

`Contact.java`

```java
@Data
public class Contact {
    private Long contactId;
    private String familyName;
    private String givenName;
    private String mobileNum;
    private Date birthDay;
}
```

`@Data` 部分注释, 即是`@Data`在被处理时同时引入下列几个

```java
 * @see Getter
 * @see Setter
 * @see RequiredArgsConstructor
 * @see ToString
 * @see EqualsAndHashCode
 * @see lombok.Value
```

并且观察Lombok的源码也可以发现其拉起了几个Handler

[core/lombok/javac/handlers/HandleData.java](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/javac/handlers/HandleData.java)

```java
handleConstructor.generateRequiredArgsConstructor(typeNode, AccessLevel.PUBLIC, staticConstructorName, SkipIfConstructorExists.YES, annotationNode);
		handleConstructor.generateExtraNoArgsConstructor(typeNode, annotationNode);
		handleGetter.generateGetterForType(typeNode, annotationNode, AccessLevel.PUBLIC, true, List.<JCAnnotation>nil());
		handleSetter.generateSetterForType(typeNode, annotationNode, AccessLevel.PUBLIC, true, List.<JCAnnotation>nil(), List.<JCAnnotation>nil());
		handleEqualsAndHashCode.generateEqualsAndHashCodeForType(typeNode, annotationNode);
		handleToString.generateToStringForType(typeNode, annotationNode);
```

这里只去看`HandleSetter` - [HandleSetter - Lombok, 不贴出](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/javac/handlers/HandleSetter.java)

通过

```shell
MijazzChan@R720 MINGW64 /e/JWorkSpace/JSourceCodeLearn/src/main/java/edu/zstu/mijazz/lomboklearn/entity (master)
$ javac -cp /c/Dev/Env/m2Repo/org/projectlombok/lombok/1.18.16/lombok-1.18.16.jar ./Contact.java

MijazzChan@R720 MINGW64 /e/JWorkSpace/JSourceCodeLearn/src/main/java/edu/zstu/mijazz/lomboklearn/entity (master)
$ ls
Contact.class  Contact.java

MijazzChan@R720 MINGW64 /e/JWorkSpace/JSourceCodeLearn/src/main/java/edu/zstu/mijazz/lomboklearn/entity (master)
$ javap ./Contact.class
Compiled from "Contact.java"
public class edu.zstu.mijazz.lomboklearn.entity.Contact {
  public edu.zstu.mijazz.lomboklearn.entity.Contact();    / # 不是Lombok的 
  public java.lang.Long getContactId();
  public java.lang.String getFamilyName();
  public java.lang.String getGivenName();
  public java.lang.String getMobileNum();
  public java.util.Date getBirthDay();
  public void setContactId(java.lang.Long);
  public void setFamilyName(java.lang.String);
  public void setGivenName(java.lang.String);
  public void setMobileNum(java.lang.String);
  public void setBirthDay(java.util.Date);
  public boolean equals(java.lang.Object);
  protected boolean canEqual(java.lang.Object);
  public int hashCode();
  public java.lang.String toString();
}

```

可以看到`lombok`注入的方法是可以通过反编译类看到的, 如果你想看到更详细的类内步骤, 可以用`IntelliJ IDEA`打开反编译.

> 这里额外提一下, `@Data`不会注入无参Constructor, 这里之所以有无参构造器, 是因为JDK自动会根据其超类-Object自动创建, 这个在Java文档[Constructor](https://docs.oracle.com/javase/tutorial/java/javaOO/constructors.html)中有提到.
>
> 如果需要自定义一个属于自己的类的无参构造器, 你需要`@NoArgsConstructor`.

```
You don't have to provide any constructors for your class, but you must be careful when doing this. The compiler automatically provides a no-argument, default constructor for any class without constructors. This default constructor will call the no-argument constructor of the superclass. In this situation, the compiler will complain if the superclass doesn't have a no-argument constructor so you must verify that it does. If your class has no explicit superclass, then it has an implicit superclass of Object, which does have a no-argument constructor
```

## Raising Another Question

既然解决了`javac`的问题, 那另一个问题, javac是编译时是有AST的, 也就是`Abstract Syntax Tree`.

反编译出来的文件中存在Lombok加入的方法, 那么在编译时, 这些方法就会有对应的AST节点. 

如果能够把注入前的AST通过静态代码抽象出来, 若静态代码的AST树不包含这些新加入方法的AST节点, 那么就可以判定Lombok的确是在编译时通过修改并补全AST来实现字节码更改的.

引入`javaparser`的Maven依赖

```xml
<dependency>
            <groupId>com.github.javaparser</groupId>
            <artifactId>javaparser-core</artifactId>
            <version>3.18.0</version>
</dependency>
```

 写一个简单的工具类

```java
public class JavacCompileUtil {
    @SneakyThrows
    public static void main(String[] args) {
        CompilationUnit compilationUnit =
                StaticJavaParser.parse(new File("$$CHANGE HERE TO CLASS FILE PATH"));
        YamlPrinter yamlPrinter = new YamlPrinter(true);
        System.out.println(yamlPrinter.output(compilationUnit));
    }
}
```

```yaml
---
root(Type=CompilationUnit): 
    packageDeclaration(Type=PackageDeclaration): 
        name(Type=Name): 
            identifier: "entity"
            qualifier(Type=Name): 
                identifier: "lomboklearn"
                qualifier(Type=Name): 
                    identifier: "mijazz"
                    qualifier(Type=Name): 
                        identifier: "zstu"
                        qualifier(Type=Name): 
                            identifier: "edu"
    imports: 
        - import(Type=ImportDeclaration): 
            isAsterisk: "false"
            isStatic: "false"
            name(Type=Name): 
                identifier: "Data"
                qualifier(Type=Name): 
                    identifier: "lombok"
        - import(Type=ImportDeclaration): 
            isAsterisk: "false"
            isStatic: "false"
            name(Type=Name): 
                identifier: "NoArgsConstructor"
                qualifier(Type=Name): 
                    identifier: "lombok"
        - import(Type=ImportDeclaration): 
            isAsterisk: "false"
            isStatic: "false"
            name(Type=Name): 
                identifier: "Date"
                qualifier(Type=Name): 
                    identifier: "util"
                    qualifier(Type=Name): 
                        identifier: "java"
    types: 
        - type(Type=ClassOrInterfaceDeclaration): 
            isInterface: "false"
            name(Type=SimpleName): 
                identifier: "Contact"
            comment(Type=JavadocComment): 
                content: "\r\n * @Time 2020-12-09 3:54 PM\r\n * @Author MijazzChan\r\n * Lombok Learn package, ENTITY class, class{Contact} as a person\r\n "
            members: 
                - member(Type=FieldDeclaration): 
                    modifiers: 
                        - modifier(Type=Modifier): 
                            keyword: "PRIVATE"
                    variables: 
                        - variable(Type=VariableDeclarator): 
                            name(Type=SimpleName): 
                                identifier: "contactId"
                            type(Type=ClassOrInterfaceType): 
                                name(Type=SimpleName): 
                                    identifier: "Long"
                - member(Type=FieldDeclaration): 
                    modifiers: 
                        - modifier(Type=Modifier): 
                            keyword: "PRIVATE"
                    variables: 
                        - variable(Type=VariableDeclarator): 
                            name(Type=SimpleName): 
                                identifier: "familyName"
                            type(Type=ClassOrInterfaceType): 
                                name(Type=SimpleName): 
                                    identifier: "String"
                - member(Type=FieldDeclaration): 
                    modifiers: 
                        - modifier(Type=Modifier): 
                            keyword: "PRIVATE"
                    variables: 
                        - variable(Type=VariableDeclarator): 
                            name(Type=SimpleName): 
                                identifier: "givenName"
                            type(Type=ClassOrInterfaceType): 
                                name(Type=SimpleName): 
                                    identifier: "String"
                - member(Type=FieldDeclaration): 
                    modifiers: 
                        - modifier(Type=Modifier): 
                            keyword: "PRIVATE"
                    variables: 
                        - variable(Type=VariableDeclarator): 
                            name(Type=SimpleName): 
                                identifier: "mobileNum"
                            type(Type=ClassOrInterfaceType): 
                                name(Type=SimpleName): 
                                    identifier: "String"
                - member(Type=FieldDeclaration): 
                    modifiers: 
                        - modifier(Type=Modifier): 
                            keyword: "PRIVATE"
                    variables: 
                        - variable(Type=VariableDeclarator): 
                            name(Type=SimpleName): 
                                identifier: "birthDay"
                            type(Type=ClassOrInterfaceType): 
                                name(Type=SimpleName): 
                                    identifier: "Date"
            modifiers: 
                - modifier(Type=Modifier): 
                    keyword: "PUBLIC"
            annotations: 
                - annotation(Type=MarkerAnnotationExpr): 
                    name(Type=Name): 
                        identifier: "Data"
                - annotation(Type=MarkerAnnotationExpr): 
                    name(Type=Name): 
                        identifier: "NoArgsConstructor"
...
```

可以很清楚的看到, 在针对`Contact.java`的AST树中, 并不存在有关Lombok注入方法的节点.

## Back to Documentation

通过了解AST在`javac`里的作用后, 几乎可以确定Lombok是编译时通过修改AST树并补全相应节点, 来实现方法的注入的.

即 `Source ClassFile` -> `Parse` -> `AST` -> `Handle Annotation` -> `Call/Find Annotation Handler` -> `Lombok Annotation Processor - handle/modify AST` -> `Analyze/Fill AST node` -> `New/Modified AST ` -> `Byte Code`

![Java Compile Process](https://openjdk.java.net/groups/compiler/doc/compilation-overview/javac-flow.png)

上述编译过程中, Lombok对应的即是`Annotation Processing`这一步. 详细可以参考`JSR-269`

同时也在[src/utils/lombok/javac/JavacTreeMaker.java](https://github.com/rzwitserloot/lombok/blob/master/src/utils/lombok/javac/JavacTreeMaker.java)找到了相应对AST进行操作的代码.

**至于为什么在Lombok下搜索`Processor`会出现多个Annotation Processor相关的类呢, 官网也给出了解释.** 

+ `src/core/lombok/core/AnnotationProcessor.java`
+ `src/core/lombok/javac/apt/Processor.java`
+ `src/core/lombok/javac/apt/LombokProcessor.java`
+ `src/launch/lombok/launch/AnnotationProcessor.java`

`lombok.launch.AnnotationProcessorHider$AnnotationProcessor`作为入口, 被javac在执行`Annotation Processing`这一步拉起. 它将被实例化并且执行`init()`. 它会开始寻找lombok的Jar File, 注: 这里的Jar包并不是`.jar`结尾的, 而是`.SCL.lombok`, 并且通过ClassLoader开始加载lombok的core包. 

`lombok.core.AnnotationProcessor`会是接下来core中先执行的类, 它也是一个入口类. 它根据运行环境是否是javac或者是eclipse ecj, 来选择对应的`Annotation Processor`来进一步处理.

最终`lombok.javac.apt.LombokProcessor`才是操作并处理注入的`Annotation Processor`. 

**同时你在Jar File处看不到Lombok的源码也有其原因的**

书写代码时, IDE会根据Jar包中索引到的类对你进行代码提示, 但是由于Lombok工程的特殊性, 你只需要在编译时需要其Jar包的依赖.

对于未编译层面的Java语句来说, 如果包在这个层级可见, 会在代码提示中或索引里增加很多你可能不需要的类.

所以Lombok类在`Java-the-language`是不可见的, 但在`Java-the-JVM`是可见的.

同时上面也说到`lombok.launch`作为入口处, 其寻找`.SCL.lombok`结尾的包, 使用ClassLoader运行时才加载, 这种反常规甚至奇妙的方式(官方用的convoluted trick)也可以避免其被索引所带来的麻烦.



> https://projectlombok.org/contributing/lombok-execution-path
>
> With `javac` (and netbeans, maven, gradle, and most other build systems), lombok runs as an annotation processor.
>
> Lombok is on the classpath, and `javac` will load every `META-INF/services/javax.annotation.processing.Processor` file on the classpath it can find, reading each line and loading that class, then executing it as an annotation processor. `lombok.jar` has this file, it lists `lombok.launch.AnnotationProcessorHider$AnnotationProcessor` as entry.
>
> This class is not actually visible (it is public, but its outer class (`AnnotationProcessorHider`) is package private, making it invisible to java-the-language), however, it is considered visible for the purposes of java-the-VM and therefore it will run. This convoluted trick is used to ensure that anybody who develops with lombok on the classpath doesn't get lombok's classes or lombok's dependencies injected into their 'namespace' (for example, if you add lombok to your project, your IDE will *not* start suggesting lombok classes for auto-complete dialogs).
>
> The `lombok.launch.AnnotationProcessorHider$AnnotationProcessor` class is loaded by `javac`, instantiated, and `init()` is called on it. This class starts lombok's `ShadowClassLoader`; it finds the jar file it is in, then will start loading classes from this jar file. It looks not for files ending in `.class` like normal loaders, it looks for files ending in `.SCL.lombok` instead (this too is for the purpose of hiding lombok's classes from IDEs and such). Via this classloader, the *real* annotation processor is launched, which is class `lombok.core.AnnotationProcessor`.
>
> The `lombok.core.AnnotationProcessor` is also a delegating processor. It can delegate to one of 2 sub-processors based on the environment lombok finds itself in: If it's javac, class `lombok.javac.apt.LombokProcessor` is used (and if the plexus compiler framework is used, which can be the case when compiling with javac, some extra code runs to patch lombok into its modular classloading architecture). If it's ecj (eclipse's compiler, which means we're either running inside eclipse itself, or being invoked as annotation processor for ecj, the standalone eclipse compiler), errors/warnings are injected into the compilation process to tell the user they should use different parameters to [use lombok in eclipse/ecj](https://projectlombok.org/setup/ecj).
>
> `lombok.javac.apt.LombokProcessor` is the 'real' annotation processor that does the work of transforming your code.



