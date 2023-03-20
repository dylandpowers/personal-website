---
title: "Testing Part 2: Unit Tests"
tags: [Java, Testing, Unit, Mockito, Hamcrest]
date: 2023-03-19
---

Unit tests will most likely be the most in number in your codebase. If you master them, you can write them quite quickly. In this post, I will go over what a good unit test looks like, and how to mock dependencies effectively.

To review the last post: a unit test should test a single unit of work. Given an input to a function, we assert on what the expected output should be.

# Mocking

{{<lead>}} Only changes to the class under test can affect the results of the test. {{</lead>}}

In our unit tests, all dependencies should be mocked. Mocking means that instead of functions in our dependencies actually being executed, we _mock_ those functions by telling them what to return for certain inputs. For all intents and purposes, those functions never actually run during tests. This concept is useful for two reasons:

1. Breaking changes to code in dependencies will not affect the results of the current test. As long as the contract stays the same - the input and output types - unit tests will not be broken by changes in dependencies. _Only changes to the class under test (CUT) can affect the results of the test._
2. We eliminate any network calls. For example, if one of our dependencies queries a database, we avoid that query by mocking. This means we don’t need to set up the unit test suite to interact with the database, and the tests will run quicker because we don’t need to make any network round trips.
3. We do not need to construct any of our dependencies. Without mocking, we would need to also construct our dependencies’ dependencies, and dependencies to those dependencies - as you can see, the problem quickly becomes exponential.

## Mockito and Hamcrest

In this series, since we are focusing on Java, I will mention what the industry believes to be the best library for mocking: [Mockito](https://site.mockito.org/). Its convenient `when(...).thenReturn(...)` syntax makes unit testing a breeze. We call this process _stubbing the method._

Usage of the `when()` clause warrants some additional explanation. In addition to telling Mockito what function to look for, we also need to tell it the expected inputs to that function. Often times, we won’t know the input to the function, or we always want it to return the same thing regardless of the input. For situations like these, we should use [Hamcrest matchers](https://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/Matchers.html), which provide a highly expressive language for more readable and maintainable testing.

Some of the most useful matchers when you want the mock to return the same thing for any input are `any()` and its variants, like `anyList()`, `anyCollection()`, etc. For example, consider our `FileMatcher` class from the [part 1 post](https://dylanpowers.me/blog/testing-part-1-the-types-of-tests/):

```java
public class FileMatcher {
  private final QueryTokenizer queryTokenizer;

  public FileMatcher(QueryTokenizer queryTokenizer) {
    this.queryTokenizer = queryTokenizer;
  }

  public Set<FileMetadata> getMatchingFiles(SearchQuery query) {
    // ...
  }
}
```

In a test, we would need to mock the `QueryTokenizer` dependency. We can use the [`@Mock` annotation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#9) on the object in the test, and then call `openMocks(this)` during setup which will initialize all the mocks. Then, we can mock methods on the `QueryTokenizer` as described above:

```java
private static final TokenizedQuery TOKENIZED_QUERY = // ...

when(queryTokenizer.tokenize(anyString()).thenReturn(TOKENIZED_QUERY);
```

If you want to change the return value based on the input, you can use the [`doAnswer()`](https://javadoc.io/doc/org.mockito/mockito-core/2.4.0/org/mockito/Mockito.html#12) API:

```java
doAnswer(invocation -> {
  String input = (String) invocation.getArguments().get(0);
  // do something with input, return a TokenizedQuery instance
}).when(queryTokenizer.tokenize(anyString());
```

Of course, you can also stub the method for a specific input:

```java
when(queryTokenizer.tokenize("file:TestFile.java").thenReturn(TEST_FILE_TOKENIZED_QUERY);
```

Note that if you do not mock a function and the underlying code calls it, Mockito has a set of default return options based on the return type. For example, functions that return collections default to empty collections, functions that return booleans default to false, etc.

### Verifying Function Calls

In addition to asserting on the return value of your CUT, you might want to verify that a dependency called some function during invocation of the function in the CUT. For this, Mockito provides the [`verify()`](<https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#verify(T)>) function. Much like when mocking the return values, you can also use Hamcrest matchers when verifying what was called, like this:

```java
verify(queryTokenizer).tokenize(anyString());
```

_However_, in cases like this I would avoid this line; if the method weren’t stubbed out, it would return `null` and the test would fail anyways. The more common use case for `verify()`, in my opinion, is verifying that some method was _never_ called. For example, maybe we don’t want to do this expensive query tokenization operation if part of the input `SearchQuery` is invalid, and bail out immediately. In that case we can use the following:

```java
verify(queryTokenizer, never()).tokenize(anyString());
```

It’s also often useful to check that a specific argument was passed:

```java
verify(queryTokenizer).tokenize("file:TestFile.java");
```

## Antipattern: Mocking Data Classes

Data classes, often referred to as plain-old-java-objects (POJOs) in Java, simply hold data structures and do not perform any business logic. I’ve seen these be mocked before, basically meaning that you mock the return value for all of the getters. The value proposition here is that often times these POJOs have many members, but you only care about one or two in the test so constructing all of the constituent objects is overkill.

Yes, it will take longer to construct the whole object, but mocking these POJOs can lead to “invalid” inputs. For example, if a lot of fields are non-nullable, then the source code may (correctly) assume that they can access these fields reliably. If an object is mocked and a developer forgets to mock a getter, and the getter is invoked in the source code, the tests may fail unexpectedly. It’s better to just create the objects from scratch instead of mocking them.

# What to Test

{{<alert>}} Every public method needs to be tested. {{</alert>}}

Private methods do not need tests but if these methods become sufficiently large, you should consider just making them public or moving them to a different utility class. Here are some general rules of thumb to follow:

1. Test the “happy path” case. This is the case where there is a valid, non-empty input. Try to test as many permutations of inputs as possible; of course if the number of permutations is unreasonable, use your discretion, but test the most important ones.
2. Test the empty input case. Typically this should also involve verifying that certain dependencies weren’t invoked.
3. Test exception cases. If your method throws exceptions, make sure they are thrown in the correct instances.

## Naming Tests

This section will become slightly more opinionated, but these are techniques I’ve used to consistently name tests that I find work well.

Imagine we are testing the `getMatchingFiles()` function from the part 1 post. The first part of the test method name should always start with the name of the function being tested. In my opinion, nothing needs to be prefixed with “test”, since we are already in a test file and thus that should be obvious. Next, add an underscore and describe the parameters being tested. Finally, add another underscore and describe the desired outcome. Here are some examples:

```java
public void getMatchingFiles_emptySearchQuery_returnsEmptySet()
public void getMatchingFiles_simpleQuery_returnsCorrectFile()
public void getMatchingFiles_complexQuery_returnsMultipleCorrectFiles()
```

Just by looking at the name of the test, it’s glaringly obvious what the test does. We should give the same attention to our test method names that we give to method names in our source code 🙂
