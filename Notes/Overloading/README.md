# Method overloading

Definition: using the same method name multiple times in the same class with different parameters.

There is 2 types of overloading: method and constructor overloading.

1. Method overloading

    There are three types of method overloading:

    1.1 Number of parameters 
     ```java
        void print(int n1, int n2) { }
        void print(int n1, int n2, int n3) { }
    ```

    1.2 Type of parameters
    ```java
        void print(int n1, int n2) { }
        void print(double n1, double n2) { }
    ```

    1.3 Order of parameters.
    ```java
        void print(double n1, int n2) { }
        void print(int n1, double n2) { }
     ```  

    **Changing a variable’s name is NOT overloading.** You cannot declare more than one method with the same name and the same number and type of arguments, because the compiler cannot tell them apart.
    **Changing only the return type (same method name and parameters) will throw compilation error.** The compiler does not consider return type when differentiating methods, so you cannot declare two methods with the same signature even if they have a different return type.

2. Constructor overloading

    ```java
        public class Calculator {
            private int number1;
            private int number2;

            public Calculator(int number1) {
                this.number1 = number1;
            }
            
            public Calculator(int number1, int number2) {
                this.number1 = number1;
                this.number2 = number2;
            }
        }
    ```

### How the JVM compiles overloaded methods?

JVM is intelligently lazy: it will always exert the least possible effort to execute a method.
There are three important compiler techniques:

1. Widening (Upcasting)

    When you want to pass a smaller size data type into a bigger sized one, or when a reference variable of a subclass is accommodated in the reference variable of its superclass.

    Example: 
    ```java
        public class Test{
            public static void main(String[] args) {
                byte a = 1;
                printNumber(a);
            }

            private static void printNumber(int a) {
                System.out.println(a);
            }
        }
    ```

    For example when we pass the number 1 to the method, JVM automatically threats it as an int. If we have a method printNumber(byte a) and we pass 1 printNumber(1) we will get a compilation error.

    ```java
        public class Test {
            public static void main(String[] args) {
                Student s = new Student();
                s.printMessage();
                Person p = s;
                p.printMessage(); 
                // reference of a subclass(Student) type is widened to the reference of superclass(Person) type.
            }
        }

        class Person {
            public void printMessage() {
                System.out.println("Person");
            }
        }

        class Student extends Person {
            @Override
            public void printMessage() {
                System.out.println("Student");
            }
        }
    ```

2. Boxing (autoboxing and unboxing)

    2.1. Autoboxing - automatic conversion that the Java compiler makes between the primitive types and their corresponding object wrapper classes. For example, converting an int to an Integer, a double to a Double, and so on.

    ```java
        List<Integer> numbers = new ArrayList<>();
        for (int i = 1; i < 100; i++){
            numbers.add(i);
        }
    ```

    Although you add the int values as primitive types, rather than Integer objects, to numbers, the code compiles.That is because it creates an Integer object from i and adds the object to numbers. 

    The compiler converts the previous code to the following at runtime:

    ```java
        List<Integer> numbers = new ArrayList<>();
        for (int i = 1; i < 100; i++){
            numbers.add(Integer.valueOf(i));
        }
    ```

    2.2 Unboxing - converting an object of a wrapper type (Integer) to its corresponding primitive (int) value.

    ```java
        public static int sumEven(List<Integer> numbers) {
            int sum = 0;
            for (Integer number: numbers)
                if (number % 2 == 0){
                    sum += number;
                }
                    
            return sum;
        }
    ```

    Because the remainder (%) and unary plus (+=) operators do not apply to Integer objects,the compiler invokes the intValue method to convert an Integer to an int at runtime.

    ```java
        public static int sumEven(List<Integer> numbers) {
            int sum = 0;
            for (Integer number: numbers)
                if (number.intValue() % 2 == 0){
                    sum += number.intValue();
                }
                    
            return sum;
        }
    ```

3. Varargs

### What to remember about overloading

When overloading a method the JVM will make the least effort possible. This is the order of the laziest path to execution:
- First is widening
- Second is boxing
- Third is Varargs

Tricky situations will arise from declaring a number directly: 1 will be int and 1.0 will be double. Also remember that you can declare these types explicitly using the syntax of 1F or 1f for a float or 1D or 1d for a double.

**It is important to realize that the JVM is inherently lazy, and will always follow the laziest path to execution.**


 Resources:
 https://www.javaworld.com/article/3268983/java-challengers-1-method-overloading-in-the-jvm.html
 https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html
 https://www.decodejava.com/widening-and-narrowing-in-java.htm
 https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html
 https://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html
