# Singleton Design Pattern

[Useful video](https://www.youtube.com/watch?v=hUE_j6q0LTQ)

The Singleton Pattern is a convention for ensuring one and only one object is instantiated for a given class (there is only **one instance** of the class). It also gives us a **global point of access**, just like a global variable.  And with the Singleton Pattern, we can create our objects only when they are needed.

```java
public class Singleton {
    private static Singleton uniqueInstance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }

        return uniqueInstance;
    }
}

public static void main(String[] args) {
    Singleton s = Singleton.getInstance();
}

```

If we never need the instance, it never gets created. This is **lazy instantiation**.

Example:
```java
public class ChocolateBoiler {
    private static ChocolateBoiler boiler;

    private boolean empty;
    private boolean boiled;

    private ChocolateBoiler() {
        empty = true;
        boiled = false;
    }

    public static ChocolateBoiler getInstance(){
        if (boiler == null) {
            boiler = new ChocolateBoiler();
        }

        return boiler;
    }

    public void fill() { // fill the boiler with a milk/chocolate mixture
        if (isEmpty()) {
            empty = false;
            boiled = false;
        }
    }

    public void drain() { // drain the boiled milk and chocolate
        if (!isEmpty() && isBoiled()) {
            empty = true;
        }
    }
    public void boil() { // bring the contents to a boil
        if (!isEmpty() && !isBoiled()) {
            boiled = true;
        }
    }
    public boolean isEmpty() {
        return empty;
    }
    public boolean isBoiled() {
        return boiled;
    }
}
```
