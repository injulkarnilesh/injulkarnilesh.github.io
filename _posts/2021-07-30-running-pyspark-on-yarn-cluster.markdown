---
layout: post
title:  Running Self Contained PySpark Package On Yarn Cluster
date:   2021-07-30 14:34:25
categories: spark
tags: spark python pyspark yarn HDP
image: /assets/article_images/2021-06-06-understanding-spark-architecture/spark2.jpg
image2: /assets/article_images/2021-06-06-understanding-spark-architecture/spark2.jpg
---
# Python packaging
Being new to Python and PySpark, and had to test PySpark feasibility on old Hortanworks Data Platform (HDP) cluster, I had many questions.
Having worked on Java, Spark I was expecting similar workflow for how we would run the PySpark application on the cluster. 
I assumed there would be a standard dependency management, packaging tool like Maven in Python world and a standard packaging format like .jar as well in Python. 

But when I started looking for them, I got to know there exists so many of the tools, not many of them are standard or are supported by specific python versions only.

There were tools like
* VirualEnv
* venv
* pipenv
* Poetry
* pip3

For dependency management, it made sense to stick with standard tool like `pip3` that comes with python, and with requirements.txt it made a little easier to manage reproducible dependencies. 

But pip3 was deploying those dependencies under site_packages of python installation. We wanted to package the dependencies along with the code, and deploy them together the way we were deploying fat jar for Spark.

To solve this problem, we had to rely on `environments`. In python, we create different environments for different projects so that different projects can use different versions of dependent libraries. And that is what we had to use to separate our project's environment from other local projects.

But luckily we can use environment to package it up as well. Because environment was just another directory created in local project structure, we can just zip it up and use it in the spark cluster, as it already has python (soft link or a real copy) and the required dependencies.

To create environments we decided to use *venv* as in recent python versions it comes bundled.
To package, though we had to use *venv-pack* library to package the environments so that those could be shipped to wherever we need them for running.

We used following commands to create new environment, install dependencies and then pack the environment.

{% highlight bash %}
$python3 -m venv testenv  #create testenv environment
$source testenv/bin/activate #activate env
$python3 -m pip install -r requirements.txt  #deploy required dependencies from requirements.txt in this env
$python3 test.py   # test code locally
$venv-pack -o testenv.tar.gz  #package current env into tar.gz file
{% endhighlight %}

Our requirements.txt
{% highlight bash %}
numpy
pyspark==2.3.2
venv-pack  #dev dependency only, ideally should not be included here.
{% endhighlight %}

After this we have our code in test.py and environment with dependencies zipped in a tar.gz file.

Following was the simple pyspark code that we wanted to run.

{% highlight python %}
from pyspark.sql import SparkSession
 
spark = SparkSession.builder.appName("Package testing").getOrCreate()
a = [1, 3, 4, 5, 6, 7, 5, 9, 12, 66, 77]  
rdd = spark.sparkContext.parallelize(a)
 
 
import numpy as np  #import numpy
sqrt = rdd.map(lambda x: np.sqrt(x))  #executor using numpy
 
for sq in sqrt.collect():
    print('SQRT ', sq)
 
print("Sum:", np.sum(a))  #driver using numpy
print("Count:", rdd.count())

{% endhighlight %}

Using numpy in test.py to print the sum of array elements was to test if numpy is available at driver node, as for driver it is just plain simple python code.

Using numpy inside map function on rdd was to test using numpy on executors. As this code would be run on executors in distributed manner.

# The Spark Side
Once we had code that we wanted to test, we were to figure out how to run it on Spark.

Spark is distributed big data processing framework where there are drivers and executors. 

We were running Spark on older HDP cluster, this cluster nodes had python2 deployed already and that scared us if we are stuck with python2 for our development. But no, luckily we can deploy python3 on nodes and let spark know which python to use with `PYSPARK_PYTHON` environment variable.

We deployed python3 on dev cluster's all nodes. Then from Ambari we set *PYSPARK_PYTHON* under `spark2-env` configuration file so that it is reflected on all the cluster nodes.

This Spark2-env files are used to set required environment variables for Spark applications. Once we had it updated on all nodes, we were good that *PYSPAK_PYTHON* would be available for both drivers and executors.

Next question we had was how Executor process (and Driver) would be able to access the dependent libraries like numpy?

After googling a lot, we came across approaches where the python environments were copied next to the code running. And *PYSPARK_PYTHON* was set to `.environment/bin/python3`.

Next thing we needed was to copy our environment zip next to our test.py when it runs on Spark cluster. For that we used `--archives` option of the Spark Submit.

{% highlight bash %}
$./spark-submit --master yarn --deploy-mode cluster 
--conf "spark.yarn.maxAppAttempts=1" \
--archives /location/of/testenv.tar.gz#environment 
/location/of/test.py
{% endhighlight %}

Parameters of spark-submit were:
* master: yarn
    * as were were running this on the HDP yarn cluster
* deploy-mode: cluster
    * to run spark application in cluster mode like how we would run in prod
* maxAppAttempts: 1
    * to fail early in case we had any failure, just a time saviour.
* archives : testenv.tar.gz#environment
    * this is set to location of the env we zipped.
    * zip file name is followed by #environment
    * with #environment, inside our cluster we get to refer to this zip with name environment.
    * that is why PYSPARK_PYTHON is set to path starting with .environment
    * this is extracted next to python file we run in the yarn container
* test.py
    * python code to run

# Debugging
Our early attempts were not fruitful, our code when run on cluster was complaining that it can not import 'numpy'.

For python code running on executors, or driver to find numpy, the directory containing the numpy installation should be in `sys.path`.

To test if this is the case, we printed values of `sys.path` and some other values to help us debug what actually was happening on the cluster.

Following code we used to print these debug variables:

{% highlight python %}
from pyspark.sql import SparkSession
 
spark = SparkSession.builder.appName("Package testing").getOrCreate()
a = [1, 3, 4, 5, 6, 7, 5, 9, 12, 66, 77]
 
def printPath(x):  #this was for debugging to print these variable from the executors.
    ar = sys.path
    ar.append('OS.PP ' + str(os.environ.get('PYSPARK_PYTHON')))
    ar.append('prefix ' + str(sys.prefix))
    ar.append('ex_prefix ' + str(sys.exec_prefix))
    ar.append('S.Ex ' + str(sys.executable))
    return ar
 
rdd = spark.sparkContext.parallelize(a)
 
rdd2 = rdd.flatMap(printPath)
 
for r in rdd2.collect(): # print debugging variables
    print('PATH:', r)

{% endhighlight %}

Earlier we were making some stupid mistake not worth mentioning here, but with above code we could see that `./environment/lib/python3.6/site-packages` was not included in sys.path; and after fixing the issue, we could see this in `sys.path` printed in above program. Once we saw it there, we knew our code should now work, and it did.

Another caveat was that the python included in the environment was just a soft link to `/usr/bin/python3`, in our case it was there in our local machine as well as the cluster nodes at the same location, so it worked well.

In your case if they are not same, you can try
* build your package from same OS, architecture as that of the cluster.
* try `--copies` option of venv common while creating env so that python is copied into your environment and not just soft-linked.


Though there were many articles on internet explaining these steps, unfortunately they failed to explain how it is supposed to work. We had to figure out the hard way. Hope you don't need to after reading this.