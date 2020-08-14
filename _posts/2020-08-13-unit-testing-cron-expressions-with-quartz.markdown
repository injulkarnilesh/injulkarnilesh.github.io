---
layout: post
title:  "Unit testing Cron Expression with Quartz"
date:   2020-08-13 14:34:25
categories: testing
tags: unit testing junit quartz spring java
image: /assets/article_images/2020-08-13-unit-testing-cron/cron-testing.jpg
image2: /assets/article_images/2020-08-13-unit-testing-cron/cron-testing.jpg
---
Once, we faced an issue that the Cron Job which was supposed to run at some frequency was in reality running at different frequency because of the wrong cron. The cron was manually written by a developer, s/he probably built it by self or built it with some online tool, but it happened to be wrong.

While fixing the issue, we though why can't we unit test the Cron. Unit testing is not limited to `imperative` code only, we test `declarative` code like SQL queries as well then why not Cron?

## Getting the Cron
To test the Cron, first thing you need to do is get the cron expression. Here we will get it from a method annotated with Spring `Scheduled` annotation just to show one of the ways. 
If your cron is at some other place, please identify a way to get hold of it inside the test.

For Spring `Scheduled` annotation, we get the cron with Java Reflection as follows:
{% highlight java %}
String cron = ClassToTest.class.getMethod("methodWithCron")
        .getAnnotation(Scheduled.class)
        .cron();
{% endhighlight %}

## Test the Cron
To test the Cron expression we do following:
* Assume current time is some time
* For the Cron expression, get next time that Cron would trigger assuming the current time is the one assumed in first step.
* Verify the next time to be expected time

To make dates more readable, we use some common date format for date comparison.
We use dates in test in readable string and while testing the Cron's current date and next date we convert the string dates to `Date` using the selected format.

Let's use following data format.
{% highlight java %}
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
{% endhighlight %}

Let's write a code, to get next Date for current Date according to Cron.

{% highlight java %}
/*
currentDate: Current date in same format
nextDate: Expected next date in same format
*/
private void verifyCron(String cron, String currentDate, String nextDate) throws ParseException {
    CronExpression expression = new CronExpression(cron);

    Date date = simpleDateFormat.parse(currentDate);
    Date next = expression.getNextValidTimeAfter(date);

    assertThat(next, is(simpleDateFormat.parse(nextDate)));
  }
{% endhighlight %}

What we do here is:
* Build CronExpression from Cron String
* Get current date parsed from String input
* Get next date from expression using `expression.getNextValidTimeAfter(date)`
* Compare next date with expected next date(parsed with same format)

Now let's use this method to verify the cron.

For simplicity we will test a cron for every hour: `0 0 0/1 ? * *`

{% highlight java %}

  @Test
  public void testCronForEveryHour() throws Exception {
    String cron = "0 0 0/1 ? * *";

    verifyCron(cron, "2020-08-10 03:45:00", "2020-08-10 04:00:00");
    verifyCron(cron, "2020-08-10 18:59:00", "2020-08-10 19:00:00");
    verifyCron(cron, "2020-08-10 18:00:01", "2020-08-10 19:00:00");
    verifyCron(cron, "2020-08-10 23:00:01", "2020-08-11 00:00:00");
  }
{% endhighlight %}

In the test method we are making multiple assertions for different corner cases. We definitely should test boundary conditions.

For every hour condition we have considered these corner cases :
* When current time is 18:59, then the next cron trigger should be at 19:00
* When current time is 18:01, then the next cron trigger should be at 19:00
* When current time is 23:01, then the next cron is next day beginning 00:00.

Once we have `verifyCron` method in place, we can test the cases you consider enough for your test Cron.

Go ahead find those Crons in your code, and test them up.