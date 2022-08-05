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

---
### Always override *hashCode* when overriding *equals*
- You must override hashCode when overriding equals, if you don't your class will violate the general contract for **hashCode** and your code will fail to function properly in the collections classes such as HashMap/HashSet 
- The general contract for ***hashCode*** as follow 
	- hashCode must return same value consistently during an execution of an application, provided that no information used information used in *equals* is changed
	- If two objects are equals according to the ***equals(Object)*** method, then calling ***hashCode*** must result in the same integer value
	- If two objects are unequal according to the ***equals(Object)*** method, it's  not required that calling ***hashCode*** on each of the objects must produce distinct results.
		* A good hash function tends to produce unequal hash codes for unequal instances
- The key provision that is violated when you fail to override hashCode is
the second one: equal objects must have equal hash codes. Two distinct
instances may be logically equal according to a class’s equals method, but to
Object’s hashCode method, they’re just two objects with nothing much in
common. Therefore, Object’s hashCode method returns two seemingly
random numbers instead of two equal numbers as required by the contract.
- **Classes	such as HashMap uses an optimization that it caches hash codes and do hashCode comparsion first if hashCodes are different then it doesn't bother to do an equailty check**
- A simple recipe to implement hashCode 
	* Declare int result and initialize it to hash code c for the first significant field in your object (*Significant field is the field the affects equals comparsion*)
	* For every remaining field compute an int c for the field: 
		1. If the field is primitive use Type.hashCode(value), where Type is the boxed primitive
		2. If the field is an object reference ans class's equals method compares the field by recursively invoking eqausl, recursively invoke hashCode on the field, if a more complex comparsion is required, compute 'canonical representation' for this field  and invoke 	the hashCode, if the field is null use 0 
		3. if the field is an array of significant use Arrays.hashCode, if fields are not signifcant elements use constant perferabley not 0 
	* Combine hash code c computed in previous step into result as follow, result = 31*result + c
	* Return the result 
	* Exclude any fields not used in the equals comparions

	```java
	// An example for a typical hashCode method
	@Override public int hashCode() {
		int result = Short.hashCode(areaCode);
		result = 31 * result + Short.hashCode(prefix);
		result = 31 * result + Short.hashCode(lineNum);
		return result;
	}
	```

- Nice property of multiplying by 31 is the multiplication can be replaced by left shift and substraction [31*i = (i << 5) - i]	
- If a class is immutable and the cost of computing the hash code is significant,
you might consider caching the hash code in the object rather than recalculating
it each time it is requested. 
- Don’t provide a detailed specification for the value returned by
hashCode, so clients can’t reasonably depend on it; this gives you the
flexibility to change it.
- Use AutoValue to generate good *hashCode*

---

### Always override toString
- The general contract for toString says that the
returned string should be 
	- "a concise but informative representation that is easy for a person to read."
	- "It is recommended that all subclasses override this method."
- Providing a good toString implementation makes your
class much more pleasant to use and makes systems using the class easier to
debug.
- It makes no sense to write a toString method in a static utility class (Item
4). Nor should you write a toString method in most enum types (Item 34)
because Java provides a perfectly good one for you. Y
- Use AutoValue to generate *toString*
- The take home advice, override Object’s toString implementation in every
instantiable class you write, unless a superclass has already done so. It makes
classes much more pleasant to use and aids in debugging. The toString
method should return a concise, useful description of the object, in an
aesthetically pleasing format.

---

### Override clone judiciously
- The Cloneable interface was intended as a mixin interface for
classes to advertise that they permit cloning. Unfortunately, it fails to serve this
purpose. Its primary flaw is that it lacks a clone method, and Object’s
clone method is protected. You cannot, without resorting to reflection, invoke clone on an object merely because it implements Cloneable.

- Even a reflective invocation may fail, because there is no guarantee that the
object has an accessible clone method.

- What does Cloneable do, given that it contains no methods? It
determines the behavior of Object’s protected clone implementation: if a
class implements Cloneable, Object’s clone method returns a field-by-
field copy of the object; otherwise it throws CloneNotSupportedException. This is a highly atypical use of
interfaces and not one to be emulated. Normally, implementing an interface says
something about what a class can do for its clients. In this case, it modifies the
behavior of a protected method on a superclass.

- The general contract for the clone method is weak. 
	- Creates and returns a copy of this object. The precise meaning of “copy” may
depend on the class of the object. The general intent is that, for any object x,
the expression
		- x.clone() != x will be true
		- x.clone().getClass() == x.getClass() will be true, not an absolute requirement.
		- x.clone().equals(x) will be true, not an absolute requirement.
		- By convention, the object returned by this method should be obtained by calling super.clone. If a class and all of its superclasses (except Object) obey this convention, it will be the case that Click here to view code image x.clone().getClass() == x.getClass().
	- Note that immutable classes should never provide a clone method because it would merely encourage wasteful copying. 

	```java
	// Clone method for class with no references to mutable state
	@Override public PhoneNumber clone() {
		try {
			/* Java supports covariant return types. In other words, 
			 * an overriding method’s return type can be a subclass of the overridden method’s return type. */ 
			return (PhoneNumber) super.clone();
		} catch (CloneNotSupportedException e) {
			throw new AssertionError(); // Can't happen
		}
	}
	```

- If an object contains fields that refer to mutable objects, the simple clone implementation shown earlier can be disastrous.

	```java
	public class Stack implements Clonable {
		private Object[] elements;
		private int size = 0;
		private static final int DEFAULT_INITIAL_CAPACITY = 16;
		public Stack() { this.elements = new Object[DEFAULT_INITIAL_CAPACITY]; }
		// Rest of stack code...
		// THIS IS BAD 
		@Override public stack clone() {
			//..try
			return (Stack) super.clone(); 
			//..catch 
		}  
	}
	```
-  If the clone method merely returns super.clone(), the resulting Stack instance will have the correct
value in its size field, but its elements field will refer to the same array as the original Stack instance. Modifying the original will destroy the invariants in the clone and vice versa.

- In effect, the clone method functions as a constructor; you must ensure that it does no harm to the original object and that it properly establishes invariants on the clone.
- The Cloneable architecture is incompatible with normal use of ***final*** fields referring to mutable objects,  except in cases where the mutable objects may be safely
shared between an object and its clone. In order to make a class cloneable, it may be necessary to remove **final** modifiers from some fields.

```java 
/* In order for the clone method
on Stack to work properly, it must copy the internals of the stack. The easiest
way to do this is to call clone recursively on the elements array:
Click here to view code image*/ 
// Clone method for class with references to mutable state
@Override public Stack clone() {
	try {
		Stack result = (Stack) super.clone();
		// apply clone recursively on elements array 
		// Calling clone on an array returns an array whose runtime and compile-time types are identical to those of the array being cloned.
		result.elements = elements.clone();
		return result;
	} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	}
}
// Deep-copying incase of reference objects contained inside the array 
@Override public HashTable clone() {
	try {
		HashTable result = (HashTable) super.clone();
		result.buckets = new Entry[buckets.length];
		for (int i = 0; i < buckets.length; i++)
			if (buckets[i] != null) result.buckets[i] = buckets[i].deepCopy();
		return result;
	} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	}
}
```


- Like a constructor, a clone method must never invoke an overridable
method on the clone under construction
	- If clone invokes a method that is overridden in a subclass, this method will execute before the subclass has
had a chance to fix its state in the clone, quite possibly leading to corruption in
the clone and the original. 

- You have two choices when designing a class for inheritance, but
whichever one you choose, the class should not implement Cloneable. You
may choose to mimic the behavior of Object by implementing a properly
functioning protected clone method that is declared to throw
CloneNotSupportedException. 

- to prevent subclasses from implementing one, by providing the following degenerate clone implementation:
	- Override protected Object's class method and make it final
 
	```java
	// clone method for extendable class not supporting Cloneable
	@Override
	protected final Object clone() throws CloneNotSupportedException {
		throw new CloneNotSupportedException();
	}
	```

- Given all the problems associated with Cloneable, new interfaces should not extend it, and new extendable classes should not implement it. While it’s less harmful for final classes to implement Cloneable, this should be viewed as a performance optimization, reserved for the rare cases where it is justified
	-  A notable exception to this rule is arrays, which are best copied with the clone method.

- **A better approach to object copying is to provide a copy constructor or copy factory**
	- A copy constructor is simply a constructor that takes a single argument whose type is the class containing the constructor

		```java
		// Copy constructor
		public Yum(Yum yum) { ... };

		//  Copy factory
		public static Yum newInstance(Yum yum) { ... };
		```
		- The copy constructor approach and its static factory variant have many advantages over Cloneable/clone: 
			- they don’t rely on a risk-prone extralinguistic object creation mechanism; 
			- they don’t demand unenforceable adherence to thinly documented conventions; 
			- they don’t conflict with the proper use of final fields; 
			- they don’t throw unnecessary checked exceptions; 
			- and they don’t require casts.
			- Furthermore, a copy constructor or factory can take an argument whose type is an interface implemented by the class. 

		- Interface-based copy constructors and factories, more properly known as conversion constructors and conversion factories, allow the client to choose the implementation type of the copy rather than forcing the client to accept the implementation type of the original. For example, suppose you have a HashSet, s, and you want to copy it as a TreeSet. The clone method can’t offer this f
	
- Most of the collections Classes implement a conversion constractor that accepts either a Collection Interface or a Map

---

### Consider implementing Comparable
- *compareTo* is similar in character to Object’s equals method, except that it permits order comparisons in addition to simple equality comparisons,
- By implementing Comparable, a class indicates that its instances have a natural ordering. 
	-  Sorting an array of objects that implement *Comparable* is as simple as this: 
		```Arrays.sort(a);```
- By implementing Comparable, you allow your class to interoperate with all of the many generic algorithms and collection implementations that depend on this interface. 
- The generalcontract of *compareTo* method is similar to that of *equals*
	- Compares this object with the specified object for order. 
		* Returns a negative integer as this object is less than the specified object
		* Zero if it's equal to the specified object, 
		* Positive integer if i's greater than the specified object 
		* Throws ClassCastException if the specified object’s type prevents it from being compared to this object.

	- In the following description, the notation ***sgn(expression)*** designates the mathematical **signum** function, which is defined to **return -1, 0, or 1, according to whether the value of expression is negative, zero, or positive**.
		* The implementor must ensure that ```sgn(x.compareTo(y)) == - sgn(y. compareTo(x))``` for all x and y. (This implies that ```x.compareTo(y)``` must throw an exception if and only if ```y.compareTo(x)``` throws an exception.)
		* The implementor must also ensure that the relation is transitive: ```(x.compareTo(y) > 0 && y.compareTo(z) > 0)``` implies ```x.compareTo(z) > 0```.
		* Finally, the implementor must ensure that ```x.compareTo(y) == 0``` implies that ```sgn(x.compareTo(z)) == sgn(y.compareTo(z))```, for all z.
		* It is strongly recommended, but not required, that ```(x.compareTo(y) == 0) == (x.equals(y))```. **Generally speaking, any class that implements the Comparable interface and violates this condition should clearly indicate this fact**. The recommended language is “Note: This class has a natural ordering that is inconsistent with equals.”

- One consequence of these three provisions is that the equality test imposed by a compareTo method must obey the same restrictions imposed by the equals cotract: reflexivity, symmetry, and transitivity. ***Therefore, the same caveat applies: there is no way to extend an instantiable class with a new value component while preserving the compareTo contract***, unless you are willing to forgo the benefits of object-oriented abstraction
-  If you want to add a value component to a class that implements Comparable, don’t extend it; write an unrelated class containing an instance of the first class. Then provide a “view” method that returns the contained instance. This frees you to implement whatever compareTo method you like on the containing class, while allowing its client to view an instance of the containing class as an instance of the contained class when needed.
- Classes that depend on comparison include the sorted collections TreeSet and TreeMap and the utility classes Collections and Arrays, which contain searching and sorting algorithms.
- If violate the last provision that says its recommend that ```(x.compareTo(y) == 0) == x.equals(y)``` you will get a different behavior in sorted classes than that used in other collections class (ex: TreeSet vs HashSet) 
	- For example, consider the BigDecimal class, whose compareTo method is inconsistent with equals. If you create an empty HashSet instance and then add new BigDecimal("1.0") and new BigDecimal("1.00"), the set will contain two elements because the two BigDecimal instances added to the set are unequal when compared using the equals method. If, however, you perform the same procedure using a TreeSet instead of a HashSet, the set will contain only one element because the two BigDecimal instances are equal when compared using the compareTo method. 

- Writing a compareTo method is similar to writing an equals method, but there are a few key differences. Because the Comparable interface is parameterized, the compareTo method is statically typed, so you don’t need to type check or cast its argument. If the argument is of the wrong type, the invocation won’t even compile. If the argument is null, the invocation should throw a NullPointer-Exception, and it will, as soon as the method attempts to access its members.
- In a compareTo method, fields are compared for order rather than equality. To compare object reference fields, invoke the compareTo method recursively. If a field does not implement Comparable or you need a nonstandard ordering, use a Comparator instead.
	- An example of using comparator class that already exist	
	
	```java
	// Single-field Comparable with object reference field
	public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
		public int compareTo(CaseInsensitiveString cis) {
			return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
		}
	}
	```
- If a class has multiple significant fields, the order in which you compare them is critical. Start with the most significant field and work your way down. If a comparison results in anything other than zero (which represents equality), you’re done; just return the result. If the most significant field is equal, compare the next-most-significant field, and so on, until you find an unequal field or compare the least significant field. 
	
	```java
	// Multiple-field Comparable with primitive fields
	@Override public int compareTo(PhoneNumber pn) {
		int result = Short.compare(areaCode, pn.areaCode);
		if (result == 0) {
			result = Short.compare(prefix, pn.prefix);
			if (result == 0)
				result = Short.compare(lineNum, pn.lineNum);
		}
		return result;
	}
	```

- In Java 8, the Comparator interface was outfitted with a set of comparator construction methods, which enable fluent construction of comparators.

	```java
	// Comparable with comparator construction methods
	private static final Comparator<PhoneNumber> COMPARATOR =
		// here we help Java type inference but after the first call there's no need to do that 
		comparingInt((PhoneNumber pn) -> pn.areaCode) 
			.thenComparingInt(pn -> pn.prefix)
			.thenComparingInt(pn -> pn.lineNum);

		@Override public int compareTo(PhoneNumber pn) {
			return COMPARATOR.compare(this, pn);
		}
	```

- Difference based comparators violates transitivty
	*  

	```java 
	// BROKEN difference-based comparator - violates transitivity!
	static Comparator<Object> hashCodeOrder = new Comparator<>() {
		public int compare(Object o1, Object o2) {
		return o1.hashCode() - o2.hashCode();
		}
	};
	```
	* **Use comparator based on static compare method or on Comparator construction method** 

		```java
		// Comparator based on static compare method
		static Comparator<Object> hashCodeOrder = new Comparator<>() {
		public int compare(Object o1, Object o2) {
			return Integer.compare(o1.hashCode(), o2.hashCode());
		}
		};

		// Comparator based on Comparator construction method
		static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
		``` 

In summary, whenever you implement a value class that has a sensible
ordering, you should have the class implement the Comparable interface so
that its instances can be easily sorted, searched, and used in comparison-based
collections. When comparing field values in the implementations of the
compareTo methods, avoid the use of the ***{<, >, -}*** operators. Instead, use the
static compare methods in the boxed primitive classes or the comparator
construction methods in the Comparator interface.

---



























 