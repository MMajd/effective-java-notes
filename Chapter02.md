# Effective Java - Chapter 2 Notes
### Use static method factories when possible, it has many advantages over constructors 
- Unlike constructors they 
    - Have names. 
    - Not required to create a new object each time they’re invoked.
    - Can return an object of any subtype of their return type.
    - Returned object can vary from call to call as a function of the input parameters.


### Perfer using Builder Pattern if there're many class constructor parameters 
- Having multiple constructor can lead to many hard to discover bugs
- JavaBeans can be used when facing many parameters situation, but eliminate class immutability while builder pattern preserve it  
- Static factory method not suitable as the main concern here is having many parameters so the problem remains
	
	```java
	// Builder pattern for class hierarchies
	public abstract class Pizza {
		public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
		final Set<Topping> toppings;
		// recursive Type parameter to allow method chaining to work properly in subclasses
		abstract static class Builder<T extends Builder<T>> {  
			EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
			public T addTopping(Topping topping) {
				toppings.add(Objects.requireNonNull(topping));
				return self();
			}
			abstract Pizza build();
		// Subclasses must override this method to return "this"
			protected abstract T self();
		}
		Pizza(Builder<?> builder) {
			toppings = builder.toppings.clone(); // See Item 50
		}
	}
	```


### Use private constructors for non-insatiable class (Ex: Math class, Collections class)
- Private constructors eliminate the possibility of sub-classed, 
  while using abstract class give the impression of being suitable for inerhitance


### Singleton
- Use static method factory as preference for more rebust and future prove code
  When singleton implements serializable, will need to implement readResolve, and declare all instances as transient 
  Method can be used as a Supplier<SingeltonType> 

	```java
	// readResolve method to preserve singleton property
	private Object readResolve() {
		// Return the one true Elvis and let the garbage collector
		// take care of the Elvis impersonator.
		return INSTANCE;
	}
	```

- Use enum with one instance, and you will get, free serilzation, free guarantee against multiple instantiation even against 
  serialization reflection attacks (AccessibleObject.setAccessible), but you will lose inheritance, though an enum can implement interface
  
	```java
	public enum Singleton {
		INSTANCE; 
		public void doSomething() { ... } 
	}
	```

### Memory Leaks 
- USE HEAP PROFILER TO DETECT MEMORY LEAKS 
- Use WeakHashMap, and LinkedHashMap (removeEldestEntry) for caching 
- You can use java.lang.ref directly for more sophisticated caching
- Be alert & careful when creating an object that manages its memory, this is a source of memory leaks
- Other sources of memory leaks are listeners and callbacks, you should always use weak reference to them, and be alert

### Avoid finalizer & cleaners 
- As of Java 9 finalizers are deprecated, replaced by Cleaners 
- Unpredictable, slow, generally unnecessary 
- One shortcoming of finalizers and cleaners is that there is no guarantee they’ll
  be executed promptly [JLS, 12.6]. It can take arbitrarily long between the time
  that an object becomes unreachable and the time its finalizer or cleaner runs.
  This means that you should never do anything time-critical in a finalizer or
  cleaner. 
- Not only does the specification provide no guarantee that finalizers or cleaners
  will run promptly; it provides no guarantee that they’ll run at all.
-  you should never depend on a finalizer or cleaner to update persistent state. 

### Prefer try-with-resource to try-finally
- try-finally can get ugly when having more than one resouce and can cause wrong close code, also can cause tangled exceptin and can lead to the last exception obliterates the ealier ones     
- Resource used inside try-with-resouce must implement AutoClosable

	```java
	// try-finally is ugly when used with more than one resource!
	static void copy(String src, String dst) throws IOException {
		InputStream in = new FileInputStream(src);
		try {
			OutputStream out = new FileOutputStream(dst);
			try {
				byte[] buf = new byte[BUFFER_SIZE];
				int n;
				while ((n = in.read(buf)) >= 0) out.write(buf, 0, n);
			} 
			finally {
				out.close();
			}
		} 
		finally {
			in.close();
		}
	}

	// try-with-resources on multiple resources - short and sweet
	static void copy(String src, String dst) throws IOException {
		try (InputStream in = new FileInputStream(src);
			OutputStream out = new FileOutputStream(dst)) {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0) out.write(buf, 0, n);
		}
		catch(IOException ioex) { throws ioex; } 
	}
	```
- Not only are the try-with-resources versions shorter and more readable than the
originals, but they provide far better diagnostics. Consider the
firstLineOfFile method. If exceptions are thrown by both the readLine
call and the (invisible) close, the latter exception is suppressed in favor of the
former. In fact, multiple exceptions may be suppressed in order to preserve the
exception that you actually want to see. These suppressed exceptions are not
merely discarded; they are printed in the stack trace with a notation saying that
they were suppressed. You can also access them programmatically with the
getSuppressed method, which was added to Throwable in Java 7.

