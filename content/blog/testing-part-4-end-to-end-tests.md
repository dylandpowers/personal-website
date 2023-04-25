---
title: "Testing Part 4: End-to-end Tests"
tags: [Java, Testing, End-to-end, Acceptance, Selenium]
date: 2023-04-24
---

As discussed in [part 3 of this series](https://dylanpowers.me/blog/testing-part-3-integration-tests/), end-to-end tests sit at the top of the testing pyramid. They are the fewest in number, yet are still incredibly important. These are also sometimes referred to as _acceptance tests._ The two terms will be used interchangeably throughout this post.

# What are End-to-end tests?

End-to-end tests, as the name suggests, test how end users (which may be within your organization) interact with your application. This could be in the form of UI interaction or API consumption, depending on the nature of the application. The reason they are also called acceptance tests is because their passing is required for a user to “accept” that the requirements of the application have been met.

End-to-end tests are not restricted to frontend or backend - they test the whole flow of your application, from interacting with components in the frontend, to requesting data from the backend, to displaying that data again in the frontend. In this way we are testing the frontend code and the backend code, as well as how they interact with each other.

These should only test workflows that are accessible to users of your application. This is in contrast to unit tests, which should test functions that are ostensibly hidden from users. The test should start by performing some sort of workflow that a user could perform, and then assert on the end state of the application after it has been performed. Nothing in this process should be mocked - all APIs and services should function just as they would in production (but in a test environment).

As mentioned previously, we can sub-divide these tests into UI interaction and API consumption, so let’s go over each in more detail.

# UI Interaction

Let’s consider our code search application from the previous posts. Users would expect to be able to search for a piece of code in a search bar and then view all files that match that piece of code in a results list. We should add an acceptance test to verify this workflow.

First, we need to set up our test data. This means creating a test account for the user and a fake repository so that we can search for code. Since we aren’t mocking anything during these tests, we need to actually save all of this data to our database. To do this, you can either set up the account and repository as part of the test itself, or have a separate “test database” that always has this information stored. The downside of using the test database is that any changes to it may affect other tests, whereas setting up data in the test itself lets you always rely on your data being how you expect it to be.

For the frontend portion of the test, [Selenium](https://www.selenium.dev/) is a very useful tool. It allows you to automate browser actions like moving the cursor and clicking, so that you can reliably repeat these actions in tests. In our situation, we would use Selenium to click on the search bar, type in a query, and hit enter.

At that point, we would wait for the results of the query to come back to the page. These results are produced by a real API call that would happen without explicit invocation, since it should be invoked by the frontend components. At that point, we should wait for the request to finish, and then assert on the contents of the results list once it has finished.

Do not forget at the end of these tests to “clean up” any test data if you need to, as not to interfere with other tests in the same suite.

# API Consumption

If the only API in your application is your REST/GraphQL API exposed by your web server, and the only interactions with it are through your web or mobile app, then UI acceptance tests should suffice. However, that is rarely the case; usually, organizations have a handful of microservices that serve requests from other microservices or web servers.

In these cases, an end-to-end test just means spinning up an instance of that service, making a request to it, and verifying what the response should be. As before, we do not mock anything, and we may need to set up some test data beforehand.

Note that just because a web server has its endpoints tested via UI interaction tests, doesn’t mean it doesn’t need pure API tests as well. In fact, they should supplement each other.

Consider the same example as above, where we are testing the capability of our search page. In this case, we are going to make a request to the web server to search for a piece of code. Then, in the test, we will assert on what we expect the response to look like based on the test data that we inserted.

```java
@Test
public void search_simpleQuery() {
  setupTestRepository();
  JsonObject request = buildRequest(someFilePath);
  JsonObject headers = getHeaders();

  Response response = restApiClient.search(headers, request);
  Response expectedResponse = // ....
  assertThat(response, is(expectedResponse));
}
```

---

That does it for our testing series! To recap, there are three types of tests that are crucial to any software stack:

1. Unit tests. These are the most in number. They involve testing individual functions in your application. All underlying dependencies should be mocked.
2. Integration tests. These involve testing classes that interact with dependencies across service boundaries, i.e. with a database. In these tests, dependencies are not mocked.
3. End-to-end tests. These test external-facing workflows in your application, like interacting with a UI or consuming an API. Even in the tests, you interact with the system as if you were an external user.

I hope this series has been useful. Happy testing!
