# End-to-End Testing in Cypress

## Learning Goals

- Describe end-to-end testing.
- Create Cypress tests.

## Introduction

As we've discussed before when discussing the Testing Pyramid, one of the
advantages of unit tests is that they can be run very quickly and thus provide a
tight feedback loop for developers. They do not, however, provide a good way to
test functionality that spans multiple components and/or that must focus on the
user experience with the application's functionality as if they were going
through it in real-life.

For that, we have end-to-end testing, which is an activity that does focus on
the entire experience and that requires specific tooling aimed at replicating a
complete browser experience.

Cypress is the tool we will use for this type of testing.

First, let's install Cypress in our application project. From the project's root
directory, run the following command:

`npm install cypress -D`

This will install Cypress in your application, after which you'll be able to run
cypress from the command line with the following command:

`npx cypress open`

This launches the cypress UI:

![Cypress Welcome](https://curriculum-content.s3.amazonaws.com/java-mod-8/ng-messaging-cypress-welcome.png)

Since we're doing our component testing with Jasmine and Karma, we will focus on
using Cypress for end-to-end testing, so select that option on the welcome
screen. Cypress then creates a set of configuration files for you and displays
the following confirmation screen:

![Cypress Configuration Files](https://curriculum-content.s3.amazonaws.com/java-mod-8/ng-messaging-cypress-configuration-files.png)

Click on "Continue" at the bottom of that screen and Cypress will ask you to
choose a browser - choose Chrome:

![Cypress Choose Browser](https://curriculum-content.s3.amazonaws.com/java-mod-8/ng-messaging-cypress-choose-browser.png)

You're now ready to create your first test spec with Cypress:

![Cypress Create First Spec](https://curriculum-content.s3.amazonaws.com/java-mod-8/ng-messaging-cypress-create-first-spec.png)

Go ahead and create an empty spec with the default values and run it. It will
give you the following output:

![Cypress Initial Results](https://curriculum-content.s3.amazonaws.com/java-mod-8/ng-messaging-cypress-initial-results.png)

Let's look at the work that Cypress did for us through the previous few steps:

1. When we install Cypress through the `npm` command, it updated our
   `package.json` and install those dependencies
2. It also created a `cypress.config.ts` config file - this allows us to
   customize Cypress' behavior, although the defaults work just fine for us at
   the moment
3. When we asked Cypress to create an empty spec file, it created `cypress/e2e`
   folder in our project directory and placed the `spec.cy.ts` file there

Let's look at that file now:

```typescript
describe("empty spec", () => {
  it("passes", () => {
    cy.visit("https://example.cypress.io");
  });
});
```

This should look familiar to you, as the base structure of this test file is
very similar to what we saw in our unit testing section:

1. `describe()` lets us define a test suite
2. `it()` lets us define a specific test

Within this first test, we simply visit a URL that cypress provided with the
`cy.visit()` function:

1. `cy` is a reference to the cypress system that the cypress framework gives
   us. We can use it to ask cypress to perform specific functions
2. In this case, we use `cy` to ask cypress to `visit()` a specific URL
3. The `visit()` commands, as many cypress commands do, has assertions built in,
   so cypress would automatically fail this test if it wasn't to load the
   content of the web site

Let's make a simple change to this test so that it loads our application that is
currently running on `localhost:4200`:

```typescript
describe("empty spec", () => {
  it("passes", () => {
    cy.visit("http://localhost:4200");
  });
});
```

This simply validates that our application is up and running. Stop your
`ng serve` command and refresh the test to see it fail.

Now start `ng serve` again, so we can continue testing the application.

> Note: you may have to start cypress again to make it recognize that the server
> is running again. If so, simply stop your `npx cypress open` command and start
> it again

Let's now update our `spec.cy.ts` file to test the navigation functionality of
our messaging application:

```typescript
describe("empty spec", () => {
  const appURL = "http://localhost:4200/";
  it("passes", () => {
    cy.visit(appURL);

    cy.contains("New Message").click();
    cy.url().should("include", "contactList");

    cy.contains("Start Message").click();
    cy.url().should("equal", appURL);
  });
});
```

Let's break down this code:

1. We create a variable to hold our application URL, since we're using it
   multiple times within our test
2. We use the `cy.contains()` function to validate that the UI does have the
   "New Message" button we are expecting
3. We then chain that call with the `click()` call that tells cypress to click
   on that element
4. We then use the `cy.url()` call to get the URL as it shows in the browser's
   address bar
5. We call the `should()` assertion to check that the URL does include
   "contactList"
6. From there, we use the same sequence of `contains()` and `click()` calls as
   before, but to navigate back to the main screen of our application
7. This time, we validate the exact value of the URL instead of using the
   `contains()` method

Even though the overall structure of the test file is similar to our unit tests,
there are a few differences to point out:

1. Even with a test as simple as the one we just created, you might have noticed
   that we have many assertions in this single test. This is something that we
   explicitly cautioned against in unit tests. For end-to-end tests, however,
   since the steps to get to where we need to be to test functionality are more
   complex, it does make sense to have multiple assertions once we get the
   application there.
2. The cypress commands combine assertions and commands sometimes. For example,
   you don't have to manually confirm that an element exists before asking
   cypress to click on it. The test will fail with a helpful error message if it
   can't find the element you requested.
3. Some calls can be chained so that you don't have to create temporary
   variables to hold the results of a call. The `contains()` call is an example
   of that here: `cy.contains('New Message').click();`

## Additional end-to-end testing considerations

One of the reasons why automated testing is very key to take on as early on a
project as possible is because making sure that your code is testable will make
your code a) more testable, obviously, which is good for long term stability,
but also b) more maintainable, as testable code is usually also well-structured
and compartmentalized code.

Let's imagine, for example, that we now want to test the "send" function of our
application, i.e. we want to test that when a user enters a message in the input
box and hits the send button, the message properly shows up in the conversation
thread.

We are unable to test this at the moment because we don't have a way to
`cy.get()` the input box for the user's message. Let's address that by giving it
an explicit `name` attribute:

```html
<input name="user-message-input" <!-- giving our input box a unique name -- />
type="input" class="form-control" placeholder="Type a message"
[(ngModel)]="messageString"/>
```

Now we can reliably use `cy.get()` to get the input element and use the `type()`
function to actually enter text in it:

```typescript
cy.get("input[name=user-message-input]").type(testMessage);
```

We can also find the "Send" button and use the cypress `click()` function to ask
cypress to click on it to test the send functionality. We can then check the
conversation thread component's view to ensure that it contains the text for the
message we just sent.

Here is the updated code for our cypress test:

```typescript
describe("empty spec", () => {
  const appURL = "http://localhost:4200/";
  const testMessage = "cypress test message"; // the message this test will type and check for

  it("passes", () => {
    cy.visit(appURL);

    cy.contains("New Message").click();
    cy.url().should("include", "contactList");

    cy.contains("Start Message").click();
    cy.url().should("equal", appURL);

    cy.get("input[name=user-message-input]").type(testMessage); // get the input box by it 'name' attribute and type in it
    cy.contains("Send").click(); // "Send" the message
    cy.get("app-conversation-thread-component").should("contain", testMessage); // check that the message was sent
  });
});
```

In general, here are some considerations to take into account when structuring
your views to make sure they will be as testable as possible:

1. The best way to make sure your views, components and services are testable is
   to actually test them as you create them - do not postpone testing until
   "later". It's alarmingly common in software projects for "later" to become
   "never".
   > Note: we did so in this course because of our sequence of learning
   > objectives. But now that you know how to write unit and end-to-end tests,
   > you should do so as soon as you create your components
2. If you have a logical grouping of components, make sure that grouping is
   reflected in your views. This is one of the reasons why we came up with the
   application hierarchy that we shared earlier.
3. Ensure that all key components can be uniquely identified.

There is one more critical consideration you should take into account when
testing, which applies for both unit and end-to-end testing. We've already
encountered this problem, but you may not have noticed because our tests have
been sufficiently loose to avoid this problem actually causing any of our tests
to fail.

The consideration is "repeatability": tests should be able to run multiple times
and yield the exact same results. In other words, we should be able to return
the application to its initial state after we're done running our tests.

This translates differently for unit vs end-to-end tests:

1. For unit tests, this usually means mocking external dependencies. This works
   in the context of unit tests because they are very narrowly focused on
   validating the functionality of a specific component.
2. For end-to-end tests, mocking is generally not a preferred solution because
   it defeats the purpose of going "end to end". These types of tests need to
   exercise the entire application by definition. Here are a couple of
   alternative to mocking for end-to-end tests to achieve repeatability:
   1. Create (and destroy) your specific test data. In a later version of our
      application, for example, one could imagine that our end-to-end tests
      would register their own users, start new conversations, test the
      functionality for those conversations, and then promptly remove those
      users.
   2. If, as is our case here, the application does not yet support create a
      whole set of test data, then the end-to-end tests can be focused on remove
      the specific data that it creates. In our case, this means deleting the
      test message after we validate that it was sent properly.

## The Cypress UI

Let's have a closer look at UI of Cypress when a test suite is selected and has
run:

![Cypress Test UI](https://curriculum-content.s3.amazonaws.com/java-mod-8/ng-messaging-cypress-annotated-ui.png)

Let's examine the main elements of the UI:

1. When you entered the command to run cypress, it started a web server on port
   `4200` and opened a browser window to connect to that server
2. This URL happens to be on the same host and port as the cypress runner, but
   that's only because that's where the application we're testing is currently
   running. If our `cy.visit()` command had pointed to "https://www.google.com",
   that's what this URL would be displaying
3. This panel shows the state of the UI as the test progresses through its
   steps:
   1. It can either show the state of the UI at a specific step, when that step
      is selected in the side panel
   2. Or it can show the end state of the UI when all the steps in the selected
      suite has been run
4. This shows a list of all the test suites in all the directories that the
   cypress runner is aware of
5. This shows a status of each test in the test suite, including how many passed
   and how many failed, and also gives the user the option to rerun the tests
6. In this panel, the user can:
   1. See a summary of the outcome of each step
   2. See what network requests were made during each step
   3. Select a specific step and see the state of the UI when that step was
      executed (more on that below)

An important feature of Cypress is that it lets us inspect any specific step in
the tests that have been run. For example, here we are looking at the step where
we asked cypress to click on the "New Message" button:

![Cypress Event Replay](https://curriculum-content.s3.amazonaws.com/java-mod-8/ng-messaging-cypress-event-replay.png)

Here are a few important things to note:

1. The "New Message" button is highlighted by cypress to show the HTML element
   that the action is being taken onl
2. At the bottom of the screen, you can see that cypress shows a "before" and
   "after" button, which lets the user see what the UI looked at before the
   current action was taken, and what it looked like after the current action is
   taken.
3. The "Highlights" toggle lets you turn the button highlight on the current
   screen on or off.
