---
layout: post
title: Javascript Array Functions In Java
author: Carlos Molero Mata
description: Java implementation of popular Javascript array manipulation functions.
header-style: text
tags: [java, javascript]
---

Javascript is a programming language known, amongst other things, by its incredibly simple and versatile syntax and the premade functions for some of the data structures it has. On the other hand, **Java has been pointed out as a verbose language which requires _too much boilerplate_ to do anything, but this is not true since long time ago. New Java features (from JDK8 - Present) allow us to write pretty concise and dry code** and this includes the manipulation of arrays.

In this post I want to tell you about how you can manipulate arrays in Java the same way you would do in Javascript.

Ready? Let's go.

## Most popular array functions in Javascript

We can head to the [MDN javascript arrays documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) to check the multiple functions that can be used to manipulate arrays in Javascript. Some of the most popular ones are the following:

- [`Array.prototype.concat()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat)

> The `concat()` method is used to merge two or more arrays. This method does not change the existing arrays, but instead returns a new `array`.

- [`Array.prototype.filter()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

> The `filter()` method creates a shallow copy of a portion of a given array, filtered down to just the elements from the given array that pass the test implemented by the provided function.

- [`Array.prototype.find()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find)

> The `find() `method returns the first element in the provided array that satisfies the provided testing function. If no values satisfy the testing function, `undefined` is returned.

- [`Array.prototype.reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)

> The `reduce()` method executes a user-supplied "reducer" callback function on each element of the array, in order, passing in the return value from the calculation on the preceding element. The final result of running the reducer across all elements of the array is a single value.

- [`Array.prototype.sort()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)

> The `sort()` method sorts the elements of an array in place and returns the reference to the same array, now sorted. The default sort order is ascending, built upon converting the elements into strings, then comparing their sequences of UTF-16 code units values.

- [`Array.prototype.shift()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/shift)

> The `shift()` method removes the first element from an array and returns that removed element. This method changes the length of the array.

- [`Array.prototype.unshift()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/unshift)

> The `unshift()` method adds one or more elements to the beginning of an array and returns the new length of the array.

## Switching to Java

Notice that all the examples shown here are referred to the `ArrayList<T>` data structure.

### Concat

In this case, we can use `addAll()` which is a method present in `List<T>` objects. The logic is simple, we add all the elements of both arrays to a new array and return this one.

```java
class Main {
    public static void main(String[] args) {
        List<Integer> arr1 = new ArrayList<>();
        List<Integer> arr2 = new ArrayList<>();

        arr1.add(1);
        arr1.add(2);
        arr2.add(3);
        arr2.add(4);

        System.out.println(concat(arr1, arr2)); // Prints [1, 2, 3, 4]
    }

    static List<Integer> concat(List<Integer> arr1, List<Integer> arr2) {
        List<Integer> concatenated = new ArrayList<>();
        concatenated.addAll(arr1);
        concatenated.addAll(arr2);
        return concatenated;
    }
}
```

### Filter

Java 8 bringed a lot of new features including the `stream()` method in `Collection` and lambda expressions to write functional code more easily. Thanks to that, filtering an `ArrayList` is as simple as you're gonna see in the example.

Now we have created a new static method called `filterOutOdds`, which will return another array with only the even numbers from our concatenated array. We first turn the array into a `stream` and use the filter method on it, the filter method accepts a `Predicate` which is nothing but a expression that evaluates to `true` or `false` and can be written as a lambda function. Finally, we use the `collect` method and turn the output into another `List`.

```java
class Main {
    public static void main(String[] args) {
        List<Integer> arr1 = new ArrayList<>();
        List<Integer> arr2 = new ArrayList<>();

        arr1.add(1);
        arr1.add(2);
        arr2.add(3);
        arr2.add(4);

        List<Integer> concatenated = concat(arr1, arr2);
        System.out.println(filterOutOdds(concatenated)); // Prints [2, 4]
    }

    static List<Integer> concat(List<Integer> arr1, List<Integer> arr2) {
        List<Integer> concatenated = new ArrayList<>();
        concatenated.addAll(arr1);
        concatenated.addAll(arr2);
        return concatenated;
    }

    static List<Integer> filterOutOdds(List<Integer> arr) {
        return arr.stream().filter((i) -> i % 2 == 0).collect(Collectors.toList());
    }
}
```

### Find

Ok, so what if I want to find only one item of my array? Let's find out.

We can use the same methods but instead of using the `collect()` function we are going to use the `findAny()`. This method will return an `Optional<T>` which describes an item from the stream, at the end we use `orElse(null)` to avoid writing the `.isPresent()` and `.get()` stuff ([<i>See the docs if you don't know about this 'stuff'</i>](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)).

```java
class Main {
    public static void main(String[] args) {
        List<Integer> arr1 = new ArrayList<>();
        List<Integer> arr2 = new ArrayList<>();

        arr1.add(1);
        arr1.add(2);
        arr2.add(3);
        arr2.add(4);

        List<Integer> concatenated = concat(arr1, arr2);
        filterOutOdds(concatenated);
        System.out.println(findNumber(concatenated, 3)); // Prints 3
    }

    static List<Integer> concat(List<Integer> arr1, List<Integer> arr2) {
        List<Integer> concatenated = new ArrayList<>();
        concatenated.addAll(arr1);
        concatenated.addAll(arr2);
        return concatenated;
    }

    static List<Integer> filterOutOdds(List<Integer> arr) {
        return arr.stream().filter((i) -> i % 2 == 0).collect(Collectors.toList());
    }

    static Integer findNumber(List<Integer> arr, Integer n) {
        return arr.stream().filter((i) -> i == n).findAny().orElse(null);
    }
}
```

### Reduce

Let's clean our example before moving on.

`reduce()` is another Javascript function that can be written in Java without much difficulty thanks to streams. In fact, this is as similar in Javascript and Java as it was `filter()`. Don't believe me? see it by yourself:

```java
class Main {
    public static void main(String[] args) {
        List<Integer> arr = Arrays.asList(1, 2, 3, 4, 5);
        System.out.println(arr.stream().reduce(0, (acc, v) -> acc + v)); // 15
        System.out.println(arr.stream().reduce(0, Integer::sum)); // 15
    }
}
```

In this case, the Integer value 0 is the identity. It stores the initial value of the reduction operation and also the default result when the stream of Integer values is empty. Likewise, the lambda expression is the accumulator since it takes the partial sum of Integer values and the next element in the stream.

To make the code even more concise, we can use a method reference instead of a lambda expression: `Integer::sum`. Read more about [Method References](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html).

### Sort

Sort is as similar in Javascript and Java as it was filter. We will use `stream` again but with the method `sort`, this method accepts a `Comparator` which can be passed as a lambda function.

```java
public class Main {
    public static void main(String[] args) {
        List<Integer> arr = Arrays.asList(1, 2, 3, 4, 5);

        List<Integer> reversedArr = arr.stream().sorted((i1, i2) -> i2.compareTo(i1)).collect(Collectors.toList());
        System.out.println(reversedArr); // This will print [5, 4, 3, 2, 1]
    }
}
```

In this example we are reversing our array and collecting the items of the stream with the `collect` method to create a new array. `compareTo` evaluates to `true` or `false` placing the items in the stream depending of the result of the expression.

But, we can do much more, imagine you have a `Person` class and want to sort an array of `Person` objects by the different properties of the model.

The `Person` class:

```java
    class Person {
        private String name;
        private String surnames;
        private int age;

        public Person(String name, String surnames, int age) {
            this.name = name;
            this.surnames = surnames;
            this.age = age;
        }

        @Override
        public String toString() {
            return String.format("Person: [Name=%s, Surnames=%s, Age=%s]", this.name, this.surnames, this.age);
        }

        // Getters ommited for brevity...
    }
```

Let's create an array of `Person` objects with some of the most beloved Game of Thrones characters, including me, and sort them by `name`, `surnames` and `age`:

```java
public class Main {
    public static void main(String[] args) {
        List<Person> persons = new ArrayList<>();
        // Yes, now I'm a Game of Thrones character
        persons.add(new Person("Carlos", "Molero", 28));
        persons.add(new Person("John", "Snow", 27));
        persons.add(new Person("Tywin", "Lannister", 63));
        persons.add(new Person("Ed", "Stark", 44));

        // Sorted by name
        System.out.println("\nSORTED BY NAME:");
        persons.stream().sorted(Comparator.comparing(Person::getName))
                .forEach(System.out::println);

        // Sorted by surnames
        System.out.println("\nSORTED BY SURNAMES:");
        persons.stream().sorted(Comparator.comparing(Person::getSurnames))
                .forEach(System.out::println);

        // Sorted by age
        System.out.println("\nSORTED BY AGE:");
        persons.stream().sorted(Comparator.comparing(Person::getAge))
                .forEach(System.out::println);

    }
}
```

Here we are using a different approach, we can also sort objects like our `Person` class. Instead of a lambda function we can use the static method `comparing()` from the `Comparator` class and pass a method reference to our getter method to make it more concise.

Output:

```
SORTED BY NAME:
Person: [Name=Carlos, Surnames=Molero, Age=28]
Person: [Name=Ed, Surnames=Stark, Age=44]
Person: [Name=John, Surnames=Snow, Age=27]
Person: [Name=Tywin, Surnames=Lannister, Age=63]

SORTED BY SURNAMES:
Person: [Name=Tywin, Surnames=Lannister, Age=63]
Person: [Name=Carlos, Surnames=Molero, Age=28]
Person: [Name=John, Surnames=Snow, Age=27]
Person: [Name=Ed, Surnames=Stark, Age=44]

SORTED BY AGE:
Person: [Name=John, Surnames=Snow, Age=27]
Person: [Name=Carlos, Surnames=Molero, Age=28]
Person: [Name=Ed, Surnames=Stark, Age=44]
Person: [Name=Tywin, Surnames=Lannister, Age=63]
```

As a side note, this operation is also possible with `compareTo`, and it is recommended to use this method if you want to change the sort order to `DESC`. Let's see an example with the `age` property.

```java
public class Main {
    public static void main(String[] args) {
        List<Person> persons = new ArrayList<>();
        // Yes, now I'm a Game of Thrones character
        persons.add(new Person("Carlos", "Molero", 28));
        persons.add(new Person("John", "Snow", 27));
        persons.add(new Person("Tywin", "Lannister", 63));
        persons.add(new Person("Ed", "Stark", 44));

        // Sorted by age
        System.out.println("\nSORTED BY AGE DESC:");
        persons.stream().sorted((p1, p2) -> p2.getAge().compareTo(p1.getAge()))
                .forEach(System.out::println);

    }
}
```

Output:

```
SORTED BY AGE DESC:
Person: [Name=Tywin, Surnames=Lannister, Age=63]
Person: [Name=Ed, Surnames=Stark, Age=44]
Person: [Name=Carlos, Surnames=Molero, Age=28]
Person: [Name=John, Surnames=Snow, Age=27]
```

### Shift

This one is dead simple, the whole Javascript `Array.prototype.shift()` implementation is covered by `myArrayList.remove(0)`. This will remove the first element of the array an return it.

### Unshift

For `unshift` we can create a method like this one using a loop and the `add` method with the index value. It will operate the same way, add the new elements to the beginning and return the new size of our array.

```java
public class Main {
    public static void main(String[] args) {
        List<Person> persons = new ArrayList<>();
        // Yes, now I'm a Game of Thrones character
        persons.add(new Person("Carlos", "Molero", 28));
        persons.add(new Person("John", "Snow", 27));
        persons.add(new Person("Tywin", "Lannister", 63));
        persons.add(new Person("Ed", "Stark", 44));

        System.out.println(unshift(
                new Person[] { new Person("Daenerys", "Targaryen", 22),
                        new Person("Theon", "Greyjoy", 33) },
                persons));
        System.out.println(persons.get(0));
        System.out.println(persons.get(1));

    }

    static int unshift(Person[] newPersons, List<Person> persons) {
        for (Person p : newPersons) {
            persons.add(0, p);
        }
        return persons.size();
    }
}
```

Output:

```
6
Person: [Name=Theon, Surnames=Greyjoy, Age=33]
Person: [Name=Daenerys, Surnames=Targaryen, Age=22]
```

---

We reached the end of this post. Hope you have learnt how to manipulate Java arrays in a similar way that in Javascript, of course there are corner cases since both languages are very different in other aspects but this is as close as it gets regarding arrays.

I encourage you to share your own tips and opinions in the comments! See you in the next post 🙂.
