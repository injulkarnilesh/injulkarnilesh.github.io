---
layout: post
title:  "Beyond Legacy Code - David Scott Bernstein"
date:   2020-12-25 14:34:25
categories: book
tags: book legacy code development agile
image: /assets/article_images/2020-12-25-medium-migration/software-development.png
image2: /assets/article_images/2020-12-25-medium-migration/software-development.png
---
>  This post has been migrated from medium.com/@injulkarnilesh.

The book is not about how to write code or build an application; but about general principles and practices for the software industry.

## Legacy Code

Legacy means something great of the dead that remains influential, in good terms. But in the software industry, we use the “legacy” word, politely, for a code that has lost all sense of vitality even though it is running.

In simple words, legacy code is the code that for different reasons is difficult to fix, enhance and work with.
>  Virtually all the software that is there in production is effectively legacy code.

The software industry hasn’t put enough value on maintainability; in most cases, businesses end up spending more on maintaining the code than on writing it. Most of the credit, or blame, for this, go to software development practices that industry has been following for many years — Waterfall Development.

Things have started to change with agile development methodologies like extreme programming (XP), where instead of trying to figure out everything up front, we figure out things as we go; designing, building and testing little bits of software at a time.

Michael Feathers defines legacy code as any code without tests. Because he puts a great value on automated unit tests that validate the code doing the intended. Having good unit tests presupposes that you have a good, testable code. But making untestable code testable is not easy a task as it may involve re-architecting the entire system.

## Something is wrong

There are so many companies stuck in a death spiral because of their software being developed are getting unmanageable and unworkable. Many of them are moving slowly towards agile practices, understanding where they had been going wrong.

The software industry has been following the Waterfall Model designed by *Winston Royce* who borrowed it form manufacturing and construction industry. But the industry failed to notice the warning he wrote that it wouldn’t work. The waterfall development process involves stages of Requirement gathering, Design, Implementation, Integration, Testing, Installation and of course Maintenance.

The biggest problem with the waterfall model is the long feedback loop for developers to know if their piece of code works or not. They have to wait for their written code to go through integration then the testing, only then would they come to know of any issue they had in their code. And this made developers believe :
>  It’s not my job to find the errors, it’s my job to create them.

We as an industry also need to stop considering software development as a formula to be followed in exact details, but as recipes open to creative interpretation of individuals to be adjusted according to the specific situations. This is because the problems we face are so diverse that we can not have a single solution for them all. When we will start considering software development as a creative process, we, especially the managers, will stop imposing the processes because the **process cannot dictate creativity.**

Managers feel comfortable with numbers, charts, and deadlines. This has led to a so-called predictable model like a waterfall which gave them a sense of progress knowing which phase the process is in. But such estimates gave them an illusion of progress, a dangerous one.

When developers get asked progress of the work they have one of these answers: *finished, not yet started, or almost done. :)*

In the software industry, there is no generally accepted body of knowledge that developers are expected to have, no industry-wide standards or practices. This is partly because the industry faces a huge diversity of problems and diversity of the ways developers approach the problems.
>  Knowing programming language doesn’t make you a software developer, just like knowing a written language doesn’t make you a writer.

In the civil industry, there are agreed-upon standards as to how good the construction is; if we keep aside the aesthetics of it. There are almost no chances of having such concrete standards in the software industry because the domain itself is not ***concrete*** but is ***abstract*** and exists only virtually.

There are a seemingly infinite number of choices we can make at any step of the software development process. Understanding the best trade-offs for a given situation involves understanding what we gain and what we lose with each option.

These challenges that the industry faces are also opportunities for us to see them and work towards a common goal of betterment of the industry. We should be challenging our fundamental assumptions and device better ways of working. This might force us to ask some difficult questions, raise above our beloved opinions and beliefs as to what works.

### CHAOS

The Standish Group, a research organization that studies the software industry, have been releasing rolling data under CHAOS report as a result of their study of 3400 projects over 10 years. This report categorizes studied projects under

“**Successful**”: Projects released with all features, on time, within budget.

“**Challenged**”: Projects completed, but with fewer features or out of budget than estimated.

“**Failed**”: Never was released, maybe canceled for whatever reason.

The data is shocking

![Credit: [https://blog.projectconnections.com]](/assets/article_images/2020-12-25-medium-migration/chaos.png)

It shows, around 27% are the chances that Software project completes successfully and 24% changes that it will never see the light of day.

But it depends a lot on how they have defined the criterion for the success and failures, which does not consider all the cases. The successfully completed project can be a failure when actually launched to the market, or a failed project could be incomplete due to external factors like business priorities, laws, etc.

However, it is true that the newly started Software Development project will be completed successfully still stands a very small chance. There are multiple [cases](https://www.washingtontimes.com/news/2014/apr/29/obamacare-website-fix-will-cost-feds-121-million/) where maintainability of projects should also be considered for majoring success of projects.

Even though we consider most of the reasons for the failure of the projects are external and are beyond the control of Software Development teams, we sure can blame the teams or their practices for projects in Challenged state. We can broadly consider three key factors that drive the low success rate :

**Changing Code.**

If the software is being used, it will receive change requests, that is inevitable. But are software built to accommodate change requests? Mostly not. Many a times teams maintaining the codebase is different than the one who wrote it, the code works but is not extendible. Making a change into the existing system requires reading and understanding existing code, but the code is generally not written keeping that in mind, making it more difficult to extend it leading new developers falling for the urge to rewrite the system. Many a time new features are forcefully added to the system some hackish way making system even worse. Also if the system does not have automated test suits, even the smallest of changes require re-testing of the entire system.

**Fixing Bugs.**

Software often fails because of bugs and efforts of finding and fixing the bugs. It’s not easy to develop a system where it is easy to find bugs or avoid introducing the ones in the first place. Developers write bugs like they write code, the difference being they aren’t paid to write bugs. Bugs often are just tips of the iceberg, finding and fixing them exposes ripple of changes required in the system and practices.

**Managing Complexity.**

80% of development costs are spent on identifying and correcting defects, that leaves only 20% of our budget to create value. A solution could be to do it right the first time, but it is not that straight forward mainly due to the ever increasing complexity of software systems. According to CHAOS, 20% of the features are commonly used while 45% of them are never used. Why then so many software create such unused features? Reasons vary from marketing teams just adding those features in their presentations and due to the waterfall model they had an only single chance of mentioning the features, to engineers overbuilding the features thinking “I can write it in few minutes only.”

### Cost of failure.

It has been estimated that **Software Defects** cost the US economy nearly $60 billion annually, it is bigger than 70% of the world’s 180 economies by GDP. Newer studies have even estimated higher numbers.

With up to 80% of the cost of software development happening after the initial release, companies can spend five times more to maintain software than they initially spent to build it. Roughly 60% of these costs go to enhancements while 17% goes to error correction. This high cost of maintenance is because we don’t value maintainability enough to make it a priority so we build software that’s risky and expensive to change and no one knows the economic impact of it.

## Agile

Toyota company in Japan had been following “Lean” practices focusing on Customer Values by eliminating waste. In the manufacturing industry, “waste” could be inventory that can be eliminated as it unnecessarily costs the company to warehouse it. Same “Lean” principle can be applied in Software Development where “waste” is any task that is started but not yet completed: work in progress tasks, anything that isn’t in software and anything that does not add value to customers.

This philosophy is shared by Agile software development which encompasses project management methodologies like Extreme Programming, Scrums, and Lean. The core of agile processes is the promise to
>  satisfy the customer through early and continuous delivery of valuable software.”

Rather than create more process to assure quality, they suggested less process so developers have more time to focus on applying solid engineering practices. They introduced technical practices such as TDD, pair programming which supports creating changeable software that was easy to deploy, maintain and extend.

### Smaller is better

Comparing software development with the race running, we should think of software development as a marathon that we don’t know the length of. If we don’t have good estimates of end of the race, it would be good to focus on short goals like next lap, or that next corner. By dividing the goal into such small goals we can focus and stay motivated; the same applies to Software Development where we can deliver small features in chunks like within 2 weeks.

Advantage of delivering in batches is that these small features are going through testing and integration within that small cycle giving developers a faster feedback loop.
> As developers themselves are going back to the same code in few weeks to add new features, this can drive them to write maintainable code.

But to be successful, teams should follow technical practices correctly; when those are ignored, improperly adopted or just misunderstood, we can end up with teams saying “We do scrums” and utterly fail to gain advantages of Agile.

### Agile Adoption

Any new technology goes through following phases of “Technology Adoption Life Cycle”:

* **Innovators** as first adopters

* Inspired by the success of innovators, **Early adopters** join in

* Once it has been seen working and is easy to use, joins the **Early majority**

* After it becomes mainstream, it gets **Late majority**

* Then comes finally **the** **Laggards** joining because no other option remains

**Agile** has hit through ***early adopters*** and crossing to the ***early majority***, but agile practices like **Extreme Programming** are still in ***innovators*** stage.

Although the Agile Manifesto states “continuous attention to technical excellence and good design enhances agility”, many believe technical excellence wasn’t stressed strongly enough. Technical excellence involves many things that take years of study and a great deal of focus to master.

## The Nine Practices

There was a time when the medical community did not believe *Ignaz Semmelweis* who was proposing that microscopic creatures could cause disease. But when during Civil War it was seen that more soldiers died because of infection after surgery than in actual battlefield, the community started to rethink its position on the germ theory. When we understand germ theory, we understand why we have to wash all the instruments. This is the difference between following a practice (the act of sterilizing a specific instrument) and following a principle (the reason for sterilizing all the instruments).

Due to the huge diversity in problems the Software industry works on and the equal diversity of developers working in the industry, we must have a shared understanding, a common set of practices and a shared vocabulary. We also must arrive at a common set of goals and be clear that we value quality and maintainability.

One of the goals we should set for the industry is to drop the cost of ownership of the software. It often costs 100 times more to find and fix bugs after software delivery than during requirement and design. For it, we should focus on maintainability of software we develop.

Expert software developers think about software development differently than the rest of us. They pay attention to technical practices and code quality. They understand what’s important and what’s not, they hold themselves to higher standards than the rest of us. Surprisingly they are not only neatest but also fastest developers as they pay particular attention to keeping their code easy to work with.
>  Expert developers wheren’t faster in spite of keeping code quality high; they were faster because they kept their code quality high.

In the physical world, high-quality products are generally costly; but in the virtual software world, a focus on quality is always less costly to execute in the long as well as short term. This does not mean software developers should never ever compromise on the quality of code, they can, but then should get back to it, fix it, before making further enhancements.
>  Software must be maintained so that it does not become a liability.

### Principles and Practices

Mastery involves more than skill and ability. The Japanese martial art Aikido defines three stages of mastery: Shu, Ha and Ri.

**Shu**: You know theories an Explicit Knowledge.

**Ha**: Once you know theory and principles, you put theory into practice.

**Ri**: Once a theory is known and is applied multiples times, with experience you get the mastery.

Same stages can be called out for software development. While building software you need techniques which broadly can be divided into Principles and Practices.

Think of principles are lofty goals, the things that we want to strive for because we know they are good and virtuous. Principles are important, but principles alone are not enough. We also need ways to achieve principles in practical situations and that’s what practices are for.

Practices must:

 1. Provide value at least most of the times,

 2. Be easy to learn and teach,

 3. Be simple to do, should come with time without thinking.

Writing code requires seemingly endless question-answer process, you need to question yourself a lot. Tough that is required, it could be tiresome sometimes. But if we have a general set of practices to follow to achieve desired principles we can help ourselves from continues questioning. For example, following a ***practice*** of removing duplicate code can help us achieve the Single Responsibility ***Principle***. You can teach yourself such practices.

### ‘Good’ in Software

How do you define a good code? We are less likely to get a consistent answer for such a question. For some, code(or software) doing what it is supposed to do and doing it efficiently might be “Good”; but for others, the readability and maintainability of it might be “Good”. Sometimes to achieve one quality you might need to trade off the other one.

External qualities like customer experience, lack of bugs, efficiency, etc. are symptoms of the software’s internal qualities that user’s don’t directly experience.

If developers consider development as a one-time activity they are less likely put any emphasis on maintainability and changeability; but if they think of it as something very likely to change, they would find better ways of achieving these internal qualities.

So, developers must focus on standards and practices that support the internal code qualities which make software easier to work with.

Even for internal qualities, we can’t reach industry-wide consensus for what is “Good”, as we are relatively young industry. But we definitely should strive for defining this.

For achieving this internal quality of software David Scott Bernstein has come up with the following 9 practices for developing better software:

 1. Say What, Why and for Whom before How

 2. Build-in Small Batches

 3. Integrate Continuously

 4. Collaborate

 5. Create a CLEAN Code

 6. Write the Test First

 7. Specify Behaviors with Tests

 8. Implement the Design Last

 9. Refactor Legacy Code

More on these practices later.
