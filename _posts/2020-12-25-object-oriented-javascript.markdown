---
layout: post
title:  "Object Oriented JavaScript"
date:   2020-12-25 14:34:25
categories: javascript
tags: featured javascript object oriented
image: /assets/article_images/2020-12-25-medium-migration/got.jpg
image2: /assets/article_images/2020-12-25-medium-migration/got.jpg
image-credit: FilmDaily
image-credit-url: https://filmdaily.co/
---
>  This post has been migrated from injulkarnilesh.blogspot.com.

Though most widely used programming language over the Internet, JavaScript is very misunderstood and weird programming language.

JavaScript is loosely typed, object oriented, functional programming language where everything is an object.

## Types in JavaScript 
JavaScript has three primitive data types :
* Number
* String 
* Boolean

Then there are two special data types :
* Null 
* Undefined

And then everything else is an object. 
Number, String and Boolean values are immutable while objects in JavaScript are mutable.

## Object
Objects in JavaScript are simply the key value pairs, where value could be of any type; a primitive, an object or even a function.

A function contained within an object is generally referred to as a method.

A key in an object is called as a property which could be any String even an empty String, whereas property value could anything but undefined.

When objects are passed around to and from functions, they are passed by reference.

## Object Literals
One of the ways, object could be created in JavaScript is using Object Literals. Here we directly initialise object with properties and the values; we specify those inside pair of curly braces, { and }, separated by comma.

{% highlight javascript %}
var imp =  {
     first_name : 'Tyrion',
     'last-name' : 'Lannister',
     payDebt : function(){
          console.log('Lannisters always pay their debts');
     },
     trial : function(){
          console.log(this.first_name, 'demands trail by combat')
     }
}

console.log(imp.first_name);
console.log(imp['last-name']);

imp.payDebt();

imp.trial();

console.log(imp.girlFriend);

imp.girlFriend = 'Shae';
console.log(imp.girlFriend);

delete imp.girlFriend;
console.log(imp.girlFriend);
{% endhighlight %}

Output:
{% highlight javascript %}
Tyrion
Lannister
Lannisters always pay their debts
Tyrion demands trail by combat
undefined
Shae
undefined
{% endhighlight %}

property value could be any String, but if it is not a valid JavaScript variable name, like last-name in above example, then it should be mentioned within single or double quotes..
this is an object that refers to the current object. So inside Object literals it could be used to access other properties. 

Object property could be retrieved by using . (dot) notation or [ ] (array like notation). If property name is not a valid JavaScript name then you will have to use [] notation only, otherwise you can use . notation which seems more user friendly. But if you want to use a variable as a property name then [] notation could be used.

When tried to access not existing property, it returns undefined. 

In JavaScript, an object could be amended by just adding a properties latter on dynamically, or also by removing using delete operator.


## Constructor Function
JavaScript doesn’t have anything like Classes, but supports a way to create multiple objects sharing similar structure, template that the classes are used for. You can achieve that using a constructor function.

Constructor functions are same as that of normal JavaScript functions. The convention is to name it with first letter in capital; it’s just a convention not a syntax.

Generally constructor function sets fields and methods of objects to be created using this.
You can call the construction function just like normal function and if it sets values using this, then the values would be set to what this is referring to and that generally would be a global scope, window in case of browser.

{% highlight javascript %}
function Person(firstname, surname, house, age){
     this.firstname = firstname;
     this.surname = surname;
     this.house = house;
     this.age = age || 50;

     this.sayHello = function(){
          console.log(this.firstname, 'of house', this.house);
     }

     this.increaseAge = function(increament){
          this.age += increament;
     }

     this.print = function(){
          console.log(this.firstname, this.surname, '(', this.age, ')' );
     }
}

Person.prototype.die = function(){
     console.log(this.firstname, this.surname, 'dies');
}

var king = new Person('Joffrey', 'Baratheon', 'Baratheon', 14);
king.sayHello();
king.increaseAge(1);
king.print();
king.die();

var ned = new Person('Ned', 'Stark', 'Stark', 55);
ned.sayHello();
ned.print();
ned.die();

Person('Jaime', 'Lannister', 'Lannister');
console.log(firstname);
console.log(surname);
sayHello();
print();
{% endhighlight %}

Output
{% highlight javascript %}
Joffrey of house Baratheon
Joffrey Baratheon ( 15 )
Joffrey Baratheon dies
Ned of house Stark
Ned Stark ( 55 )
Ned Stark dies
Jaime
Lannister
Jaime of house Lannister
Jaime Lannister ( 50 )
{% endhighlight %}

If you want to create an object from constructor function, use new. Then in that case the this will be referring to the object being newly created. So, variables and methods set using this in the constructor function would be available for the newly created object.

In above example when Person is called without new, this is referring to global scope and so the properties firstname, house etc and functions sayHello, print etc. are available on global scope.

You can set functions at prototype level as well. That way also methods could be created for all the objects created with new on the constructor function. This is a part of prototype inheritance that we will see later.

We will see later what exactly happen when you call new on any function.


## Calling Function
There are three ways to call functions in JavaScript.
1. Calling using conventional () notation.
2. Using call method
3. Using apply method


{% highlight javascript %}
house = 'Stark';
function sayWords(words, expression){
     console.log('House', this.house, ':', words, expression);
};

sayWords('Winter is coming', ':)');
var lannister = {
     house : 'Lannister'
};
sayWords.call(lannister, 'Hear Me Roar!', ':|');

var baratheon = {
     house : 'Baratheon'
};
sayWords.apply(baratheon, ['Ours is the Fury!', ':(']);

console.log(typeof Function.prototype.call);
console.log(typeof Function.prototype.apply);
{% endhighlight %}

Output
{% highlight javascript %}
House Stark : Winter is coming :)
House Lannister : Hear Me Roar! :|
House Baratheon : Ours is the Fury! :(
function
function
{% endhighlight %}

In traditional () way, you just use () after the function name to call it. If you have arguments to pass, you pass those into the parenthesis separated by comma.

Apply and call are methods from Function.prototype, as all functions in JavaScript inherit prototypically from Function.prototype, these two methods are available on all the functions. With these two methods, you can pass in the value of this for the context of the function you are calling. That means, first argument to call or apply method is an object that acts as this within the function you are calling. 

Only difference between call and apply is how argument are passed to the function to be called. In apply we pass the arguments in the form of array. So second argument to apply is an array of arguments for the function that you are calling apply on.

In case of call, second onwards all arguments are used as the arguments for function that you are calling the call upon.


## New in JavaScript
When new is called on a construction function, a new object is created that is referring to the construction function (actually the prototype of it) through \_\_proto\_\_ property.
Every Object in JavaScript has property named \_\_proto\_\_ that refers to prototypical parent of that object. Also it is important to note that every function in JavaScript has prototype property which refers to a common object that is shared by all the objects created with new on the function.

{% highlight javascript %}
function Stark (firstName ) {
     this.firstName = firstName;
    
     this.print = function(){
          console.log(this.firstName, this.lastName);
     }
}
Stark.prototype.lastName = 'Stark';
Stark.prototype.say = function(){
     console.log('Winter is coming');
}

var arya = new Stark('Arya');
arya.say();
arya.print();

var robb = new Stark('Robb');
robb.print();
robb.say();
{% endhighlight %}

Output:
{% highlight javascript %}
Winter is coming
Arya Stark
Robb Stark
Winter is coming
{% endhighlight %}

When new is called in JavaScript on some function new object is created and returned. It happens in three steps.

When constructor function is defined it will have following structure. lastName and say would be added to Stark.prototype.

When Stark is called, it just sets firstName and print to whatever this will be referring to while execution.


![Javascript Function Defined](/assets/article_images/2020-12-25-medium-migration/function_defined.jpg)


When new is called as 

*var arya = new Stark('Arya');*

following three steps happen :

1) Empty JavaScript object is created with arya pointing to it.

2) Stark method is called using call or apply so as to use arya object as this, so that properties set to this from within Stark will be added to arya. I don’t know whether it’s call or apply. Just know that purpose is to have new object to be used as this, that is what first parameter of either of these two methods represents. 

So when called as Stark.call(arya, ‘Arya’), this function sets firstName and print as properties of arya object.

![New Called On Function](/assets/article_images/2020-12-25-medium-migration/new_function.jpg)

3) Next step is to set \_\_proto\_\_ of this new object to prototype property of constructor function that is Stark.prototype. As \_\_proto\_\_ property is used as a parental reference in prototypical inheritance, properties of Stark.prototype gets available to arya object as well.

![\_\_proto\_\_ of new object being set](/assets/article_images/2020-12-25-medium-migration/proto.jpg)

Final picture looks like this :

![Calling new on Constructor Function](/assets/article_images/2020-12-25-medium-migration/constructor.jpg)

Advantage of having methods at prototype of constructor function (e.g., say) is that only single copy of that function/method would exist and the same would be shared by all the instances. And if methods are added to the created objects using this (eg. print), then each created object will have its own copy though the same body.


## Inheritance in JavaScript
JavaScript supports prototypical inheritance, which is all different than conventional class based inheritance.

For this purpose every object in JavaScript has a property named \_\_proto\_\_, which points to the prototypical parent of given object.

Also every function has prototype property, which refers to the common object that every object created with a new on the function shares.

When you withdraw or get a property on an object, JavaScript checks if that property exists in that object, if yes then value of that property is returned. If the property is not found, JavaScript checks object referred by \_\_proto\_\_ property of that object, if object.\_\_proto\_\_  contains that property then its value is returned. If not \_\_proto\_\_ of this object is checked and so on until \_\_proto\_\_ is null on some object in parent hierarchy.  If the property that we are trying to get does not exist anywhere in the hierarchy, then it returns undefined.

But when you set a property and that object contains that property then value of that property is updated to new value. But while setting if that object does not contain the property, new property is added to that object with given value, without checking the prototypical hierarchy.

> So we can say, get is deep and set is shallow.

Using \_\_proto\_\_ and prototype properties JavaScript supports inheritance.

{% highlight javascript %}
function Person(name) {
     this.name = name;
     this.greet = function(greeting){
          console.log(this.name);
          this.say(greeting);
     }
}

Person.prototype.say = function(message){
     console.log('saying', message);
}

var littleFinger = new Person('Baelish');
littleFinger.greet('I did warn you not to trust me.');
littleFinger.say('Hello');

function Lannister(name){
     this.name = name;
     this.payDebt = function(){
          console.log(this.name, 'paying the debt');
     }
}

Lannister.prototype = new Person();
Lannister.prototype.trick = function(){
     console.log('Tricking someone');
}

var tyrion = new Lannister('Imp');
tyrion.payDebt();
tyrion.greet('I deemand trial by combat');
tyrion.say('Hi');
tyrion.trick();
console.log(tyrion.__proto__.name);
{% endhighlight %}

Output
{% highlight javascript %}
Baelish
saying I did warn you not to trust me.
saying Hello
Imp paying the debt
Imp
saying I deemand trial by combat
saying Hi
Tricking someone
undefined
{% endhighlight %}


Here we are setting Person constructor function as a prototypical parent of Lannister constructor function. That we do with 

{% highlight javascript %}
Lannister.prototype = new Person();
{% endhighlight %}

So the objects created with Lannister contains properties set by Lannister function, properties of Lannister.prototype, properties set by Person and properties of Person.prototype (and obviously Object.prototype).


![Inheritance in JavaScript](/assets/article_images/2020-12-25-medium-migration/inheritance.jpg)

In above case we have created Lannister as a child constructor function of Person constructor function. 

When we try to access some property on tyrion object, JavaScript checks if that property is available on following objects in given sequence :
1. tyrion
2. Lannister.prototype
3. Person.prototype
4. Object.prototype

Value of first matching property in above objects in given sequence is retrieved as a value of the property. If no match is found in entire hierarchy, undefined is returned.

The idea behind having this way of inheritance is to have \_\_proto\_\_ of child function’s prototype to point to Parent function’s prototype, so that properties of both the prototypes are available to object of child function. Also it is important to have properties set by parent function itself, and for that we have called new on parent function and value created is set to child function’s prototype.
This all is done in

{% highlight javascript %}
Lannister.prototype = new Person();
{% endhighlight %}

To understand how exactly this happens you need to know what happens when you call new on some function in JavaScript. 

Points to note :
1. We are setting Lannister.prototype to a new object, so anything added to Lannister.prototype before we call Lannister.prototype = new Person(); will be lost. So it’s important to add anything to Lannister.prototype (trick function for example) after we have set it to new Person.
2. Lannister.prototype.name will be undefined as we are calling Person constructor without parameters.

If you don’t want to use constructor functions for creating objects and still want to achieve inheritance, you can do that by just setting \_\_proto\_\_ of child object to parent object or by using Object.create() method, which does the same for you.


{% highlight javascript %}
var ned_stark = {
     lastName : 'Stark',
     say : function(){
          console.log('Winter is coming');
     }
};

var sansa = {
     firstName : 'Sansa',
     believeBlindly : function(){
          console.log('Once a fool, alway a fool.');
     }
}
sansa.__proto__ = ned_stark;
console.log(sansa.firstName);
console.log(sansa.lastName);
sansa.say();
sansa.believeBlindly();

var arya = Object.create(ned_stark);
arya.firstName  = 'Arya';
arya.beBrave = function(){
     console.log('Being brave');
};
console.log(arya.firstName);
console.log(arya.lastName);
arya.say();
arya.beBrave();
{% endhighlight %}


Output:

{% highlight javascript %}
Sansa
Stark
Winter is coming
Once a fool, alway a fool.
Arya
Stark
Winter is coming
Being brave
{% endhighlight %}

The difference is that to explicitly set \_\_proto\_\_ of child object, object could be created earlier and set other properties, but while using Object.create() method, which creates new object with \_\_proto\_\_ being set to given argument and returns it, you need to set other properties after it is created with Object.create method.

The methods available to all the functions like call are at Function.prototype as this is at prototypical hierarchy of all the functions in JavaScript. Similarly all common methods available for all the objects in JavaScript are available at Object.prototype as this is at root of prototypical hierarchy.

This could used appropriately to have methods available to all the functions of all the objects. 


## JavaScript for in
for in loop in JavaScript iterates over all the properties of object including the ones available from prototypical inheritance.


{% highlight javascript %}
function Stark (firstName ) {
     this.firstName = firstName;
    
     this.print = function(){
          console.log(this.firstName, this.lastName);
     }
}
Stark.prototype.lastName = 'Stark';
Stark.prototype.say = function(){
     console.log('Winter is coming');
}


var robb = new Stark('Robb');

for(var robbProperty in robb){
     console.log(robbProperty, typeof robb[robbProperty], robb[robbProperty]);
}
{% endhighlight %}

Output
{% highlight javascript %}
firstName string Robb
print function function (){
console.log(this.firstName, this.lastName);
}
lastName string Stark
say function function (){
console.log('Winter is coming');
}
{% endhighlight %}

Notice that var robbProperty refers to property in robb object not the values of properties. So you need to use robb[robbProperty] to access properties.

If you want to restrict the loop to actual properties of robb object only, skipping the properties available from inheritance, then you can use hasOwnProperty method of objects which return if the given property is its own.


{% highlight javascript %}
for(var robbProperty in robb){
     if(robb.hasOwnProperty(robbProperty)){
          console.log(robbProperty, typeof robb[robbProperty], robb[robbProperty]);
     }
}
{% endhighlight %}

Output:
{% highlight javascript %}
firstName string Robb
print function function (){
console.log(this.firstName, this.lastName);
}
{% endhighlight %}