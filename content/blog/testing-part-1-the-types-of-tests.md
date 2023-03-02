---
title: "Testing Part 1: The Types of Tests"
tags: [Java, Testing, Unit, Integration, Acceptance, Sourcegraph]
date: 2023-03-01
---

# Testing Part 1: The Types of Tests

> Why do I need to write tests? I wrote the code so obviously it works.

Well, yeah, everybody knows that. However, writing tests is a fact of life in the software world, and if we’re going to do something, we might as well do it right. In this series I will outline my testing philosophy and the methods I believe to be the best for writing tests.

At it’s core, all software testing can be distilled to one sentence:

> Given an input, assert on an expected output.

However, as we’ll see, there’s much more to it than that.

Throughout this series, the example app I will use is a code search app, something like [Sourcegraph](https://about.sourcegraph.com/). Because [todo lists are boring](https://dylanpowers.me/micros/why-is-everything-a-todo-app/) (what an incredible author that wrote that).

# The Types of Tests

The first step toward creating a great testing suite is to understand the types of tests that you need. For any large application, I believe your organization needs (at minimum) three types of tests, each of which I will describe in detail:

1. Unit
2. Integration
3. Acceptance A.K.A. end-to-end

Individually, each type of test is extremely powerful, but together they make your application resilient to almost anything.

## Unit Tests

The first type of tests is unit tests, which undoubtedly will be the most by number in your codebase. They are called unit tests because they test one “unit” of work isolated from everything else. Typically, this means a single function. Something like, given a query, return a list of file metadata objects that match that query:

```java
public class FileMatcher {
	private final QueryTokenizer queryTokenizer;

	public Set<FileMetada> getMatchingFiles(SearchQuery query) {
		// ...
	}
}
```

When unit testing a function, you provide an input, and assert on what you would expect the output to be for that input. Here, we would construct a test `SearchQuery`, and then assert on what we would expect the `FileMetadata` to look like for a given query.

This unit of work needs to be isolated from other units of work that would occur during your function call. Other units of work might include dependencies calling other functions. In this example, a dependency might be some sort of `QueryTokenizer`.

When we say that each unit needs to be tested in isolation, this means that we shouldn’t allow the results of other units to affect this unit. This means we need to [mock](https://circleci.com/blog/how-to-test-software-part-i-mocking-stubbing-and-contract-testing/) those dependencies, a concept I will cover in a later post. However, the main takeaway is that ********************\*\*********************\*\*********************\*\*********************unit tests should never make network calls.********************\*\*********************\*\*********************\*\********************* If you have a unit test talking to a database, then it’s not a unit test anymore.

## Integration Tests

These test integration with other services that your code connects to, typically over a network. These might be databases, search indexes, or other microservices within your company. With integration tests, network calls are normal and usually expected. Imagine we have a class `HistoricalQueryStore`, which will return past queries for a given user:

```java
public class HistoricalQueryStore {
	private final DbClient db;

	public Set<SearchQuery> getPastQueries(long userId) {
		// ...
  }
}
```

Here, calls to `DbClient` directly communicate with a database server.

We still treat the testing process the same: given an input, assert on the expected output. The main difference is that we are not mocking our underlying dependencies.

However, it should be noted that not every class with dependencies needs both unit and integration tests. Integration tests should only test classes that interact with **\*\***other**\*\*** parts of the system, outside of your domain, and most likely over a network. For example, if the `QueryTokenizer` above is in your domain and involves a simple in-process function call, there would be no need for an integration test. However, if it had a dependency on an `ElasticSearchClient`, an integration test may be appropriate and warranted.

## Acceptance/End-to-End Tests

These test your application from a 10,000 foot view, typically from an API level. Given an API request, assert what you would expect an API response to be. Typically, these do not involve any mocking. Whether your application is a REST or RPC or some other sort of API, these tests should be treated the same. These are called “acceptance” tests, because without these passing, no application build should be accepted.

On the frontend, these tests typically involve simulating user interactions on a page - however, in this series I will only cover backend testing practices.

---

In follow-up posts, I will deep dive into each of the three types of tests outline above. After that, I will rewrite this entire series using Rust instead of Java.
