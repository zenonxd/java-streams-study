# About

Brief study about Stream with java.

The ideia of this study is to simplify data processing with Streams :)

# Example

Imagine a simple program that we create a list, insert some itens inside of it and want to print a specific item.

Generally, we are going to use a for, right? Like this:

```java
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        var items = new ArrayList<<String>();
        items.add("apple");
        items.add("banana");
        items.add("orange");
        
        for (String item : items) {
            if (item.startsWith("a")) {
                System.out.println(item.toUpperCase());
            }
        }
    }
}
```

This type of code is super valid, but in the real word can be seen as "verbose" or... too much code. So we can use another
way, **streams**!

The ideia is to "transform" this list in a stream.

# What is streams?

Streams represent a chain of elements (or chain of data).

# Advantages

Streams already have aggregate operations (like filters, ordering, transformation). 

Another thing is that streams usually don't result in a lot of code! So, how to change the code above to a stream?  We
are going to use something called "**declarative programming**", focusing on what **we want to do**, instead on HOW to do it.

```java
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        var items = new ArrayList<<String>();
        items.add("apple");
        items.add("banana");
        items.add("orange");
        
        for (String item : items) {
            if (item.startsWith("a")) {
                System.out.println(item.toUpperCase());
            }
        }
        
        items.stream()
                //only the itens that starts with "a" are going to pass to the new
                //operation
                .filter(item -> item.startsWith("a"))
                //the map receives the filtered item, and transform it to upperCase
                .map(String::toUpperCase)
                //prints each item, one by one
                .forEach(System.out::println);
    }
}
```

# Types of stream operations

Streams have different types of operations.

**Intermediate operations** - They transform the stream (filter and map are intermediate op)


**Terminal operations** - They close the stream, if we tried to do a filter after the for each, wouldn't be possible.

## Types of terminal operations

### forEach

We already saw it on the code above.

### collect

Transform the stream back in a collection, and we could store the result in a variable, being able to print it.

```java
        var itemsModified = items.stream()
                .filter(item -> item.startsWith("a"))
                .map(String::toUpperCase)
                .collect(Collector.toList());
        System.out.println(itemsModified);
```

### reduce

Reduces the list (streams) of elements in a UNIQUE value! It's very used on cumulative arithmetic operations.

Reduce needs to have two arguments: accumulator and the operation itself, check it:

```java
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        var items = new ArrayList << String > ();
        items.add("apple");
        items.add("banana");
        items.add("orange");


        var numbers = List.of(1, 2, 3, 4, 5);
        
        //we start at zero and we make a sum with every value on the list
        //into a unique value
        numbers.stream().reduce(0, Integer::sum)
        System.out.println(sum);
    };
}
```

# parallelStream()

It's a functionality from the streams API, that allow us to process elements from a collection in parallel, using different
threads.

It can be useful to accelerate intensive operations with big collections. But, for this to work, the collections have
to be independent (without colateral effects) and the parallelism has to be worth it.

## How does it work?

### Parallel execution

A ``parallelStream()`` divides the collection in different parts, and process each part in a different thread, using
Fork/Join Framework.

### Load balancing

He tries to balance the processing between available threads, using thread pool with Fork/Join Framework.

### Independence

To allow correct results, the operations that are carried out wit the stream, needs to be "without colateral effects"
and needs to be "associate". Basically: **the execution order can't affect the result**.

### Difference between stream() and parallelStream()

| stream()                                                           | parallelStream()                                              |
|--------------------------------------------------------------------|---------------------------------------------------------------|
| Executes the operations in sequencial way,<br/>in a unique thread. | Executes operations in parallel, using multiple<br/>threads.  |
| More adequate to light tasks or low complexity.                    | Better to tasks that are intensive or big collections.        |
| Execution order is predictable.                                    | Execution order can be unpredictable (unless it's specified). |
| Uses less system resource.                                         | Can consume more resources from the system, like CPU.         |

### Example

Imagine that you want to calculate the square from numbers from a big list.

#### With stream

```java
import java.util.List;

public class StreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        numbers.stream()
               .map(n -> n * n)
               .forEach(System.out::println);
    }
}
```

##### Output

1
4
9
16
25
36
49
64
81
100

#### With parallelStream

```java
import java.util.List;

public class ParallelStreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        numbers.parallelStream()
               .map(n -> n * n)
               .forEach(System.out::println); // A ordem pode variar!
    }
}
```

##### Output

36
16
4
64
49
81
9
25
1
100

**Note:** this order can be random!

# When to use parallelStream?

## Intensive tasks in CPU

When the operations that are carried out in each element is computationally expensive (complex calculations, image 
processing, etc.).

## Big collections

When there's too many elements to process, parallel cost can be justified.

## Without dependencies

When operations in different elements don't depend on one another or don't have side effect.

## System with multiple cores 

It's more Effective on machines with multicore CPU, where parallelism can be taken advantage of.

# Care when using parallelStream

## Tasks with side effects

In this case, we have an empty list that is going to receive the numbers from ``numbers``. But, since we are using
parallel, a lot of threads are going to try to add the number simultaneously, **the result will be unpredictable**.

```java
// WRONG: Modificating state when sharing parallel
List<Integer> numbers = List.of(1, 2, 3, 4, 5);
List<Integer> results = new ArrayList<>();

numbers.parallelStream().forEach(results::add); // can cause incosistences
```

### Solution

Use thread-safe collection or avoid the change of state.

#### Thread-safe

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);
List<Integer> results = Collections.synchronizedList(new ArrayList<>());

numbers.parallelStream().forEach(results::add); // Correto
System.out.println(results);
```

#### Avoid the change of state

Work if data that is immutable or create new collections.

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);
List<Integer> results = numbers.parallelStream()
                               .map(n -> n * 2) // Operação sem modificar estado externo
                               .toList(); // Cria nova lista
System.out.println(results);
```

## Parallelism cost

Parallelism it's not always fast. The cost of the creation and management of threads can overcome the gains to small
collections or light operations.

## Order of results

The order is not guarantied , unless you use ordered collections or specific methods, like ``forEachOrdered()``.

```java
// Keeping the order
numbers.parallelStream()
       .map(n -> n * n)
       .forEachOrdered(System.out::println); // order will be preserved
```

## Deadlocks and competition

If you are using shared resources or locks in your logic, parallelism can cause deadlocks.

### Deadlocks

Occurs when two threads gets blocked while storing resources that are never going to be released. It's an error that
paralises the execution.

```java
List<Integer> listA = new ArrayList<>();
List<Integer> listB = new ArrayList<>();

synchronized (listA) {
    synchronized (listB) {
        // simultaneous operations on the List
    }
}

// the other thread does the opposite
synchronized (listB) {
    synchronized (listA) {
        // simultaneous operations on the List
    }
}
```

In this case, one thread blocks the ``listA`` and waits for ``listB``, while the other thread blocks ``listB`` and
waits for ``listA``. None of the threads goes forward.
