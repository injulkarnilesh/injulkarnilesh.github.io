---
layout: post
title:  "Un-Productive way to rename all the test methods"
date:   2020-07-20 14:34:25
categories: automation
tags: automation shell
image: /assets/article_images/2020-07-20-rename-tests/terminal.jpg
image2: /assets/article_images/2020-07-20-rename-tests/terminal-mobile.jpg
---
I had been working on a new micro-service in a newly joined team. Once basic functionality was ready and I got some breathing time, I thought I would look for any technical debt that I have accrued. During this, I got to know that the team has already been using Sonar for statistical analysis of the code.

When I checked the project's sonar status, I got to know many smells, debt and issues that I would need to fix. One of them was about naming the test methods. The rule configured was expecting test method name to follow

> ^test[A-Z][a-zA-Z0-9]*$

Basically test name should start with **test** word.

How I had been using was like `twoPlusTwoEqualsFour`. So to fix these apparent code smells, I had to rename around 230 methods to follow expected pattern.

Eg.
`twoPlusTwoEqualsFour` to become `testTwoPlusTwoEqualsFour`

I would have let it go and follow the naming convention for new tests I write. But that would cause me many sleepless nights knowing those smells exist there, which I can fix but chose not to.

I could have manually done it or have checked if that can be done with IntelliJ editor.
But the automation freak that I am, I decided to do it with few linux commands. 

I knew it would take more time to figure out the commands, but you know the compulsive disorder, couldn't beat it.

Besides

![Absolute Power](/assets/article_images/2020-07-20-rename-tests/commands.jpg)

Stepwise what I had to was :
1. Find methods annotated with `@Test`
2. Rename methods by capitalizing first letter and pre-pending `test` word.

Final command I used was 

{% highlight shell %}
grep -r -A1 --include=*Test.java "@Test"  | grep -v @Test |
 grep -v test | grep com | sed s/.java-/.java/ | cut -d' ' -f5,1 | 
 xargs -n2 sh -c 'sed -i "s/$1/test\u&/g" $0'
{% endhighlight %}

Stepwise explanation goes like this:

### grep -r -A1 --include=*Test.java "@Test" 
Using grep command to find methods annotated with `@Test`.
Options used:
* -r : to recursively find match in all files in current directory
* -A1 : Include line 1 after the matching line, there exists -C and -B like -A.
* --include : flag to include only Test classes

Partial output was:
{% highlight shell %}
src/test/java/com/org/project/name/util/TestClass.java:  @Test
src/test/java/com/org/project/name/util/TestClass.java-  public void twoPlusTwoEqualsFour() {
--
{% endhighlight %}


### grep -v @Test
Using -v option to remove the lines matching the word.
Result was:
{% highlight shell %}
src/test/java/com/org/project/name/util/TestClass.java-  public void twoPlusTwoEqualsFour() {
--
{% endhighlight %}


### grep -v test
Remove methods which already has test in the name


### grep com
To remove line `--` and include only the line with method name


### sed s/.java-/.java/ 
To replace `.java-` with `.java`. 
It is done to for resulting line to have file location required for next commands.
Result was:
{% highlight shell %}
src/test/java/com/org/project/name/util/TestClass.java  public void twoPlusTwoEqualsFour() {
{% endhighlight %}


### cut -d' ' -f5,1
To break the lines by delimiter ' ' (space) 
* -d : Specify delimiter that is space.
* -f : to pick up 1st and 5th column of split text
{% highlight shell %}
src/test/java/com/org/project/name/util/TestClass.java  public void twoPlusTwoEqualsFour() {
1                                                      2 3     4    5                      6
{% endhighlight %}

Due to double space between file name and public word, columns I had to pick were 1,5.
Result was:
{% highlight shell %}
src/test/java/com/org/project/name/util/TestClass.java twoPlusTwoEqualsFour()
{% endhighlight %}


### xargs -n2
`xargs` is used to build arguments for commands that follows. Option -n is for number of arguments.

A command that is passed to `xargs` would have argument in variables coming from result of previous command.

* $0 : src/test/java/com/org/project/name/util/TestClass.java
* $1 : twoPlusTwoEqualsFour()

### sh -c 'cmd'
Command used to run a command 'cmd', allows to build cmd at runtime


### sed -i "s/$1/test\u&/g" $0
Here $0 would be replaced by method name and $1 by file path, so the command run is 
> sed -i "s/twoPlusTwoEqualsFour()/test\u&/g" src/test/java/com/org/project/name/util/TestClass.java

Now this is some complex command.
This is sed (Stream Editor) command generally used to process and replace text in files.

Meaning of options as
* -i : Replace content in file itself (in-place), other related options are to print changed content without making actual changes or to make changes while creating backup file.
* $0 : coming from xargs is a file to edit
* `s/$1/test\u&/g`: this means find content matching $1(twoPlusTwoEqualsFour(), method name) and replace it with `test\u&` where `&` means matched content(twoPlusTwoEqualsFour()) \u does upper case next character. 

So it would replace  `twoPlusTwoEqualsFour()` with `testTwoPlusTwoEqualsFour()` which is what we want.

Due to magic of pipe, these commands get applied to all the matching methods in the directory.


Hurray! I did rename all the methods with few commands (taking more time than if it had been done manually). 

Had I done it manually though, I probably wouldn't have learnt these commands in a little more details.

Next time I need to do something similar, I would do it faster (may be).

Hope this it at-least helped you.
