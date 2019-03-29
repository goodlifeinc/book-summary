# Effective Java
## Chapter 2. Creating and Destroying Objects

### Item 1: Consider static factory methods instead of constructors

Useful video: [Item 1](https://www.youtube.com/watch?v=WUROOKn2OTk)

Static factory method is simply a static method that returns an instance of the class.

**ADVANTAGES**
- **unlike constructors, static factory methods have names**
  
  It is better to use static factory methods when a class seems to require multiple constructors with the same signature. Then we replace the constructors with static factory methods and carefully chosen names to highlight their differences. Using constructors with the same parameters, but in different order can cause some serious problems. The user of the API can end up calling the wrong constructor by mistake.

- **unlike constructors, static factory methods are not required to create a new object each time they’re invoked**
  
  This allows immutable classes to use preconstructed instances, or to cache instances as they’re constructed, and dispense them repeatedly to avoid creating unnecessary duplicate objects. The ```Boolean.valueOf(boolean)``` never creates an object. This ability, to return the same object from repeated invocations allows classes to maintain strict control over what instances exist at any time. Classes that do this are said to be **instance controlled**.
  
  Instance controll allows:
  * a class to guarantee that it is a singleton or noninstantiable 
  * an immutable value class to make the guarantee that no two equal instances exist: ```a.equals(b)``` if and only if ```a == b```

- **unlike constructors, static factory methods can return an object of any subtype of their return type**

  This gives you great flexibility in choosing the class of the returned object.
  
  One application of this flexibility is that an API can return objects without making their classes public. Hiding implementation classes in this fashion leads to a very compact API. This technique lends itself to interface-based frameworks, where interfaces provide natural return types for static factory methods.

- **the class of the returned object can vary from call to call as a function of the input parameters**
  
  Any subtype of the declared return type is permissible. The EnumSet class has no public constructors, only static factories. In the OpenJDK implementation, they return an instance of one of two subclasses, depending on the size of the underlying enum type: if it has sixty-four or fewer elements, as most enum types do, the static factories return a RegularEnumSet instance, which is backed by a single long; if the enum type has sixty-five or more elements, the factories return a JumboEnumSet instance, backed by a long array.
  
  Clients neither know nor care about the class of the object they get back from the factory; they care only that it is some subclass of EnumSet.

- **the class of the returned object need not exist when the class containing the method is written**

**DISADVANTAGES**
- **classes without public or protected constructors cannot be subclassed**

- **they are hard for programmers to find**

### Item 2: Consider a builder when faced with many constructor parameters
Useful video: [Item 2](https://www.youtube.com/watch?v=2GMp8VuxZnw)

Static factories and constructors do not scale well to large numbers of optional parameters.

*There are a few options:*

   *Telescoping constructor pattern* - you provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters.
   
   Telescoping constructor pattern works, but it is hard to write client code when there are many parameters, and harder still to read it. The reader is left wondering what all those values mean and must carefully count parameters to find out.
   
  *JavaBeans pattern* - you call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest.
  
  Because construction is split across multiple calls, a JavaBean may be in an inconsistent state partway through its construction.
  
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

Making a class a singleton can make it difficult to test its clients because it’s impossible to substitute a mock implementation for a singleton unless it implements an interface that serves as its type. 

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
  // Singleton with static factory
  public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() {
      return INSTANCE;
    }
    public void leaveTheBuilding() { ... }
  }
```

A third way to implement a singleton is to declare a single-element enum.
```java
  // Enum singleton - the preferred approach
  public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() { ... }
  }
```
**A single-element enum type is often the best way to implement a singleton.** Note that you can’t use this approach if your singleton must extend a superclass other than ```Enum```.


### Item 4: Enforce noninstantiability with a private constructor

If you want to make a class noninstantiable, make the default constructor ```private```. Do not make the class abstract. The class can be subclassed and the subclass instantiated. It also misleads the user into thinking the class was designed for inheritance.

```java
  // Noninstantiable utility class
  public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
      throw new AssertionError();
     }
  }
```

The ```AssertionError``` isn’t strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class. It guarantees the class will never be instantiated under any circumstances.

As a side effect, this idiom also prevents the class from being subclassed. All constructors must invoke a superclass constructor, explicitly or implicitly, and a subclass would have no accessible superclass constructor to invoke.

### Item 5: Prefer dependency injection to hardwiring resources
### Item 6: Avoid creating unnecessary objects
Instead of creating new objects which are equivalen, reuse a single object. It makes it more understandable and faster. An object can always be reused if it is immutable.

**NEVER DO THIS:**
```String s = new String("bikini");```

The statement creates a ```new String``` instance each time it is executed, and none of those object creations is necessary.

**Instead DO THIS:**
```String s = "bikini";```

This version uses a single ```String``` instance, rather than creating a new one each time it is executed.

You can avoid creating unnecessary objects by using **static factory method** in preference to constructors on immutable classes that provide both. That way you will not create a new object each time, but you can reuse already existing one. An example of this is the ```Boolean.valueOf(String)```. It does not creates new objects every time it is invoked, but returns already existing values. In addition to reusing immutable objects, you can also reuse mutable objects if you know they won’t be modified.

If you’re going to need such an “expensive object” repeatedly, it may be advisable to cache it for reuse.

**DO NOT USE THIS:**
```java
  static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
  }
```
The problem is that it internally creates a ```Pattern``` instance for the regular expression and uses it only once, after which it becomes eligible for garbage collection. To improve the performance, explicitly compile the regular expression into a ```Pattern``` instance (which is immutable) as part of class initialization, cache it, and reuse the same instance for every invocation of the ```isRomanNumeral``` method.

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
### Item 8: Avoid finalizers and cleaners
### Item 9: Prefer try-with-resources to try-finally


## Chapter 3. Methods Common to All Objects
### Item 10: Obey the general contract when overriding equals
### Item 11: Always override hashCode when you override equals
### Item 12: Always override toString
### Item 13: Override clone judiciously
### Item 14: Consider implementing Comparable


## Chapter 4. Classes and Interfaces
### Item 15: Minimize the accessibility of classes and members
### Item 16: In public classes, use accessor methods, not public fields
### Item 17: Minimize mutability
### Item 18: Favor composition over inheritance
### Item 19: Design and document for inheritance or else prohibit it
### Item 20: Prefer interfaces to abstract classes
### Item 21: Design interfaces for posterity
### Item 22: Use interfaces only to define types
### Item 23: Prefer class hierarchies to tagged classes
### Item 24: Favor static member classes over nonstatic
### Item 25: Limit source files to a single top-level class


## Chapter 5. Generics
### Item 26: Don’t use raw types
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
  Most methods and constructors have some restrictions on what values may be passed into their parameters. It is not uncommon that index values must be non-negative and object references must be non-null, so you should clearly document all such restrictions and enforce them with checks **at the beginning of the method body** (you should attempt to detect errors as soon as possible after they occur).

The  ```Objects.requireNonNull ``` method is flexible and convenient, so there’s no reason to perform  ```null ``` checks manually. Nonpublic methods can check their parameters using assertions:

  ```java
  private static void sort(long a[], int offset, int length) {
      assert a != null;
      assert offset >= 0 && offset <= a.length;
      assert length >= 0 && length <= a.length - offset;
      ... // Do the computation
  }
  ```
  
  Each time you write a method or constructor, you should think about what restrictions exist on its parameters. You should document these restrictions and enforce them with explicit checks at the beginning of the method body.

### Item 50: Make defensive copies when needed

### Item 51: Design method signatures carefully
- Choose method names carefully.

- Avoid long parameter lists - Aim for four parameters or fewer.
  * break the method up into multiple methods, each of which requires only a subset of the parameters.
  * create helper classes to hold groups of parameters (Typically these are static member classes)
  * adapt the Builder pattern from object construction to method invocation.

- For parameter types, favor interfaces over classes - By using a class instead of an interface, you restrict your client to a particular implementation and force an unnecessary and potentially expensive copy operation if the input data happens to exist in some other form.

- Prefer two-element enum types to boolean parameters - Enums make your code easier to read, write and easy to add more options later.
  * Ex: 
  ```java
    public enum TemperatureScale { 
      FAHRENHEIT, CELSIUS 
    }
  ```

### Item 52: Use overloading judiciously
### Item 53: Use varargs judiciously
### Item 54: Return empty collections or arrays, not nulls
### Item 55: Return optionals judiciously
### Item 56: Write doc comments for all exposed API elements


## Chapter 9. General Programming
### Item 57: Minimize the scope of local variables
### Item 58: Prefer for-each loops to traditional for loops
### Item 59: Know and use the libraries
### Item 60: Avoid float and double if exact answers are required
### Item 61: Prefer primitive types to boxed primitives
### Item 62: Avoid strings where other types are more appropriate
### Item 63: Beware the performance of string concatenation
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
