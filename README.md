## Why Javaslang? [![Build Status](https://travis-ci.org/rocketscience-projects/javaslang.png)](https://travis-ci.org/rocketscience-projects/javaslang)

**Javaslang** is a functional library for Java&trade; 8 and above. With the release of Java 8 we face a new programming paradigm - Java goes functional. Objects are not obsolete - the opposite is true. We've learned from other JVM languages like Scala, that it is a good practice to take the best from both worlds, Objects and Functions. Javaslang adds some API and best practices, to take advantage of Lambdas & Co in the daily programming.

Javaslang makes your code more concise by reducing boilerplate. In particular

* it closes the gap between primitive data types and Objects by providing extensions methods, e.g. to uniform the streaming API. Did you notice for example, that IntStream and Stream have no common super type which provides the methods map, flatMap and filter?
* it adds a clean and functional way to handle different sates in a stateless way. I know, this sounds weird. But a function call can be considered as point in time if it is referential transparent. That means, that state is bound to time. If there is no state, the function call can be substituted with its return value given a set of parameters. Using the type system to project different states to immutable values of a common type and adding some sugar, like monadic operations (read: map, flatMap and filter), which preserve our structure (read: type), we are able to transport the state (read: value of a specific type) back to the caller of a specific (business) function. The state may be mutated on its way but structural properties (provable laws in a math. sense) are preserved.
* as described above, types are needed to create structure-preserving operations. Examples are Option, Try and Either (see below). To evaluate computation results, Javaslang introduces a new Matcher API that is a switch on steroids.

The functions described above are the basis to provide powerful APIs based on Java 8 and Javaslang. This is demonstrated by the javaslang.io API for resource reading and conversion. Future releases of Javaslang will add additional high-level API for text parsing, a functional jdbc layer, etc.

Most libraries, also the popular ones, like spring, apache-commons and google-guava, can be considered as outdated from the perspective of Java 8. Even if new functionality is added to these libraries or they build using Java 8, they still will carry all the clutter of the past. Javaslang is a fresh and lightweight start into the second age of Java. It is no re-implementation of existing API in a new fashion. Javaslang is simple and focused.

## Packages and dependency overview:

```
|                     io                    |
| collection | either | exception | matcher |
|         lang        |       option        |
 - - - - - - - - - - - - - - - - - - - - - - 
|                   java 8                  |
```

## Content

1. Language
    1. Lang - assertions and a better println
    2. Arrays - conversion and bulk operations
    3. Strings - string operations
    4. Runtimes - definite jvm termination
    5. Timers - syntactic sugar for Timer
2. Monads and Matching
    1. Option - null avoidance
    2. Matcher - type and value matching
    3. Try - deferred exception handling
    4. Either - Variety of results
3. Collections
    1. Collections - missing collection functions
    2. Sets - set operations (math.)
4. Input/Output
    1. IO - resource loading and encoding
5. (_scheduled_) Text - a parser framework
6. (_scheduled_) Jdbc - a functional jdbc layer
7. (_scheduled_) Json - another json api
8. (_scheduled_) Xml - missing xml functions

## Option - Avoid use of null

Use Option to represent value which may be undefined. Some represents the a value (which may be null), and None is the placeholder for nothing. Both types implement Option, so that no more NullPointerExceptions should occur.

```java
<T> Option<T> head(List<T> list) {
    if (list == null || list.isEmpty()) {
        return None.instance();
    } else {
        return new Some<>(list.get(0));
    }
}

void test() {

    List<Integer> list = Arrays.asList(3, 2, 1);

    String result = head(list)
        .filter(i -> i < 2)
        .map(el -> el.toString())
        .orElse("nothing");    

}
```

If list is null or first element >= 2, result is "nothing" else the first list element is returned as string.

## Use match expression instead of if/return statements 

Instead of

```java
static Stream<?> getStream(Object object) {
    if (o == null) {
        throw new IllegalArgumentException("object is null");
    }
    final Class<?> type = o.getClass().getComponentType();
    if (type.isPrimitive()) {
        if (boolean.class.isAssignableFrom(type)) {
            return Arrays.stream((boolean[]) o);
        } else if (byte.class.isAssignableFrom(type)) {
            return Arrays.stream((byte[]) o);
        } else if (char.class.isAssignableFrom(type)) {
            return Arrays.stream((char[]) o);
        } else if (double.class.isAssignableFrom(type)) {
            return Arrays.stream((double[]) o);
        } else if (float.class.isAssignableFrom(type)) {
            return Arrays.stream((float[]) o);
        } else if (int.class.isAssignableFrom(type)) {
            return Arrays.stream((int[]) o);
        } else if (long.class.isAssignableFrom(type)) {
            return Arrays.stream((long[]) o);
        } else if (short.class.isAssignableFrom(type)) {
            return Arrays.stream((short[]) o);
        } else {
            throw new IllegalStateException("Unknown primitive type: " + o.getClass());
        }
    } else {
        return Arrays.stream((Object[]) o);
    }
}
```

the match API allows us to write

```java
    static final Matcher<Stream<?>> ARRAY_TO_STREAM_MATCHER = Matcher.<Stream<?>>create()
            .caze((boolean[] a) -> Arrays.stream(a))
            .caze((byte[] a) -> Arrays.stream(a))
            .caze((char[] a) -> Arrays.stream(a))
            .caze((double[] a) -> Arrays.stream(a))
            .caze((float[] a) -> Arrays.stream(a))
            .caze((int[] a) -> Arrays.stream(a))
            .caze((long[] a) -> Arrays.stream(a))
            .caze((short[] a) -> Arrays.stream(a))
            .caze((Object[] a) -> Arrays.stream(a));
    
    static Stream<?> getStream(Object object) {
        return ARRAY_TO_STREAM_MATCHER.apply(object);
    }
```

It is also possible to match values instead of types by passing prototype objects to the caze function:

```java
Matcher matcher = Matchers.caze("Moin", s -> s + " Kiel!");

// and then...
String s = matcher.apply("Moin"); // = "Moin Kiel!"
```

## Write fluent code and reduce exception handling boilerplate

Exception handling adds additional technical boilerplate to our source code. Also Java does not distinguish between Fatal (non-recoverable) and NonFatal (recoverable) exceptions.

Use Try to handle exceptions in a clean way.

```java
Try<byte[]> bytes = IO.loadResource("some/system/resource.txt");

String s = Matchers
    .caze((Success<byte[]> s) -> new String(s))
    .caze((Failure f) -> f.toString())
    .apply(bytes);
```

No try/catch needed here.
