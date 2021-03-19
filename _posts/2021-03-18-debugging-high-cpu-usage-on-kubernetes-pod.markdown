---
layout: post
title:  "Debugging High CPU usage on Kubernetes pod"
date:   2021-03-18 14:34:25
categories: kubernetes
tags: featured kubernetes resource analysis 
image: /assets/article_images/2021-03-18-kubernetes/kubernetes.png
image2: /assets/article_images/2021-03-18-kubernetes/kubernetes.png
image-credit: SoftwareEngineeringDaily
image-credit-url: https://softwareengineeringdaily.com/
---

We had this Java Spring Boot based micro-service not doing any heavy lifting as such, but on whichever environment we deployed it, it always had close to 200% CPU usage.

For debugging the cause behind such an unexpected behavior, we followed following steps and came across a reason behind it.

## Checking CPU Usage
Where did we get to know the fact that the service had a high CPU?

For you can use `kubectl top` command to see the resource usage by pods.

{% highlight shell %}
$kubectl top pods -n a_team | grep the-service
NAME                                                  CPU(cores)   MEMORY(bytes)
the-service-5cdd47bc56-b2qlc                        2000m        1122Mi
the-service-5cdd47bc56-x7tx5                        1998m        1681Mi
{% endhighlight %}

We had 2 CPU cores assigned to each pod of the service.

Here you can see 200% CPU usage mentioned as ~2000m values for CPU cores.

## Taking the Thread Dump
In the output of kubectl top command response, you can see CPU usage of the-service being very high. Does not matter how many pods we ran, all had a very high CPU usage. As there was no load on the service, there must be something wrong happening within itself that was consuming the resources like this.

From this we knew we had to look at the thread dump of the service, but the app was running in kubernetes and the docker base image it was using did not have `jmap` in java that it was using. Neither could we install `jstack` like tool into the running pod.

We had another option though: to send kill -3 signal to the java process of the service which would not kill the app but would print the thread dump to the console/logs.

To capture logs have kubectl logs command running in one terminal.
Sample command looks like this:

{% highlight shell %}
$kubectl logs -f --tail 100 the-service-5cdd47bc56-b2qlc -n a_team
#a_team: namespace
#the-service-5cdd47bc56-b2qlc: one of the running pods of the service
{% endhighlight %}

To send kill signal to java process, we need to exec into the pod. 

For that in another terminal run:
{% highlight shell %}
$kubectl exec --stdin --tty the-service-5cdd47bc56-b2qlc -n a_team -- /bin/bash
{% endhighlight %}

Once into the pod bash, we need to look at processes running with just `top` command.
`top` command printed the Processes running in the pod.

{% highlight shell %}
$top
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
    1 app       20   0 9380672 832056  13020 S 197.9  1.3  26358:44 java
  ...
 9105 app       20   0   56192   1984   1444 R   0.0  0.0   0:00.01 top
{% endhighlight %}

In response of top command you can see the java process with PID 1 is running with CPU usage close to 200 percent.

Before we send kill -QUIT signal to this process, we need also to look at what threads were consuming the CPU at this level.

To show thread details of the top command, press SHIFT + H while top is running
{% highlight shell %}
shift + H on top
 PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
   50 app       20   0 9380672 832004  13020 R 99.7  1.3  13053:15 java
   26 app       20   0 9380672 832004  13020 R 99.3  1.3  13050:38 java
  ...
{% endhighlight %}

Here you can see Threads with PID 50 and 26 were taking close to 100% of the each CPU core.

Now that we have Thread and Process PID, we are ready to take a dump, a thread dump.

For that just run kill command as:
{% highlight shell %}
$kill -QUIT {Java Process PID, in our example 1}
{% endhighlight %}

When you hit this command from inside the pod, the terminal which is listening to the logs would just print the thread dump to the terminal.


## Finding the culprit
Once you have the dump, you need to find the threads which are causing the hight CPU usage.

For that convert PID of threads to HEX representation:
{% highlight shell %}
50  ->  0x32
26  -> 0x1a
{% endhighlight %}


Then find in the thread dump the thread stack with nid as these HEX values. Those are the threads you are looking for.

{% highlight java %}

"Thread-4" #17 prio=5 os_prio=0 tid=0x00007f33c5133000 nid=0x1a runnable [0x00007f33a02de000]
   java.lang.Thread.State: RUNNABLE
        at java.lang.Thread.sleep(Native Method)
        at com.ateam.theservice.MyClass.run(MyClass.java:73)
        at java.lang.Thread.run(Thread.java:748)

"Thread-10" #42 prio=5 os_prio=0 tid=0x00007f33c5096800 nid=0x32 runnable [0x00007f3359ff6000]
   java.lang.Thread.State: RUNNABLE
        at java.lang.Thread.sleep(Native Method)
        at com.ateam.theservice.MyClass.run(MyClass.java:73)
        at java.lang.Thread.run(Thread.java:748)

{% endhighlight %}

In our case, the Threads which were problematic where running a infinite loop with while(true), and inside they were waiting for a condition. Between these condition checks, threads were sleeping for some time. By mistake the sleep time was misconfigured to be 0 so the threads where actually never sleeping and running forever leaving CPU and us awake.