---
title: Hamcrest contains() is weird
tags: [Java, Hamcrest]
date: 2023-02-02
---

I have been using [Hamcrest in Java](https://hamcrest.org/JavaHamcrest/) for as long as I can remember. I love using it because it's _much_ more powerful than native JUnit assertions, and the assertions read much easier. So instead of this:

```java
assertEquals(actual, expected);
```

We get:

```java
assertThat(actual, is(expected));
```

Yeah, it's actually more keystrokes, but that's Java's thing 😎.

My favorite part about Hamcrest is using it for asserting on items in collections, but the behavior of the `contains()` matcher can be quite odd. Say you have a list `[A, B]` and you store it in a variable `list`. If you run

```java
assertThat(list, contains("A"));
```

The assertion will actually fail - this is because the `contains()` matcher actually means `containsExactly()`. I use this custom assertion as a convenience:

```java
public static void <T> containsExactly(T... members) {
  contains(members);
}
```
