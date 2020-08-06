---
layout: post
title:  "Why I Unit Test my code"
date:   2020-08-04 14:34:25
categories: tesing
tags: unit testing
image: /assets/article_images/2020-08-04-testing/software-testing.jpg
image2: /assets/article_images/2020-08-04-testing/software-testing-mobile.jpg
---
In my career of more than 9 years as a Fullstack Software Engineer, I have worked on different types of projects, few legacy, few under maintenance and few newly started.
There were many things to observe, many things to learn. One of the things I learned by experience and from other developers was importance of Unit Tests.

In case of Unit testing I went through a few steps of learning which I believe have been important for my career, and those were as follows:

## Unit What?
The first project I worked on had no unit tests. I never had even heard of the term. But testing was unavoidable which happened to be all manual for that project.

To implement a feature or fix a bug I would go through following steps:
1. Make a code change
2. Start the application locally.
3. Open web app in browser and go through pages or hit some URL so that code flows through my changes.
4. If any error/bug is found stop the app and goto step 1.
5. If everything is ok commit the changes.
6. Have application deployed on pre-prod envirment.
7. Test application in pre-prod.
8. If any error/bug found goto step 1.
9. If everything is ok then the task is done.

Think of how unproductive it is. On even simple error, I would need to start the app and do some manual test to verify the change. 

Feedback loop is so lengthy, think how many corner cases any lazy or non-lazy engineer would skip considering.


## Unit Tests for Code Review
I then joined a new team with new project. When I worked on my first task in the new team, I followed the old lengthy cycle and had a code ready to commit. I commited the code, and as requested by new teammates I opened a code review. 

My new teammates were like

![We don't do that here](/assets/article_images/2020-08-04-testing/dont-do-it-here.jpg)

They asked me to write unit tests for code changes. I wasn't shocked, by this time I had learned writing Unit tests syntactically. I wrote the unit tests and commited the code.

Now I had to follow writing unit tests for all the code changes.
My development cycle became: 
1. Make a code change
2. Start the application locally.
3. Open web app in browser and go through pages or hit some URL so that code flows through my changes.
4. If any error/bug is found stop the app and goto step 1.
5. **If everything is ok then write unit tests**.
6. Then commit the changes.
7. Have application deployed on pre-prod envirment.
8. Test application in pre-prod.
9. If any error/bug found goto step 1.
10. If everything is ok then the task is done.

Guess what, writing Unit tests just added an extra step. Anybody would think it unproductive, I too considered it really unproductive.

## Unit Tests to find bugs early
On writing more unit tests, I started realising that they are actully helping me to find my mistakes and errors. So I thought why not write them before starting the application because that should definitely help me find bugs early. 

Then I moved writing unit tests to ealier part in my cycle. While working on typical layered architecture, I started working on one layer at a time, write a code for a layer and write tests for that layer, then move on to another layer. 
This helped me to find bugs way early and fixing them early made my development faster.

With this, starting application locally was just a double check more to test the integeration of parts. I also would do either of the local or pre-prod testing because of the confidence my tests gave me.

So the new cycle became:
1. Make a code change
2. **Write Unit tests.**
3. If bug is found, go to step 1 to fix the code.
4. Start the application locally.
5. Open web app in browser and go through pages or hit some URL so that code flows through my changes.
6. If any error/bug is found stop the app and goto step 1. (Very less likely now)
7. Then commit the changes.
8. Task is done.

The quick code-test cycles with faster feedbacks helped me to find bugs very early.

## TDD
On my path to explore importance of Unit Tests, I was exposed to the art of TDD.
TDD stands for Test Driven Development.

TDD has cycle of following steps for writting a code:
1. Write a failing test
2. Write a production code fix the test.
3. Refactor the code.

TDD is a bigger topic, which demands a post of itself.

Many people I know still consider Unit Tests as a more code which takes more time to write.
They fail to see advantages in terms of confidence one gains, and smaller feedback cycles which in tern increases the productivity.

~ Happy Unit Testing.