---
layout: post
title:  "Unit testing Exceptions in Java with JUnit"
date:   2020-08-11 14:34:25
categories: testing
tags: unit testing junit
image: /assets/article_images/2020-08-11-unit-testing-exceptions/exception-handling.jpg
image2: /assets/article_images/2020-08-11-unit-testing-exceptions/exception-handling.jpg
---
I have seen people unit testing the core logic and skipping on the Exception cases of their code considering it of little importance or harder to test. In my opinion, exception cases are not of lesser importance and by no means they are harder to test. 

Here we will consider many ways to `mock` exceptions and `assert` exceptions with JUnit.

## Mocking Exceptions
With `Mockito` you can not only mock methods to return something but also you can mock them to throw exceptions using `Mockito.when`.

You can mock method to throw exception of some type or appropriate exception object itself.

Let's see a class `TestMe` that we are testing

{% highlight java %}
public class TestMe {

  private Validator validator;

  public TestMe(Validator validator) {
    this.validator = validator;
  }

  public String execute(int number) {
    try {
      validator.validate(number);
    } catch (Exception e) {
      return null;
    }
    return String.valueOf(number);
  }

}
{% endhighlight %}

Suppose we want to test a case where validation fails and the `execute` method returns null.

We can mock exception and test the code as follows.
{% highlight java %}
public class TestMeTest {

  @Mock Validator validator;
  private TestMe testMe;

  @Before
  public void setUp() throws Exception {
    MockitoAnnotations.initMocks(this);
    testMe = new TestMe(validator);
  }

  @Test
  public void resultOfExecutingInvalidNumberIsNull() {
    int number = -1;
    Mockito.doThrow(IllegalArgumentException.class).when(validator).validate(number);

    String result = testMe.execute(number);

    Assert.assertNull(result);
  }
}  

{% endhighlight %}

This test makes `validator.validate` method of mock validator to throw `IllegalArgumentException` on which the method returns null and we make our test expect null as a result.

You can use object of Exception instead of class in `doThrow` as 

{% highlight java %}
Mockito.doThrow(new IllegalArgumentException("No negative number"))
.when(validator).validate(number);
{% endhighlight %}

There is another way to mock exception, that is using `Mockito.when`:

{% highlight java %}
Mockito.when(validator.validate(number)).thenThrow(IllegalArgumentException.class);
{% endhighlight %}

I used to write it this way, but it does not work for mocking method that returns `void`. So I switched to using `doThrow`, `doReturn` and `doNothing` instead of using `when` for all the mocking.

`doReturn` way is more readable and less confusing if you compare it with `when`.

## Asserting Exceptions
Sometimes your code is expected to throw exception and there is no reason to not to test that case. It's not hard, rather it is so simple that there are 3 different ways to do it, each with it's own pros and cons.

Consider the code below that we are testing:
{% highlight java %}
public class TestMe {

  private Validator validator;

  public TestMe(Validator validator) {
    this.validator = validator;
  }

  public String perform(int number) {
    if (number < 0) {
      throw new IllegalArgumentException("Number can not be negative");
    }
    return String.valueOf(number);
  }

}
{% endhighlight %}

We want to assert that when `perform` methods is called with a negative number it fails.

Let's consider three ways to assert that case.

### Using expected param of @Test annotation
@Test annotation of JUnit takes `expected` param of class type that extends from Throwable.

If the code inside the test throws the exception of type given in param, then the test passes otherwise it fails.

So, the test becomes.
{% highlight java %}
  @Test(expected = IllegalArgumentException.class)
  public void performingNegativeNumberFailsWithExpected() {
    testMe.perform(-1);
  }
{% endhighlight %}

Pros 
* Concise as there is no extra code to be written to validate exception

Cons
* Can not test details like message of the exception
* Code written after method call `perform` is not executed so it prevents writing verification if we have any.


### Using try catch explicitly
Inside the test itself we can invoke the method we are testing inside a try catch block and expect code to throw exception and to fail if no exception is thrown or thrown exception is of wrong type.

We use `Assert.fail` at places where code is not supposed to reach.

{% highlight java %}
  @Test
  public void performingNegativeNumberFailsWithTryCatch() {
    try {
      testMe.perform(-1);
      Assert.fail("Exception expected");
    } catch (IllegalArgumentException expected) {
      String message = expected.getMessage();
      assertThat(message, containsString("can not be negative"));
    } catch (Exception anyOther) {
      Assert.fail("Unexpected Exception");
    }
  }
{% endhighlight %}
If `perform` method does not throw any exception `Assert.fail("Exception expected");` will be executed which fails the test.

If `perform` method throws exception of expected type, that is `IllegalArgumentException` here, then first catch block would be executed where we also can assert extra stuff.

If `perform` method throws exception of some other type then second catch would be executed where `Assert.fail("Unexpected Exception");` would fail the test.

Pros
* Code flow continues so we can make verifications after the method call.
* Can perform extra validations like expected message.

Cons
* Too much of a code.


### Using ExpectedException rule
JUnit comes with a handy rule to assert exceptions. We use it to assert exception type, message and some other details.

For this you need to use ExpectedException `@Rule`.
{% highlight java %}
  @Rule
  public ExpectedException expectedException = ExpectedException.none();

  @Test
  public void performingNegativeNumberFailsWithExpectedExceptionRule() {
    expectedException.expect(IllegalArgumentException.class);
    expectedException.expectMessage("can not be negative");

    testMe.perform(-1);
  }

{% endhighlight %}

Here you make assertions about expected exception type and message before the method call which is expected to throw exception.
If method throws right type of exception with right message content (checked with `contains`), then the test passes otherwise fails.

Pros
* Concise code, no try catch blocks.
* Can assert details of exception.

Cons
* Code written after method call `perform` is not executed so it prevents writing verification if we have any.

Considering these options, I generally prefer to use `ExpectedException` rule unless I want explicit verifications to be done after the method call.

By explicit verifications, I mean verifying some method was in fact called with using `Mockito.verify`. Usually that case should be covered with non-exception cases, but sometimes cases where you have some code in exception handing that you want to assert that it was called, then you need to use the verbose `try-cath` way only.

So, are you going to start testing your sad exception cases or leave them neglected?
