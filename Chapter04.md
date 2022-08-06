# Effective Java Chapter 04 Notes

### Minimize the accessibility of classes members
- **Information hiding or encapsulation**
    - A good designed class hides all implemenations details and cleanly separate its API from its implementation 
    - Communication done through API 
    - Decouples components, allowing them to be developed, test, understood and modified in isolation 
    - Components can be developed in parallel, which speed up the development process
    - Eases the burden of maintenance, components are isolated and small as possible what makes them easy to understand 
    - It enables effective performance tuning: once a system is complete and profiling has determined which components are causing performance problems, those components can be optimized without affecting the correctness of others.
    - Information hiding decreases the risk in building large systems because individual components may prove successful even if the system does not.
- You should start by making class members private, after that package-private
    - as private and package-private (default level) considered part of implementation not the API export by the class
- Making members protected or public shoud be rare as possible as this means that you will have to support them for ever if any client depends on them 
- If a method overrides a superclass method, it cannot have a more restrictive access level in the subclass than in the superclass [JLS, 8.4.8.3]. This is necessary to ensure that an instance of the subclass is usable anywhere that an instance of the superclass is usable (the Liskov substitution principle) 
    -  A special case of this rule is that if a class implements an interface, all of the class methods that are in the interface must be declared public in the class.

- Instance fields of public classes should rarely be public
- Classes with public mutable fields are not generally thread-safe, even if the fields are final or static one exception to this is field declared public static final, that contains primitive or reference to an immutable object 
    - a field containing a reference to a mutable object has all the disadvantages of a nonfinal field. While the reference cannot be modified, the referenced object can be modified—with disastrous results 
    - Note that a nonzero-length array is always mutable, so it is wrong for a class to have a public static final array field, or an accessor that returns such a field.

    ```java
    // Potential security hole!
    public static final Thing[] VALUES = { ... };
    ```
    - Don't return reference through accessor, there two solutions to this problem 
    
    ```java 
    /* Beware of the fact that some IDEs generate accessors that return references to private array fields, 
       resulting in exactly this problem. There are two ways to fix the problem. 
       You can make the public array private and add a public immutable list: */

    private static final Thing[] PRIVATE_VALUES = { ... };
    public static final List<Thing> VALUES =
    // Make unmodifiable copy  accessible 
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));

    // returns a copy of a private array:
    private static final Thing[] PRIVATE_VALUES = { ... };
    public static final Thing[] values() {
        return PRIVATE_VALUES.clone();
    }
    ```

- As of Java 9, there are two additional, implicit access levels introduced as part of the module system. A module is a grouping of packages, like a package is a grouping of classes. A module may explicitly export some of its packages via export declarations in its module declaration (contained in file named module-info.java). 

--- 

### In public classes, use accessor methods, not public fields
-  If a class is accessible outside its package, provide accessor methods to preserve the flexibility to change the class’s internal representation.-  if a class is package-private or is a private nested class, there is nothing inherently wrong with exposing its data fields—assuming they do an adequate job of describing the abstraction provided by the class.

### Minimize mutability
- An immutable class is simply a class whose instances cannot be modified.
- They are less prone to error and are more secure. To make a class immutable, follow these five rules: 
    - Don’t provide methods that modify the object’s state (Mutators/Setters) 
    - Ensure that the class can’t be extended. (Preventing subclassing is generally accomplished by making the class final, but there is an alternative that we’ll discuss later.)
    - Make all fields final. (Expresses intention, ensures correct behavior if a reference to a newly created instance is passed from one thread to another without synchronization, as spelled out in the memory model [JLS, 17.5; Goetz06, 16])
    - Make all fields private. While it is technically permissible for immutable classes to have public final fields containing primitive values or references to immutable objects, it is not recommended because it precludes changing the internal representation in a later release
    - Ensure exclusive access to any mutable components. Never initialize such a field to a client-provided object reference or return the field from an accessor. Make defensive copies 
    
    ```java 
    // Example on class; Immutable complex number class
    public final class Complex {
        private final double re;
        private final double im;
        public Complex(double re, double im) {
            this.re = re;
            this.im = im;
        }
        public double realPart() { return re; }
        public double imaginaryPart() { return im; }
        public Complex plus(Complex c) { return new Complex(re + c.re, im + c.im); }
        public Complex minus(Complex c) { return new Complex(re - c.re, im - c.im); }
        public Complex times(Complex c) { return new Complex(re * c.re - im * c.im, re * c.im + im * c.re); }
        public Complex dividedBy(Complex c) {
            double tmp = c.re * c.re + c.im * c.im;
            return new Complex((re * c.re + im * c.im) / tmp, (im * c.re - re * c.im) / tmp);
        }
        @Override public boolean equals(Object o) {
            if (o == this) return true;
            if (!(o instanceof Complex)) return false;
            Complex c = (Complex) o;
            return Double.compare(c.re, re) == 0 && Double.compare(c.im, im) == 0;
        }
        @Override public int hashCode() { return 31 * Double.hashCode(re) + Double.hashCode(im); }
        @Override public String toString() { return "(" + re + " + " + im + "i)"; }
    }   
    ```
- Immutable objects are inherently thread-safe; they require no synchronization. 
    - They cannot be corrupted by multiple threads accessing them concurrently. This is far and away the easiest approach to achieve thread safety. Since no thread can ever observe any effect of another thread on an immutable object, immutable objects can be shared freely. 
    - Immutable classes should therefore encourage clients to reuse existing instances wherever possible. One easy way to do this is to provide public static final constants for commonly used values. For example, the Complex class might provide these constants:

        ```java
        public static final Complex ZERO = new Complex(0, 0);
        public static final Complex ONE = new Complex(1, 0);
        public static final Complex I = new Complex(0, 1);
        ```
    - You never have to make any copies at all because the copies would be forever equivalent to the originals. Therefore, you need not and should not provide a clone method or copy constructor on an immutable class.
    - Not only can you share immutable objects, but they can share their internals. 
        - The BigInteger class uses a sign-magnitude representation internally. The sign is represented by an int, and the magnitude is represented by an int array. The negate method produces a new BigInteger of like magnitude and opposite sign. It does not need to copy the array even though it is mutable; the newly created BigInteger points to the same internal array as the original.

    - Immutable objects provide failure atomicity for free. Their state never changes, so there is no possibility of a temporary inconsistency.
- The major disadvantage of immutable classes is that they require a separate object for each distinct value. Creating these objects can be costly, especially if they are large. For example, suppose that you have a million-bit BigInteger and you want to change its low-order bit:
    ```BigInteger moby = ...;
    moby = moby.flipBit(0);``` The flipBit method creates a new BigInteger instance, also a million bits long, that differs from the original in only one bit.

- If a multistep operation is provided as a primitive, the immutable class does not have to create a separate object at each step. Internally, the immutable class can be arbitrarily clever. For example, BigInteger has a package-private mutable “companion class” that it uses to speed up multistep operations such as modular exponentiation. It is much harder to use the mutable companion class than to use BigInteger, for all of the reasons outlined earlier. Luckily, you don’t have to use it: the implementors of BigInteger did the hard work for you. The package-private mutable companion class approach works fine if you can accurately predict which complex operations clients will want to perform on your immutable class. If not, then your best bet is to provide a public mutable companion class. The main example of this approach in the Java platform libraries is the String class, whose mutable companion is StringBuilder (and its obsolete predecessor, StringBuffer).

- Recall that to guarantee immutability, a class must not permit itself to be subclassed. This can be done by making the class final, but there is another, more flexible alternative. Instead of making an immutable class final, you can make all of its constructors private or package-private and add public static factories in place of the public constructors. 

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
        // ... Remainder unchanged
    }
    ```
---

### Favor composition over inheritance
- **The problems discussed here do not apply to interface inheritance**

- Inheritance is a powerful way to achieve code reuse, but it is not always the best tool for the job. Used inappropriately, it leads to fragile software. It is safe to use inheritance within a package, where the subclass and the superclass implementations are under the control of the same programmers. It is also safe to use inheritance when extending classes specifically designed and documented for extension. Inheriting from ordinary concrete classes across package boundaries, however, is dangerous. 

- A subclass depends on the implementation details of its superclass for its proper function. The superclass’s implementation may change from release to release, and if it does, the subclass may break, even though its code has not been touched. As a consequence, a subclass must evolve in tandem with its superclass, unless the superclass’s authors have designed and documented it specifically for the purpose of being extended.

- A related cause of fragility in subclasses is that their superclass can acquire new methods in subsequent releases. 
    - Suppose a program depends for its security on the fact that all elements inserted into some collection satisfy some predicate. This can be guaranteed by subclassing the collection and overriding each method capable of adding an element to ensure that the predicate is satisfied before adding the element. This works fine until a new method capable of inserting an element is added to the superclass in a subsequent release. Once this happens, it becomes possible to add an “illegal” element merely by invoking the new method, which is not overridden in the subclass.

-  Instead of extending an existing class, give your new class a private field that references an instance of the existing class. This design is called composition because the existing class becomes a component of the new one.

    - Each instance method in the new class invokes the corresponding method on the contained instance of the existing class and returns the results. This is known as forwarding, and the methods in the new class are known as forwarding methods. The resulting class will be rock solid, with no dependencies on the implementation details of the existing class. 

- An example on using composition-and-forwarding

    ```java
        // Decorator pattern 
        // Wrapper class - uses composition in place of inheritance
        public class InstrumentedSet<E> extends ForwardingSet<E> {
            private int addCount = 0;
            public InstrumentedSet(Set<E> s) {
                super(s);
            }
            @Override public boolean add(E e) {
                addCount++;
                return super.add(e);
            }
            @Override public boolean addAll(Collection<? extends E> c) {
                addCount += c.size();
                return super.addAll(c);
            }
            public int getAddCount() {
                return addCount;
            }
        }
        // Reusable forwarding class
        public class ForwardingSet<E> implements Set<E> {
            private final Set<E> s;
            public ForwardingSet(Set<E> s) { this.s = s; }
            public void clear() { s.clear(); }
            public boolean contains(Object o) { return s.contains(o); }
            public boolean isEmpty() { return s.isEmpty(); }
            public int size() { return s.size(); }
            public Iterator<E> iterator() { return s.iterator(); }
            public boolean add(E e) { return s.add(e); }
            public boolean remove(Object o) { return s.remove(o); }
            public boolean containsAll(Collection<?> c)
            { return s.containsAll(c); }
            public boolean addAll(Collection<? extends E> c)
            { return s.addAll(c); }
            public boolean removeAll(Collection<?> c)
            { return s.removeAll(c); }
            public boolean retainAll(Collection<?> c)
            { return s.retainAll(c); }
            public Object[] toArray() { return s.toArray(); }
            public <T> T[] toArray(T[] a) { return s.toArray(a); }
            @Override public boolean equals(Object o)
            { return s.equals(o); }
            @Override public int hashCode() { return s.hashCode(); }
            @Override public String toString() { return s.toString(); }
        }
    ```
    
    - The disadvantages of wrapper classes are few. One caveat is that wrapper classes are not suited for use in callback frameworks, wherein objects pass self-references to other objects for subsequent invocations. 

- Inheritance is appropriate only in circumstances where the subclass really is a subtype of the superclass. In other words, a class B should extend a class A only if an “is-a” relationship exists between the two classes.
    - Always ask your self is every B really an A? 

- Google's Guava provides forwarding classes for all of the collection interfaces

- To summarize, inheritance is powerful, but it is problematic because it violates encapsulation. It is appropriate only when a genuine subtype relationship exists between the subclass and the superclass. Even then, inheritance may lead to fragility if the subclass is in a different package from the superclass and the superclass is not designed for inheritance. To avoid this fragility, use composition and forwarding instead of inheritance, especially if an appropriate interface to implement a wrapper class exists. Not only are wrapper classes more robust than subclasses, they are also more powerful.

--- 






























