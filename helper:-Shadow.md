Shadow is my take on Java reflection. It's an annotation based system which provides an alternative way to "reflect" into a class and access it's members.

The system was inspired by the [`Shadow`](http://jenkins.liteloader.com/job/Mixin/javadoc/org/spongepowered/asm/mixin/Shadow.html) feature in the SpongePowered [Mixin](https://github.com/SpongePowered/Mixin) library.


### Example usage

Take the following class as an example.

```java
public class Person {
    private final String firstName;
    private final String secondName;

    public Person(String firstName, String secondName) {
        this.firstName = firstName;
        this.secondName = secondName;
    }

    public String getFirstName() {
        return this.firstName;
    }

    public String getSecondName() {
        return this.secondName;
    }
    
    private void printInfo() {
        System.out.println("Hi name is " + getFirstName() + " " + getSecondName());
    }
}
```

#### Using standard java reflection

The Java reflection API allows us to call the `#printInfo` method and modify the value of the `firstName` and `secondName` fields, even though they're private (and final).

However, the process to do this is bulky, especially if we want to cache the lookups.

```java
private static final Field FIRST_NAME_FIELD;
private static final Field SECOND_NAME_FIELD;
private static final Method PRINT_INFO_METHOD;

static {
    try {
        FIRST_NAME_FIELD = Person.class.getDeclaredField("firstName");
        FIRST_NAME_FIELD.setAccessible(true);

        SECOND_NAME_FIELD = Person.class.getDeclaredField("secondName");
        SECOND_NAME_FIELD.setAccessible(true);
        
        PRINT_INFO_METHOD = Person.class.getDeclaredMethod("printInfo");
        PRINT_INFO_METHOD.setAccessible(true);
    } catch (NoSuchFieldException | NoSuchMethodException e) {
        throw new ExceptionInInitializerError(e);
    }
}

public static void setFirstName(Person person, String firstName) {
    try {
        FIRST_NAME_FIELD.set(person, firstName);
    } catch (IllegalAccessException e) {
        throw new RuntimeException(e);
    }
}

public static void setSecondName(Person person, String secondName) {
    try {
        SECOND_NAME_FIELD.set(person, secondName);
    } catch (IllegalAccessException e) {
        throw new RuntimeException(e);
    }
}

public static void printInfo(Person person) {
    try {
        PRINT_INFO_METHOD.invoke(person);
    } catch (IllegalAccessException | InvocationTargetException e) {
        throw new RuntimeException(e);
    }
}
```

#### Using shadow

The first step is to define the shadow definition. You do this by creating a "shadow" interface.

```java
@ShadowClass(className = "me.lucko.test.Person")
public interface ShadowPerson extends Shadow {
    
    @ShadowField
    void setFirstName(String firstName);
    
    @ShadowField
    void setSecondName(String secondName);
    
    @ShadowMethod
    void printInfo();
    
}
```

The implementation is able to infer that `setFirstName` targets the `firstName` field.

Once the shadow has been defined, we can create an instance of the shadow using the `ShadowFactory`.

The three methods from before now become...

```java
public static void setFirstName(Person person, String firstName) { 
    ShadowPerson shadow = ShadowFactory.shadow(ShadowPerson.class, person);
    shadow.setFirstName(firstName);
}

public static void setSecondName(Person person, String secondName) {
    ShadowPerson shadow = ShadowFactory.shadow(ShadowPerson.class, person);
    shadow.setSecondName(secondName);
}

public static void printInfo(Person person) {
    ShadowPerson shadow = ShadowFactory.shadow(ShadowPerson.class, person);
    shadow.printInfo();
}
```


Much better! (I think so anyway :) )
