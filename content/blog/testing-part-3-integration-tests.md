---
title: "Testing Part 3: Integration Tests"
tags: [Java, Testing, Integration, Database, SQS]
date: 2023-04-02
---

The _testing pyramid_ describes how your codebase should think about tests. Unit tests are the bottom layer because they are the most in number, and represent the baseline for all functionality in your app. The top layer is _end-to-end tests_, which are typically the fewest in number, but are of the utmost importance (these are sometimes also called _acceptance tests_ - we will cover these in a future post). The middle layer, and the subject of this post, is _integration tests_.

# What Is an Integration Test?

Unfortunately, defining this is an impossible task, as the definition can be quite tricky and can vary wildly between organizations. As the name suggests, the general idea is to test how your components (classes) integrate with other components. At surface level, it sounds like integration tests are just unit tests without the mocking - instead, we call into every service just as we would in a production instance. Technically, this is correct. The tricky part is setting boundaries for what does and doesn’t need an integration test.

Writing an integration test for every class that has unit tests is overkill. If that were the case, then we would have less of a testing triangle and more of a testing pentagon, since integration and unit tests would be the same in number.

# What Components to Test

It’s helpful to think about this in terms of your _service boundary._ The boundary separates your system from other external, usually third-party systems. Anything that reaches across the service boundary needs an integration test. This means if a component is querying a database, sending messages to Kafka, interacting with AWS, etc., it probably needs an integration test where it _actually_ does those things, instead of through a mock.

For example, consider our `FileMatcher` class from the previous posts:

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

In this instance, let’s assume that our team owns both the `FileMatcher` and `QueryTokenizer` classes. In this instance, we do not need to write an integration test for the `FileMatcher`, as we are not reaching across the service boundary.

On the other hand, consider a `QuerySaver`, which is run after the files have been matched, and saves a user’s queries to the database. A skeleton of this class might look like:

```java
public class QuerySaver {
  private final DbContext db;

  public QuerySaver(DbContext db) {
    this.db = db;
  }

  public void save(SearchQuery query) {
    // ...
  }
}
```

In this case, the `DbContext`, which directly queries the db, reaches across a service boundary. In this instance, an integration test is necessary.

One tricky part is when dealing with other microservices within your organization, typically owned by other teams. Spinning up actual instances of those in a test environment may be tricky without coordination from another team, as you have no idea what other services their service coordinates with. However, if you can reliably know that their service will be able to spin up, having an integration test using an actual instance of their microservice is worthwhile.

# How to Test

The nature of the test will vary depending on what the external system is. Here are a couple common external systems that need integration tests, and opinions on how to test them:

### Databases

To test classes that interact with a database, you likely want to ensure that loading and saving work properly. This typically involves storing data manually through a database context and then loading it through a class method, and asserting that the results are what you expect. The reverse is to store data through a class method, and then load it manually through the database context and assert it is what you expect. The reason we store through the database context _directly_ when testing load methods is that, if we don’t do that, then we are relying on the save method to work properly all the time, and that is not a reliable assumption. In order to test just a single function, one of the operations (load or save) must go directly through the `DbContext`.

### Third-party Services (i.e. AWS)

Say you have a class that publishes a message to [AWS Simple Queue Service](https://aws.amazon.com/sqs/):

```java
public class SQSMessagePublisher implements MessagePublisher {
  private final AmazonSQS client;

  public SQSMessagePublisher(AmazonSQS client) {
    this.client = client;
  }

  @Override
  public void publish(Message message) {
    // ...
  }
}
```

In this case, try using an actual instance of `AmazonSQS`, and test the `publish` method on your class. Much like in the databases example, we should manually verify somehow that a message was actually published to SQS, by using a method on the `AmazonSQS` client itself. You will likely also need to do some extra setup like setting up the queue, etc.

### Intra-organization Microservices

As mentioned earlier, exercise caution with these, as you likely will not know what services need to be spun up in a testing environment to run another team’s microservice. However, testing these is similar to how you would test a third-party service. Use the function in your component which will send or receive data from the microservice, and then use the microservice client itself for data in the opposite direction, whether that is sending data to set up a test or fetching data to verify.
