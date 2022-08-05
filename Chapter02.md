### Obey the general contract when overriding *equals* 
- Wrong overriding equals can lead to many errors, the easiest way is to not override equals, this can be possible in the following cases
    - Each instance is inherently unique, this is true for classes such as Thread 
    - There is no need for the class to provide logical equality test, for example, java.util.regex.Pattern haven't implement equality check as there's no obvious need to do so
    - A super-class has already overriden *equals* and the super-class behavior is appropriate for the sub-class, example most Set implementation inhertit their equals from the *AbstractSet*, List implementations from *AbstractList*, and Map implementations from *AbstractMap* 
    - The class is private or package-private, and you are certain that its equals method will never be invoked. 
    
    ```java 
    @Override 
    public boolean equals(Object o) {
        throw new AssertionError(); // Method is never called
    }
    ``` 

- When *equals* class implementation is appropriate? 
    - Class has a notion of logical equality such as Integer, Double, Long boxed class 
      **This is generally the case in value classes**, also it enables Objects to be used as map keys or set elements 
    - One type of value class that doesn't require *equals* overriden is the instance-controlled class (ex: singleton, discussed in ch1) 

- Always adhere to the general contract of the equals method, the contract from the specification of Object: 
    * **Properties of Set *Equivlance classes*, taken from discrete mathemtics**
    - Reflexive: For any non-null reference value x, x.equals(x) must return true
    - Symmetric: For any non-null reference values x and y, x.equals(y) must <=>(if and only if)  y.equals(x) returns true. 
    - Transitive: For any non-null reference values x, y, z, if x.equals(y) returns true and y.equals(z) returns true then x.equals(z) must return true
    - Consistent: For any non-null reference values x and y, multiple invocations of x.equals(y) must always return ture or false, provided that no information used in the comparsion is modified 
    - For any non-null value x, x.equals(null) must always return false 

- There is no way to extend an instantiable class and add a value component while preserving the equals contract, While there is no satisfactory way to extend an instantiable class and add a value component, there is a fine workaround: Follow the advice of Item 18, “Favor composition over inheritance.” Instead of having ColorPoint extend Point, give ColorPoint a private Point field and a public view method that returns the point at the same position as this color point:

```java
// Adds a value component without violating the equals contract
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }
    /** Returns the point-view of this color point. */
    public Point asPoint() { return point; }
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
```

- There are some classes in the Java platform libraries that do extend an
instantiable class and add a value component. For example, java.sql.Timestamp extends java.util.Date and adds a nanoseconds field. The equals implementation for Timestamp does violate symmetry and can cause erratic behavior if Timestamp and Date objects are used in the same collection or are otherwise intermixed. The Timestamp class has a disclaimer cautioning programmers against mixing dates and timestamps. While you won’t get into trouble as long as you keep them separate, there’s nothing to prevent you from mixing them, and the resulting errors can be hard to debug. This behavior of the Timestamp class was a mistake and should not be emulated

- Consistent: do not write an equals method that depends on unreliable resources. It’s extremely difficult to satisfy the consistency requirement if you violate this prohibition. For example, java.net.URL’s equals method relies on comparison of the IP addresses of the hosts associated with the URLs. Translating a host name to an IP address can require network access, and it isn’t guaranteed to yield the same results over time. This can cause the URL equals method to violate the equals contract and has caused problems in practice. The behavior of URL’s equals method was a big mistake and should not be emulated. Unfortunately, it cannot be changed due to compatibility requirements. To avoid this sort of problem, equals methods should perform only deterministic computations on memory-resident objects

- Good practice is to use instanceof operator before doing any casting in *equals* also this instanceof operator cover null case, so no need to write an if statement to handle the non-nullity check

```java 
@Override public boolean equals(Object o) {
    if (!(o instanceof MyType)) return false; // cover the if (o==null) return false; 
    MyType mt = (MyType) o;
    ...
}
```

- Use the == operator to check if the argument is a reference to this object. If so, return true. This is just a performance optimization but one that is worth doing if the comparison is potentially expensive.
- For primitive fields whose type is not float or double, use the == operator for comparisons; for object reference fields, call the equals method recursively; for float fields, use the static Float.compare(float, float) method; and for double fields, use Double.compare(double, double). The special treatment of float and double fields is made necessary by the existence of Float.NaN, -0.0f and the analogous double values; see JLS 15.21.1 or the documentation of Float.equals for details. While you could compare float and double fields with the static methods Float.equals and Double.equals, this would entail autoboxing on every comparison, which would have poor performance. For array fields, apply these guidelines to each element. If every element in an array field is significant, use one of the Arrays.equals methods.
- Some object reference fields may legitimately contain null. To avoid the possibility of a NullPointerException, check such fields for equality using the static method Objects.equals(Object, Object).
- The performance of the equals method may be affected by the order in
which fields are compared. For best performance, you should first compare fields that are more likely to differ, less expensive to compare, or, ideally, both. You must not compare fields that are not part of an object’s logical state, such as lock fields used to synchronize operations. You need not compare derived fields, which can be calculated from “significant fields,” but doing so may improve the performance of the equals method. If a derived field amounts to a summary description of the entire object, comparing this field will save you the expense of comparing the actual data if the comparison fails. For example, suppose you have a Polygon class, and you cache the area. If two polygons have unequal areas, you needn’t bother comparing their edges and vertices.
- Always override hashCode when you override equals
- Don’t substitute another type for Object in the equals declaration, functions uses parameter other than one of type Object class doesn't **override** the equals method, but it **overloads** it 
- Use Google’s open source AutoValue framework, which
automatically generates these methods for you, triggered by a single annotation on the class . In most cases, the methods generated by AutoValue are essentially identical to those you’d write yourself.
