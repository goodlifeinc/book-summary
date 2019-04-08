# Effective Java
## Chapter 2. Creating and Destroying Objects

### Item 1: Consider static factory methods instead of constructors

Useful video: [Item 1](https://www.youtube.com/watch?v=WUROOKn2OTk)

Static factory method is simply a static method that returns an instance of the class.

**ADVANTAGES**
- **unlike constructors, static factory methods have names**
  
  When a class seems to require multiple constructors with the same signature we better use static factory methods with carefully chosen names to highlight their differences. Using constructors with the same parameters, but in different order can cause some serious problems. The user of the API can end up calling the wrong constructor by mistake.

- **unlike constructors, static factory methods are not required to create a new object each time they’re invoked**
  
  This allows **immutable classes** to use *preconstructed instances*, or to *cache instances as they’re constructed*, and dispense them repeatedly to avoid creating unnecessary duplicate objects. The `Boolean.valueOf(boolean)` never creates an object. The ability to return the same object from repeated invocations allows classes to maintain strict control over what instances exist at any time. Classes that do this are said to be **instance controlled**.
  
  Instance controll allows:
  * a class to guarantee that it is a *singleton* or *noninstantiable*. 
  * an immutable value class to make the guarantee that no two equal instances exist: `a.equals(b)` if and only if `a==b`(Enum types provide this guarantee).

- **unlike constructors, static factory methods can return an object of any subtype of their return type**

  This gives you great flexibility in choosing the class of the returned object.
  An API can return objects without making their classes public. Hiding implementation classes in this fashion leads to a very compact API. This technique lends itself to interface-based frameworks, where interfaces provide natural return types for static factory methods.

- **the class of the returned object can vary from call to call as a function of the input parameters**
  
  Any subtype of the declared return type is permissible. The `EnumSet` class has no public constructors, only static factories. In the OpenJDK implementation, they return an instance of one of two subclasses, depending on the size of the underlying enum type: if it has sixty-four or fewer elements, as most enum types do, the static factories return a `RegularEnumSet` instance, which is backed by a **single `long`**; if the enum type has sixty-five or more elements, the factories return a ```JumboEnumSet``` instance, backed by a `long` array
  
  Clients neither know nor care about the class of the object they get back from the factory;, they care only that it is some subclass of `EnumSet`.

- **the class of the returned object need not exist when the class containing the method is written**

**DISADVANTAGES**
- **classes without public or protected constructors cannot be subclassed**

- **they are hard for programmers to find**

### Item 2: Consider a builder when faced with many constructor parameters
Useful video: [Item 2](https://www.youtube.com/watch?v=2GMp8VuxZnw)

Static factories and constructors do not scale well to large numbers of optional parameters.

*There are a few options:*

   *Telescoping constructor pattern* - you provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters.
   
   Telescoping constructor pattern works, but it is hard to write client code when there are many parameters, and harder still to read it. The reader is left wondering what all those values mean and must carefully count parameters to find out. Also this constructor invocation will require many parameters that you don't want to set, but you are forced to pass a value anyway.
   
  *JavaBeans pattern* - you call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest.
  
  Because construction is split across multiple calls, a JavaBean **may be in an inconsistent state partway through its construction.**
  ```java
    NutritionFacts cocaCola = new NutritionFacts();
    cocaCola.setServingSize(240);
    cocaCola.setServings(8);
    cocaCola.setCalories(100);
    cocaCola.setSodium(35);
    cocaCola.setCarbohydrate(27);
  ```
  
  **Builder pattern** - Instead of making the desired object directly, the client calls a constructor (or static factory) with all of the required parameters and gets a builder object. Then the client calls setter-like methods on the builder object to set each optional parameter of interest. Finally, the client calls a parameterless build method to generate the object, which is typically immutable.

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    
    public static class Builder {
        private final int servingSize;
        private final int servings;
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;
        
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        
        public Builder calories(int val) {
            calories = val; 
            return this;
        }
        
        public Builder fat(int val) {
            fat = val;
            return this;
        }
        
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        public NutritionFacts build() {
          return new NutritionFacts(this);
        } 
    }
    
    private NutritionFacts(Builder builder) {
      servingSize = builder.servingSize;
      servings = builder.servings;
      calories = builder.calories;
      fat = builder.fat;
      sodium = builder.sodium;
      carbohydrate = builder.carbohydrate;
    }
}
```

```java 
    NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                                  .calories(100)
                                  .sodium(35)
                                  .carbohydrate(27)
                                  .build();
```


### Item 3: Enforce the singleton property with a private constructor or an enum type
Useful video: [Item 3 - 5](https://www.youtube.com/watch?v=kVuOveApdCk)

A singleton is simply a class that is *instantiated exactly once.*

Making a class a singleton can make it **difficult to test** its clients because it’s impossible to substitute a mock implementation for a singleton unless it implements an interface that serves as its type. 

There are two common ways to implement singletons:

```java
  // Singleton with public final field
  public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
  }
```

```java
  // Singleton with static factory - the API makes it clear that the class is a singleton.
  public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() {
      return INSTANCE;
    }
    public void leaveTheBuilding() { ... }
  }
```

One advantage of the method above is that it gives you the flexibility to change your mind about whether the class is a singleton or not without changing the API. The factory method returns the sole instance, but it could be modified to return, say, a separate instance for each thread that invokes it.

A third way to implement a singleton is to declare a single-element enum.
```java
  // Enum singleton - the preferred approach
  public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() { ... }
  }
```

**A single-element enum type is often the best way to implement a singleton.** Note that you can’t use this approach if your singleton must extend a superclass other than `Enum`.

### Item 4: Enforce noninstantiability with a private constructor

If you want to make a class noninstantiable, **make the default constructor `private`**. Do NOT make the class abstract. The class can be subclassed and the subclass instantiated. It also misleads the user into thinking the class was designed for inheritance.

```java
  // Noninstantiable utility class
  public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
      throw new AssertionError();
     }
  }
```

The `AssertionError` isn’t strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class. It guarantees the class will never be instantiated under any circumstances.

As a side effect, this idiom also prevents the class from being subclassed. All constructors must invoke a superclass constructor, explicitly or implicitly, and a subclass would have no accessible superclass constructor to invoke.

### Item 5: Prefer dependency injection to hardwiring resources
**Static utility classes and singletons are inappropriate for classes whose behavior is parameterized by an underlying resource.**

What is required is the ability to support multiple instances of the class, each of which uses the resource desired by the client. For example we can pass the resource into the constructor when creating a new instance. This is one form of dependency injection: the dictionary is a dependency of the spell checker and is injected into the spell checker when it is created.

```java
  public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
      this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    
    public List<String> suggestions(String typo) { ... }
  }
```

Remember: **don't use a singleton or static utility class to implement a class that depends on one or more underlying resources whose behavior affects that of the class, and do not have the class create these resources directly. Instead, pass the resources, or factories to create them, into the constructor (or static factory or builder)**.

### Item 6: Avoid creating unnecessary objects

Instead of creating new objects which are equivalen, reuse a single object. It makes it more understandable and faster. **An object can always be reused if it is immutable.**

**NEVER DO THIS:**
`String s = new String("bikini");`

The statement creates a `new String` instance each time it is executed, and none of those object creations is necessary.

**Instead DO THIS:**
`String s = "bikini";`

This version uses a single `String` instance, rather than creating a new one each time it is executed.

You can avoid creating unnecessary objects by using **static factory method** in preference to constructors on immutable classes that provide both. That way you will not create a new object each time, but you can reuse already existing one. An example of this is the `Boolean.valueOf(String)`. It does not creates new objects every time it is invoked, but returns already existing values. In addition to reusing immutable objects, **you can also reuse mutable objects if you know they won’t be modified.**

If you’re going to need such an “expensive object” repeatedly, it may be advisable to cache it for reuse.

**DO NOT USE THIS:**
```java
  static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
  }
```
The problem is that it internally creates a `Pattern` instance for the regular expression and uses it only once, after which it becomes eligible for garbage collection. To improve the performance, explicitly compile the regular expression into a `Pattern` instance (which is immutable) as part of class initialization, cache it, and reuse the same instance for every invocation of the `isRomanNumeral` method.

```java
  public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
      return ROMAN.matcher(s).matches();
    }
  }
```

Another way to create unnecessary objects is autoboxing. It allows you to mix primitive and boxed primitive types, boxing and unboxing automatically as needed. Autoboxing blurs but does not erase the distinction between primitive and boxed primitive types. **Always prefer primitives to boxed primitives, and watch out for unintentional autoboxing.**

### Item 7: Eliminate obsolete object references
Memory leak example:

```java
  public class Stack {
    ...
    // Memory Leak
    public Object pop() {
      if (size == 0){
        throw new EmptyStackException();
      }

      return elements[--size];
    }
    ...
  }
```

If the stack grows and then shrinks, the objects that were poped off the stack will not be garbage collected, because the stack maintains obsolete references to these objects. An **obsolete reference is simply a reference that will never be dereferenced again**. In this case, any references outside of the “active portion” of the element array are obsolete. The active portion consists of the elements whose index is less than size. A fix for problem like this is to null out the references once they become obsolete.

In the case of our Stack class, the reference to an item becomes obsolete as soon as it’s popped off the stack.

```java
public Object pop() {
  if (size == 0){
    throw new EmptyStackException();
  }
  
  Object result = elements[--size];
  elements[size] = null; // Eliminate obsolete reference

  return result;
} 
```

The best way to eliminate an obsolete reference is to let the variable that contained the reference fall out of scope. This occurs naturally if you define each variable in the narrowest possible scope. Whenever an element is freed, any object references contained in the element should be nulled out.

Another common source of memory leaks is caches. Once you put an object reference into a cache, it’s easy to forget that it’s there and leave it in the cache long after it becomes irrelevant. 

Some solutions: 
 - If you implement a cache for which an entry is relevant exactly so long as there are references to its key outside of the cache, represent the cache as a **WeakHashMap**. Entries will be removed automatically after they become obsolete. 
 - the useful lifetime of a cache entry is less well defined, with entries becoming less valuable over time. Under these circumstances, the cache should occasionally be cleansed of entries that have fallen into disuse. This can be done by a background thread (perhaps a ScheduledThreadPoolExecutor) or as a side effect of adding new entries to the cache. 

A third common source of memory leaks is listeners and other callbacks. If you implement an API where clients register callbacks but don’t deregister them explicitly, they will accumulate. One way to ensure that callbacks are garbage collected promptly is to store only weak references to them, for instance, by storing them only as keys in a WeakHashMap.


### Item 8: Avoid finalizers and cleaners

Finalizers and cleaners are unpredictable, often dangerous, slow and generally unnecessary.

In Java, the garbage collector reclaims the storage associated with an object when it becomes unreachable and try-with-resources or try-finally block is used to reclaim other nonmemory resources. 

One shortcoming of finalizers and cleaners is that there is no guarantee they’ll be executed promptly. It can take arbitrarily long between the time that an object becomes unreachable and the time its finalizer or cleaner runs. **So you should never do anything time-critical in a finalizer or cleaner.**

**Never depend on a finalizer or cleaner to close files because open file descriptors are a limited resource**. If many files are left open as a result of the system’s tardiness in running finalizers or cleaners, a program may fail because it can no longer open files.

Providing a finalizer for a class can arbitrarily delay reclamation of its instances. Cleaners are a bit better than finalizers in this regard because class authors have control over their own cleaner threads, but cleaners still run in the background, under the control of the garbage collector, so there can be no guarantee of prompt cleaning.

**Don’t use the methods System.gc and System.runFinalization.** They may increase the odds of finalizers or cleaners getting executed, but they don’t guarantee it.

Another problem with finalizers is that **an uncaught exception thrown during finalization is ignored**, and finalization of that object terminates. **Uncaught exceptions can leave other objects in a corrupt state**. If another thread attempts to use such a corrupted object, arbitrary nondeterministic behavior may result. Normally, an uncaught exception will terminate the thread and print a stack trace, but not if it occurs in a finalizer - it won’t even print a warning. Cleaners do not have this problem because a library using a cleaner has control over its thread.

Instead of writing a finalizer or cleaner for a class whose objects encapsulate resources that require termination, such as files or threads, just **have your class implement AutoCloseable**, and require its clients to invoke the close method on each instance when it is no longer needed, typically using try-with-resources to ensure termination even in the face of exceptions.

One detail worth mentioning is that the instance must keep track of whether it has been closed: the close method must
record in a field that the object is no longer valid, and other methods must check this field and throw an `IllegalStateException` if they are called after the object has been closed.

Cleaners and finalizers have perhaps two legitimate uses:
- One is to act as a safety net in case the owner of a resource neglects to call its close method. While there’s no guarantee that the cleaner or finalizer will run promptly (or at all), it is better to free the resource late than never if the client fails to do so. If you’re considering writing such a safety-net finalizer, think long and hard about whether the protection is worth the cost. Some Java library classes, such as `FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor`, and `java.sql.Connection`, have finalizers that serve as safety nets.

- A second legitimate use of cleaners concerns objects with native peers. A native peer is a native (non-Java) object to which a normal object delegates via native methods. Because a native peer is not a normal object, the garbage collector doesn’t know about it and can’t reclaim it when its Java peer is reclaimed. A cleaner or finalizer may be an appropriate vehicle for this task, assuming the performance is acceptable and the native peer holds no critical resources. If the performance is unacceptable or the native peer holds resources that must be reclaimed promptly, the class should have a close method, as described earlier.

### Item 9: Prefer try-with-resources to try-finally

There are many resources that must be closed manually by invoking a close method. Examples include `InputStream`,
`OutputStream`, and `java.sql.Connection`.

A try-finally statement was the best way to guarantee that a resource would be closed properly, even in the face of an exception or return, but it looks terrible when you add more than one resource. **Instead of using try-finaly, use try-with-resources.** This is the the best way to close a resource. To be usable with this construct, a resource must implement the `AutoCloseable` interface, which consists of a single void-returning close method, so if you write a class that represents a resource that must be closed, your class should implement **AutoCloseable**. 

```java
// try-with-resources on multiple resources 
static void copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src); OutputStream out = new FileOutputStream(dst)) {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
      out.write(buf, 0, n);
  }
}

// try-with-resources with a catch clause
static String firstLineOfFile(String path, String defaultVal) {
  try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
  } catch (IOException e) {
    return defaultVal;
  }
}
```

## Chapter 3. Methods Common to All Objects
### Item 10: Obey the general contract when overriding equals
### Item 11: Always override hashCode when you override equals
### Item 12: Always override toString

Providing a good `toString` implementation makes your class much more pleasant to use and makes systems using the class easier to debug. 

**The returned `String` from `toString` method should be a concise but informative representation that is easy for a person to read and it is recommended that all subclasses override it.** The `toString` method should return all of the interesting information contained in the object. It is impractical if the object is large or if it contains state that is not conducive to `String` representation. Ideally, the `String` should be self-explanatory. 

One important decision you’ll have to make when implementing a `toString` method is whether to specify the format of the return value in the documentation. 

- The advantage of specifying the format is that it serves as a standard, unambiguous, human-readable representation of the object. This representation can be used for input and output and in persistent human-readable data objects, such as CSV files. If you specify the format, it’s usually a good idea to provide a matching static factory or constructor so programmers can easily translate back and forth between the object and its string representation. 

- The disadvantage of specifying the format of the `toString` return value is that once you’ve specified it, you’re stuck with it for life, assuming your class is widely used. Programmers will write code to parse the representation, to generate it, and to embed it into persistent data. If you change the representation in a future release, you’ll break their code and data, and they will yowl. By choosing not to specify a format, you preserve the flexibility to add information or improve the format in a subsequent release.

Whether or not you decide to specify the format, you should clearly document your intentions. 

It makes no sense to write a `toString` method in a static utility class. Nor should you write a `toString` method in most enum types because Java provides a perfectly good one for you. You should, however, write a `toString` method in any abstract class whose subclasses share a common string representation

### Item 13: Override clone judiciously
### Item 14: Consider implementing Comparable


## Chapter 4. Classes and Interfaces
### Item 15: Minimize the accessibility of classes and members
(Encapsulation) Information hiding decouples the components that comprise a system, allowing them to be developed, tested, optimized, understood and modified in isolation. This speeds up system development because components can be developed in parallel, eases the burden of maintenance because components can be understood more quickly and debugged or replaced with little fear of harming other components. It also increases software reuse.

There are 3 access modifiers - **private, protecred, public.**

As a rule **ALWAYS MAKE EACH CLASS OR MEMBER AS INACCESSIBLE AS POSSIBLE.**
For *top-level* classes(non nested) and interfaces there are 2 possible access levels: *public* and *package-private*.
**If a top-level class or interface can be made package-private, it should be.** If you make a class public, you are obligated to support it forever to maintain compatibility.

If a package-private top level class or interface is used by only one class *consider making it a private static nested class* of the sole class that uses it.  

For members there are 4 possible access level: **private**, **package-private**, **protected** and **public**.

The protected access modifier gives a huge increase in accessibility and it should be rarely used.

If a method overrides a superclass method, it cannot have a more restrictive access level in the sublass than in the superclass.

If a class implements an interface, all of the class methods that are in the interface must be declared public in the class.

**Instance fields of public classes should rarely be public**. If an instance field is nonfinal or is a reference to a mutable object, then by making it public, you give up the ability to limit the values that can be stored in the field.

It is wrong for a class to have a public static final array field, or an accesor that returns such a field. That way client will be able to modify the array. So always ensure that objects references by public static final fields are immutable.

```java
  // Potential security hole
  public static final Thing[] VALUES = { ... };
```

There are two ways of fixing this problem:

1. You can make the public array private and add a public immutablle list.

```java
  private static final Thing[] PRIVATE_VALUES = { ... };
  public static final List<Thing> VALUES = Collection.unmodifiableList(Array.asList(PRIVATE_VALUES));
```
2. You can make the public array private and a public method that returns a copy of a private array.
```java
  private static final Thing[] PRIVATE_VALUES = { ... };
  public static final Thing[] values() {
    return PRIVATE_VALUES.clone()
  }
```

### Item 16: In public classes, use accessor methods, not public fields

Making class fields public is a very bad idea. Such classes do not offer the benefits of encapsulation. You cannot change the representation without changing the API.

Instead you should make the fields private and create gettes and setters for access and modification.

If a class is package-private or is a private nested class, there is nothing inherently wrong with exposing its data fields—assuming they do an adequate job of describing the abstraction provided by the class.

While it’s never a good idea for a public class to expose fields directly, it is less harmful if the fields are immutable. You can’t change the representation of such a class without changing its API, and you can’t take auxiliary actions when a field is read, but you can enforce invariants.

```java
// Public class with exposed immutable fields - questionable
public final class Time {
  private static final int HOURS_PER_DAY = 24;
  private static final int MINUTES_PER_HOUR = 60;
  public final int hour;
  public final int minute;

  public Time(int hour, int minute) {
    if (hour < 0 || hour >= HOURS_PER_DAY) {
      throw new IllegalArgumentException("Hour: " + hour);
    }

    if (minute < 0 || minute >= MINUTES_PER_HOUR) {
      throw new IllegalArgumentException("Min: " + minute);
    }
      
    this.hour = hour;
    this.minute = minute;
  }
}
```

**Public classes should never expose mutable fields. It is less harmful, though still questionable, for public classes to expose immutable fields. It is, however, sometimes desirable for package-private or private nested classes to expose fields, whether mutable or immutable.**

### Item 17: Minimize mutability

An immutable class is simply a class whose instances cannot be modified. Example of such classes are `String`, `BigInteger`, `BigDecimal`. Immutable classes  provide many advantages, and their only disadvantage is the potential for performance problems under certain circumstances.

There are 5 rules to make a class immutable:

1. **Don't provide methods that modify the object's state**
2. **Ensure that the class cannot be extended** (final class ot static factory methods with private constructors)
3. **Make all fields final**
4. **Make all fields private**
5. **Ensure exclusive access to any mutable components** - if your class has any fields that refer to mutable objects, ensure that clients of the class cannot obtain references to there objects.

Example:

```java
  public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
      this.re = re;
      this.im = im;
    }

    public double realPart() {
      return re;
    }

    public double imaginaryPart() {
      return im;
    }

    public Complex plus(Complex c){
      return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c){
      return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c){
      return new Complex(re * c.re - im * c.im,
                        re * c.re + im * c.im);
    }

    public Complex dividedBy(Complex c) {
      double tmp = c.re * c.re + c.im * c.im;
      return new Complex((re * c.re + im * c.im) / tmp, 
                        (im * c.re - re * c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
      if (o == this){
        return true;
      }

      if (!(o instanceof Complex)){
        return false;
      }

      Complex c = (Complex) o;

      return Double.compare(c.re, re) == 0 && 
            Double.compare(c.im, im) == 0;
    }

    @Override 
    public int hashCode() {
      return 31 * Double.hashCode(re) +
                  Double.hashCode(im);
    }

    @Override public String toString() {
      return "(" + re + " + " + im + "i)";
    }
  }
```

In this example we can see that the arithmetic operations create and return a new Complex instance rather than modifying this instance. Also the method names are prepositions (such as plus) rather than verbs (such as add). This emphasizes the fact that methods don’t change the values of the objects. An immutable object can be in exactly one state, the state in which it was created. 

Immutable classes are inherently **thread-safe**; they require no synchronization. They cannot be corrupted by multiple threads accessing them concurrently. This is far and away the easiest approach to achieve thread safety. 

Immutable classes should encourage clients to **reuse existing instances wherever possible**.
One easy way to do this is to provide **public static final constants** for commonly used values. 

```java
  public static final Complex ZERO = new Complex(0, 0);
  public static final Complex ONE = new Complex(1, 0);
  public static final Complex I = new Complex(0, 1);
```

An immutable class can provide **static factories that cache frequently requested instances to avoid creating new instances** when existing ones would do. This causes clients to share instances instead of creating new ones, reducing memory footprint and garbage collection costs. 

Opting for static factories in place of public constructors when designing a new class gives you the flexibility to add caching later, without modifying clients.

The major **disadvantage** of immutable classes is that they require a separate object for each distinct value. Creating these objects can be costly, especially if they are large. 

If we want to guarantee immutability, a class must not permit itself to be subclassed. We can make the class final, but there is another alternative. We can make all of its constructors private or package-private and add public static factories in place of the public constructors.

```java
  // Immutable class with static factories instead of constructors
  public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
      this.re = re;
      this.im = im;
    }

    public static Complex valueOf(double re, double im) {
      return new Complex(re, im);
    }
  }
```

Some immutable classes have one or more nonfinal fields in which they cache the results of expensive computations the first time they are needed. If the same value is requested again, the cached value is returned, saving the cost of recalculation. This trick works precisely because the object is immutable, which guarantees that the computation would yield the same result if it were repeated. This technique, an example of lazy initialization, is also used by String.

There are some classes for which immutability is impractical. If a class cannot be made immutable, limit its mutability as much as possible. Reducing the number of states in which an object can exist makes it easier to reason about the object and reduces the likelihood of errors. Therefore, make every field final unless there is a compelling reason to make it nonfinal.

### Item 18: Favor composition over inheritance

**Inheritance violates encapsulation, because a sublass depends on the implementational details of its superclass to function properly and if the superclass's implementation change in a new release, the sublass may break even though its code has not been touched. So our sublass must evolve in tandem with its superclass.**

If the superclass acquires a new method in a subsequent release and you have the bad luck to have given the subclass a method with the same signature and a different return type, your subclass will no longer compile.

**Instead of extending an existing class use composition: give your new class a private field that references an instance of the existing class** (existing class becomes a component of the new one).

Each instance method in the new class invokes the corresponding method on the contained instance of the existing class and returns the results. This is known as forwarding, and the methods in the new class are known as forwarding methods. The resulting class will be rock solid, with no dependencies on the implementation details of the existing class. Even adding new methods to the existing class will have no impact on the new class. 

Inheritance is appropriate only in circumstances where the subclass really is a subtype of the superclass. In other words, a class B should extend a class A only if an “is-a” relationship exists between the two classes. I

### Item 19: Design and document for inheritance or else prohibit it
### Item 20: Prefer interfaces to abstract classes 
### Item 21: Design interfaces for posterity

### Item 22: Use interfaces only to define types

When a class implements an interface, the interface serves as a type that can be used to refer to instances of the class. That a class implements an interface should therefore say something about what a client can do with instances of the class. 

**Interfaces should be used only to define types. They should not be used merely to export constants.**

Constant interface (interface that contains no methods only static final fields, each exporting a constant)

```java
  // Constant interface antipattern - DO NOT USE!
  public interface PhysicalConstants {
    // Avogadro's number (1/mol)
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    // Boltzmann constant (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    // Mass of the electron (kg)
    static final double ELECTRON_MASS = 9.109_383_56e-31;
  }
```

If in a future release the class is modified so that it no longer needs to use the constants, it still must implement the interface to ensure binary compatibility. If a nonfinal class implements a constant interface, all of its subclasses will have their namespaces polluted by the constants in the interface.

If you want to export constants, there are several reasonable choices.
If the constants are strongly tied to an existing class or interface, you should add them to the class or interface. If the constants are best viewed as members of an enumerated type, you should export them with an enum type. Otherwise, you should export the constants with a noninstantiable utility class.

```java
  // Constant utility class
  package com.effectivejava.science;

  public class PhysicalConstants {
    private PhysicalConstants() { } // Prevents instantiation

    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    public static final double BOLTZMANN_CONST = 1.380_648_52e-23;
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
  }
```

### Item 23: Prefer class hierarchies to tagged classes

Tagged class is a class whose instances come in two or more flavors and contain a tag field indicating the flavor of the instance. For example:

```java
// Tagged class - vastly inferior to a class hierarchy!
 class Figure {
    enum Shape { RECTANGLE, CIRCLE };
    // Tag field - the shape of this figure
    final Shape shape;
    
    // These fields are used only if shape is RECTANGLE
    double length;
    double width;
  	
    // This field is used only if shape is CIRCLE
    double radius;
    
    // Constructor for circle
    Figure(double radius) {
      shape = Shape.CIRCLE;
      this.radius = radius;
    }

    // Constructor for rectangle
    Figure(double length, double width) {
      shape = Shape.RECTANGLE;
      this.length = length;
      this.width = width;
    }

    double area() {
      switch(shape) {
        case RECTANGLE:
          return length * width;
        case CIRCLE:
          return Math.PI * (radius * radius);
        default:
          throw new AssertionError(shape);
      }
    }
}
```

Tagged classes have numerous shortcomings:
- readability is harmed because multiple implementations are jumbled together in a single class

- memory footprint is increased because instances are burdened with irrelevant fields belonging to other flavors

- fields can’t be made final unless constructors initialize irrelevant fields, resulting in more boilerplate

- constructors must set the tag field and initialize the right data fields with no help from the compiler: if you initialize the wrong fields, the program will fail at runtime

- you can’t add a flavor to a tagged class unless you can modify its source file. If you do add a flavor, you must remember to add a case to every switch statement, or the class will fail at runtime

- the data type of an instance gives no clue as to its flavor

There is a better implementation, using class hierarchy. First we define an abstract class containing an abstract method for each method in the tagged class whose behavior depends on the tag value. Next, define a concrete subclass of the root class for each flavor of the original tagged class. 

```java
  // Class hierarchy replacement for a tagged class
  abstract class Figure {
    abstract double area();
  }

  class Circle extends Figure {
    final double radius;
  
    Circle(double radius) { 
      this.radius = radius; 
    }
  
    @Override
    double area() {
       return Math.PI * (radius *radius); 
    }
  } 

  class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
      this.length = length;
      this.width = width;
    }

    @Override 
    double area() { 
      return length * width; 
    }
  }
```

This code is much more simple and clear. The implementation of each flavor is allotted its own class, and none of these classes is encumbered by irrelevant data fields. All fields are final. The compiler ensures that each class’s constructor initializes its data fields and that each class has an implementation for every abstract method declared in the root class. This eliminates the possibility of a runtime failure due to a missing switch case. There is a separate data type associated with each flavor, allowing programmers to indicate the flavor of a variable and to restrict variables and input parameters to a particular flavor. Another advantage of class hierarchies is that they can be made to reflect natural hierarchical relationships among types, allowing for increased flexibility and better compile-time type checking. 

Suppose the tagged class in the first example also allowed for squares. The class hierarchy could be made to reflect the fact that a square is a special kind of rectangle (assuming both are immutable):

```java
class Square extends Rectangle {
  Square(double side) {
    super(side, side);
  }
}
```


**The default method construct was added to allow the addition of methods to existing interfaces.** The declaration for a default method includes a default implementation that is used by all classes that implement the interface but do not implement the default method.

While the addition of default methods makes it possible to add methods to an existing interface, there is no guarantee that these methods will work in all preexisting implementations. 

Using default methods to add new methods to existing interfaces should be avoided unless the need is critical, in which case you should think long and hard about whether an existing interface implementation might be broken by your default method implementation. Default methods are, however, extremely useful for providing standard method implementations when an interface is created, to ease the task of implementing the interface.

If an interface contains a minor flaw, it may irritate its users forever, if an interface is severely deficient, it may doom the API that contains it.

### Item 24: Favor static member classes over nonstatic

**A nested class is a class defined within another class.**

A nested class should exist only to serve its enclosing class. If a nested class would be useful in some other context, then it should be a top-level class. 

There are four kinds of nested classes: 
- static member classes
- nonstatic member classes
- anonymous classes
- local classes

Nonstatic member classes, anonymous and local classes are  known as inner classes.

1. Static member class:

  A class that is declared inside another class and has access to all of the enclosing class's members, even those declated private. One common use of a static member class is as a **public helper class**, useful only in conjunction with its outer class. 
  For example: Calculator.Operation.PLUS;
 
2. Nonstatic member class:

  Each instance of a nonstatic member class is implicitly associated with an enclosing instance of its containing class. Within instance methods of a nonstatic member class, you can invoke methods on the enclosing instance or obtain a reference to the enclosing instance. If an instance of a nested class can exist in isolation from an instance of its enclosing class, then the nested class must be a static member class: **it is impossible to create an instance of a nonstatic member class without an enclosing instance.**

  ```java
    // Typical use of a nonstatic member class
    public class MySet<E> extends AbstractSet<E> {
      @Override 
      public Iterator<E> iterator() {
        return new MyIterator();
      }

      private class MyIterator implements Iterator<E> {
        //...
      }
    }
  ```

3. Anonymous class

  A class with no name. It is simultaneously declared and instantiated at the point of use. Anonymous classes are permitted at any point in the code where an expression is legal. Anonymous classes have enclosing instances if and only if they occur in a nonstatic context. But even if they occur in a static context, they cannot have any static members other than constant variables, which are final primitive or string fields initialized to constant expressions.

  - You can’t instantiate anonymous classes except at the point they’re declared.
  - You can’t perform instanceof tests or do anything else that requires you to name the class.
  - You can’t declare an anonymous class to implement multiple interfaces or to extend a class and implement an interface at the same time. 
  - Clients of an anonymous class can’t invoke any members except those it inherits from its supertype.

4. Local class:
  
  A local class can be declared practically anywhere a local variable can be declared and obeys the same scoping rules. They have names and can be used repeatedly. Also, they have enclosing instances only if they are defined in a nonstatic context, and they cannot contain static members and like anonymous classes. They should be kept short so as not to harm readability.

If a nested class needs to be visible outside of a single method or is too long to fit comfortably inside a method, use a member class. If each instance of a member class needs a reference to its enclosing instance, make it nonstatic; otherwise, make it static. Assuming the class belongs inside a method, if you need to create instances from only one location and there is a preexisting type that characterizes the class, make it an anonymous class; otherwise, make it a local class.

### Item 25: Limit source files to a single top-level class


## Chapter 5. Generics
### Item 26: Don’t use raw types

If you use raw types for example only `List` and you want to add only numbers to it, then accidentaly add `String` and try to foreach and sum the elements, you won’t discover the error until runtime, long after it has happened.

**Parameterized collection adds typesafety.**

For example: `private final Collection<Stamp> stamps = ... ;`

From this declaration, the compiler knows that stamps should contain only `Stamp `instances and guarantees it to be true, assuming your entire codebase compiles without emitting any warnings. 

**The compiler inserts invisible casts for you when retrieving elements from collections and guarantees that they won’t fail.**

Difference between the raw type `List` and the parameterized type `List<Object>`:
  The former has opted out of the generic type system, while the latter has explicitly told the compiler that it is capable of holding objects of any type. While you can pass a `List<String>` to a parameter of type `List`, you can’t pass it to a parameter of type `List<Object>`. There are sub-typing rules for generics, and `List<String>` is a subtype of the raw type `List`, but not of the parameterized type `List<Object>`. As a consequence, **you lose type safety if you use a raw type such as `List`, but not if you use a parameterized type such as `List<Object>`.**

If you want to use a generic type but you don’t know or care what the actual type parameter is, you can use a **question mark instead (unbounded wildcard types)**.
 For example, the unbounded wildcard type for the generic type `Set<E>` is `Set<?>`. It is the most general parameterized Set type, capable of holding any set. The wildcard type is safe and the raw type isn’t. You can put any element into a collection with a raw type, easily corrupting the collection’s type invariant. Not only can’t you put any element (other than null) into a `Collection<?>`, but you can’t assume anything about the type of the objects that you get out. If these restrictions are unacceptable, you can use generic methods or bounded wildcard types.

Because generic type information is erased at runtime, it is illegal to use the `instanceof` operator on parameterized types other than unbounded wildcard types. The use of unbounded wildcard types in place of raw types does not affect the behavior of the instanceof operator in any way. In this case, the angle brackets and question marks are just noise. This is the preferred way to use the instanceof operator with generic types:

```java
// Legitimate use of raw type - instanceof operator
if (o instanceof Set) { // Raw type
  Set<?> s = (Set<?>) o; // Wildcard type
  ...
}
```

| Term                    | Example | 
| :-----------------------|---------| 
| Parameterized type      | `List<String>`
| Actual type parameter   | `String`
| Generic type            | `List<E>`
| Formal type parameter   | `E`
| Unbounded wildcard type | `List<?>`
| Raw type                | `List`
| Bounded type parameter  | `<E extends Number>`
| Recursive type bound    | `<T extends Comparable<T>>`
| Bounded wildcard type   | `List<? extends Number>`
| Generic method          | `static <E> List<E>` `asList(E[] a)`
| Type token              | `String.class`

### Item 27: Eliminate unchecked warnings
### Item 28: Prefer lists to arrays
### Item 29: Favor generic types
### Item 30: Favor generic methods
### Item 31: Use bounded wildcards to increase API flexibility
### Item 32: Combine generics and varargs judiciously
### Item 33: Consider typesafe heterogeneous containers


## Chapter 6. Enums and Annotations
### Item 34: Use enums instead of int constants
### Item 35: Use instance fields instead of ordinals
### Item 36: Use EnumSet instead of bit fields
### Item 37: Use EnumMap instead of ordinal indexing
### Item 38: Emulate extensible enums with interfaces
### Item 39: Prefer annotations to naming patterns
### Item 40: Consistently use the Override annotation
### Item 41: Use marker interfaces to define types


## Chapter 7. Lambdas and Streams
### Item 42: Prefer lambdas to anonymous classes
### Item 43: Prefer method references to lambdas
### Item 44: Favor the use of standard functional interfaces
### Item 45: Use streams judiciously
### Item 46: Prefer side-effect-free functions in streams
### Item 47: Prefer Collection to Stream as a return type
### Item 48: Use caution when making streams parallel


## Chapter 8. Methods
### Item 49: Check parameters for validity
### Item 50: Make defensive copies when needed
### Item 51: Design method signatures carefully
### Item 52: Use overloading judiciously
### Item 53: Use varargs judiciously
### Item 54: Return empty collections or arrays, not nulls
### Item 55: Return optionals judiciously
### Item 56: Write doc comments for all exposed API elements


## Chapter 9. General Programming
### Item 57: Minimize the scope of local variables

By minimizing the scope of local variables, you increase the readability and maintainability of your code and reduce the likelihood of error.

Techniques for minimazing the scope of a local variable:

- **Declare variable where it is first used** - if you declare all the variables at the head of the block, the reader might not remember the variables type or ilitial value. The scope of a local variable extends from the point where it is declared to the end of the enclosing block. If a variable is declared outside of the block in which it is used, it remains visible after the program exits that block and if it is used accidentally before or after its region of intended use, the consequinces can be desastrous.

- **If a variable is initialized to an expression whose evaluation can throw a checked exception, the variable must be initialized inside a try block**. If the value must be used outside of the try block, then it must be declared before the try block, where it cannot yet be “sensibly initialized.” 

- **Prefer for loops to while loops** - for loops in both forms: traditional and for-each, allows you to declare loop variables, limiting their scope to the exact region where they are needed.

- **Keep methods small and focused** - if you combine two activities in the same method, local variables relevant to one activity may be in the scope of the code performing the other activity. To prevent this from happening, simply separate the method into two: one for each activity.

### Item 58: Prefer for-each loops to traditional for loops

The for-each loop provides compelling advantages over the traditional for loop in clarity, flexibility, and bug prevention, with no performance penalty. The for-each loop gets rid of the clutter and the opportunity for error by hiding the iterator or index variable.

Unfortunately, there are three common situations where you can’t use foreach:

- **Destructive filtering** - If you need to traverse a collection removing selected elements, then you need to use an explicit iterator so that you can call its remove method. You can often avoid explicit traversal by using Collection’s removeIf method, added in Java 8.

- **Transforming** - If you need to traverse a list or array and replace some or all of the values of its elements, then you need the list iterator or array index in order to replace the value of an element.

- **Parallel iteration** - If you need to traverse multiple collections in parallel, then you need explicit control over the iterator or index variable so that all iterators or index variables can be advanced in lockstep.

If possible always use for-each loop instead of any other loop.

### Item 59: Know and use the libraries

### Item 60: Avoid float and double if exact answers are required

Don’t use ```float``` or ```double``` for any calculations that require an exact answer (for example: monetary calculations - it is impossible to represent 0.1 as a ```float``` or ```double``` exactly). They are designed primarily for scientific and engineering calculations. Instead use ```BigDecimal```, ```int```, or ```long```.

There are two **disadvantages** to using ```BigDecimal```: it’s a lot less convenient than using a primitive arithmetic type, and it’s a lot slower.

An alternative to using ```BigDecimal``` is to use ```int``` or ```long```, depending on the amounts involved, and to keep track of the decimal point yourself.

If the quantities don’t exceed nine decimal digits, you can use ```int```; if they don’t exceed eighteen digits, you can use ```long```. If the quantities might exceed eighteen digits, use ```BigDecimal```.


### Item 61: Prefer primitive types to boxed primitives
The boxed primitives corresponding to int, double, and boolean are Integer, Double, and Boolean.

There are three major differences between primitives and boxed primitives:

- **primitives have only their values, whereas boxed primitives have identities distinct from their values.** In other words, two boxed primitive instances can have the same value and different identities. 

- **primitive types have only fully functional values, whereas each boxed primitive type has one nonfunctional value, which is null**, in addition to all the functional values of the corresponding primitive type. 

- **primitives are more timeand space-efficient than boxed primitives**

**Example 1:**
```java
  naturalOrder.compare(new Integer(42), new Integer(42)) 
```
In the example above both ```Integer``` instances represent the same value (42), so the value of this expression should be 0, but it’s 1, which indicates that the first ```Integer``` value is greater than the second.

Evaluating the expression ```i < j``` causes the ```Integer``` instances referred to by ```i``` and ```j``` to be auto-unboxed (it extracts their primitive values). The evaluation proceeds to check if the first of the resulting int values is less than the second. But suppose it is not. Then the next test evaluates the expression ```i==j```, which performs an identity comparison on the two object references. **Applying the == operator to boxed primitives is almost always wrong.**

Yout could fix the problem in the broken comparator by adding two local variables to store the primitive ```int``` values corresponding to the boxed ```Integer``` parameters, and performing all of the comparisons on these variables. 

```java
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
  int i = iBoxed
  int j = jBoxed;

  return i < j ? -1 : (i == j ? 0 : 1);
};
```


**Example 2:**
```java
public class Unbelievable {
  static Integer i;
  
  public static void main(String[] args) {
    if (i == 42){
      System.out.println("Unbelievable");
    }
  }
}
```

The example above throws a ```NullPointerException``` when evaluating the expression ```i==42```. The problem is that ```i``` is an ```Integer```, not an ```int```, and like all nonconstant object reference fields, its initial value is ```null```. When the program evaluates the expression ```i==42```, it is comparing an ```Integer``` to an ```int```. 

In nearly every case when you mix primitives and boxed primitives in an operation, the boxed primitive is auto-unboxed. If a ```null``` object reference is auto-unboxed, you get a ```NullPointerException```. 

**Example 3:**  
```java
public static void main(String[] args) {
  Long sum = 0L;
  for (long i = 0; i < Integer.MAX_VALUE; i++) {
    sum += i;
  }
  
  System.out.println(sum);
}
```

The program above is much slower than it should be because of the boxed primitive type ```Long```. The program compiles without error or warning, and the variable is repeatedly boxed and unboxed, causing the observed performance degradation.


### Item 62: Avoid strings where other types are more appropriate

Often when a piece of data comes into a program from a file, from the network, or from keyboard input, it is often in a string form. So there is a natural tendency to leave it that way, but this tendency is justified only if the data really is textual in nature.

If the data is numeric, it should be translated into the appropriate numeric type, such as int, float, or BigInteger. If it’s the answer to a yes-or-no question, it should be translated into an appropriate enum type or a boolean.  More generally, if there’s an appropriate value type, whether primitive or object reference, you should use it; if there isn’t, you should write one. Strings are poor substitutes for enum types and aggregate types. 

If an entity has multiple components, it is usually a bad idea to represent it as a single string. To access individual fields, you have to parse the string, which is slow, tedious, and error-prone. A better approach is simply to write a class to represent the aggregate, often a private static member class.

### Item 63: Beware the performance of string concatenation

Using the string concatenation operator repeatedly to concatenate n strings requires time quadratic in n. This is an unfortunate consequence of the fact that strings are immutable. When two strings are concatenated, the contents of both are copied.

**Don’t use the string concatenation operator to combine more than a few strings** unless performance is irrelevant. Use StringBuilder’s append method instead. Alternatively, use a character array, or process the strings one at a time instead of combining them.

### Item 64: Refer to objects by their interfaces
### Item 65: Prefer interfaces to reflection
### Item 66: Use native methods judiciously
### Item 67: Optimize judiciously
### Item 68: Adhere to generally accepted naming conventions


## Chapter 10. Exceptions
### Item 69: Use exceptions only for exceptional conditions
### Item 70: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
### Item 71: Avoid unnecessary use of checked exceptions
### Item 72: Favor the use of standard exceptions
### Item 73: Throw exceptions appropriate to the abstraction
### Item 74: Document all exceptions thrown by each method
### Item 75: Include failure-capture information in detail messages
### Item 76: Strive for failure atomicity
### Item 77: Don’t ignore exceptions


## Chapter 11. Concurrency
## Chapter 12. Serialization
