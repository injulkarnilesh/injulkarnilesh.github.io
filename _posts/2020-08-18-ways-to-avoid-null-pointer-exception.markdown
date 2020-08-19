---
layout: post
title:  "7 ways to avoid NullPointerException in Java"
date:   2020-08-18 14:34:25
categories: java
tags: java null pointer exception
image: /assets/article_images/2020-08-18-npe/npe.jpg
image2: /assets/article_images/2020-08-18-npe/npe.jpg
---
Sir Charles Antony Richard Hoare, British Computer Scientist, inventor of Quick Sort, first introduced the null reference in 60's programming language ALGOL W. Afterwards he himself called this decision of his a 'A Billion Dollar Mistake'.

If you have been developing in programming language like Java where there is `null`, you must have faced NullPointerException in production, in-fact that might be the most faced exception in your application.

Following are some ways that I have learned one could use to avoid `NullPointerException` in `Java`.

# Use Optional
`Null` is used to represent something that is not known or that has no value.
`Optional` class was first introduced in Java version 8, with main purpose to represent a variable whose value can exist or not exist. Before Java 8, Google's Guava library had a similar class with same purpose.

Instead of returning null or taking parameter whose value can be null, prefer to use Optional.

In this example, if user is logged in it returns User object and if not logged in then return null.
{% highlight java %}
public User getUser() {
  if(isLoggedIn()) {
    return loggedInUser();
  }
  return null;
}
{% endhighlight %}

The code calling this method, would need to check if user returned is null or not. But the fact that it can return null is not explicit.
{% highlight java %}
User user = getUser();
if(user != null) {
  String userName = user.getName()
}
{% endhighlight %}


Some clients of the code might not add that `null` check and directly use the object resulting in our beloved NullPointerException.

We can make `not-exist` possibility of the returned User object explicit by making it Optional as:
{% highlight java %}
public Optional<User> getUser() {
  if(isLoggedIn()) {
    return Optional.of(loggedInUser());
  }
  return Optional.empty();
}
{% endhighlight %}

The client code now knows, the returned object can or cannot exist. So it would use methods of `Optional` to check that.

{% highlight java %}
Optional<User> user = getUser();
if (user.isPresent()) {
  String userName = user.get().getName()
}
{% endhighlight %}

You can use Optional at method parameters, fields as well to represent that the value can be absent.

# Use Empty Collections
Purpose of `Null` is to represent that value is not there. For collections of any type, we have a representation for not having any value, and that is `Empty Collection`.

Instead of initializing Collection like `List` to `null`, passing `null` for parameters of collection type or returning `null` for type of collection, we can use empty collection to represent no value.

If we have a field of collection type like below, if we don't initialize it to anything, it would be initialized to null, which could come and bite us back.
{% highlight java %}
public class User {
  private List<Permission> permissions;
}
{% endhighlight %}

To avoid possible NullPointerException at accessing permissions fields and using it, we can initialize it to empty `List` so that if it is not explicitly initialized it would be empty list and not null and so that it would be safe to access it and use.
{% highlight java %}
public class User {
  private List<Permission> permissions = new ArrayList<>();
}
{% endhighlight %}

Don't limit it to avoid NullPointerException but it is generally a good practice to follow.

# Null Object Pattern
Null Object is a design pattern to have a special object of a class to represent a no value case.
Instead of null we set value of the object to this special value which generally is singleton.

We can initialize null object as a static field of the class as follows:

{% highlight java %}
public class User {

  public static final User UNKNOWN_USER = new User("UNKNOWN", new ArrayList<>());

  private String userName;
  private List<String> permissions;

}
{% endhighlight %}

Here we have a UNKNOWN_USER field of User to represent a non known or no user.

Such a class can have a field or method that can be used to represent if value is `null object` or normal object. 

Example: User class above can have a isValid field/method that would be true for normal object and false for UNKNOWN_USER. Client code could use that method/field as :
{% highlight java %}
if(user.isValid()) {
  //
}
{% endhighlight %}

Another way to know if user is UNKNOWN_USER if compare it with UNKNOWN_USER with equals.
{% highlight java %}
if(!User.UNKNOWN_USER.equals(user)) {
  //
}
{% endhighlight %}

These ways are used to know if user is valid one or not. In some cases you don't event need to check that and you can just use the user object to whatever you want. In that case the code would use UNKNOWN_USER instead of null and would avoid NullPointerException. Like `user.getName` would be UNKNOWN and that could be ok in many cases and there we don't event check if object is null or UNKNOWN_USER.

# Avoid mixing Primitives and Wrappers
Image a method `myMethod` that takes parameter of type `int`. Java lets you pass variable of type `Integer` to it performing magic of `boxing` and `unboxing`.

{% highlight java %}
public void myMethod(int age) {
  //
}
Integer age = null;
myMethod(age);
{% endhighlight %}

Above code compiles but fails at runtime with the deadly exception we are talking about.

Here we use `int` to represent age at one place and `Integer` at another place. Which is valid. But can lead to NullPointerException when we try to assign `Integer` variable to `int` variable.

So it is better to always use either wrapper or primitive to represent an entity in entire of the code base. Choose whatever you want, but stick to it everywhere, do not mix.

# Use Util Classes
At multiple places you generally check if object is not null and do some extra check followed by `&&`.
Code like following might seem usual to many: 

{% highlight java %}
if (userName != null && userName.length() > 0) {
  //
}
{% endhighlight %}

Many a times we forget such a null checks at places where it might be required. To avoid having to remember such checks we can create or use existing utility methods that does it for us.

{% highlight java %}
public class Size {

  public static int of(Collection<?> collection) {
    return collection == null ? 0 : collection.size();
  }

  public static boolean isEmpty(Collection<?> collection) {
    return collection == null || collection.isEmpty();
  }

  public static <T> Collection<T> safe(Collection<T> collection) {
    return collection == null ? Collections.emptyList() : collection;
  }

  public static <T> Stream<T> stream(Collection<T> collection) {
    return collection == null ? Stream.empty() : collection.stream();
  }
}

{% endhighlight %}

This Size utility can be used at all the places to perform collection related safe operations.

For Strings, there exists many utility classes that does null checks and extra checks for you. You can build a practice to use one of such utilities for manipulating or checking strings.

# Cover null cases in Unit Tests
This may sound very idealistic, but many of the NullPointerExceptions should be caught in Unit Testing itself. NullPointerException means object was null and we tried to perform some operation on it. If we know some object can be null then why don't we write unit test cases covering scenarios where that object is null?

No need to add null checks for all the objects. But at places we expect them to be null, we should add null checks as well as Unit tests to make the nullability explicit.

If we follow above mentioned ways to replace null with Optional, empty Collections or Null Objects we should not have a code that passes null or returns null anywhere inside our codebase. But sometimes we get objects form other systems and libraries which we don't control, there we can be benefited by having unit tests covering null scenarios.

# Use Kotlin
Yeah, I know! Not many folks gonna like this option. But let me explain what it is.

Kotlin is a modern JVM based programming language from JetBrains with many promising features. Android project has Kotlin as main programming language and Spring has recently started support for Kotlin.

One of the great features of Kotlin we are interested in is difference between nullable types and non-nullable types. 
A variable at declaration should explicitly be declared nullable if it can hold null value. Then methods called on such nullable would be called with a little different syntax which checks null first and does not call the method if object is null.

{% highlight kotlin %}
var userName: String = "Nilesh"  //userName is non null
var userPermission: String? = null //userPermission is nullable

var userNameLength = userName.length  //userNameLength is of type Int
var userPermissionLength = userPermission?.length 
//if userPermission is null .length is not called
//userPermissionLength is of type Int? (nullable Int)
// userPermission.length would not even compile

{% endhighlight %}

Advantage of Nullable types is that, nullability is explicit. `?` operator makes NullPointerException impossible.

There are many other interesting features that Kotlin has which makes it worth consideration or at-least worth learning for knowing its features.


Anyway, with these approaches, you might be able to reduce chances of NullPointerException but you can't run away form it until you make bigger changes in your coding practices.