---
layout: post
title: "Inside Object Oriented JavaScript Explained (Part – 1)"
excerpt_separator: <!--more-->
tags:
    - oop
    - javascript
---

In most of our programming languages like C#, C++, Java etc, we use `this` keyword to denote current object we are working on.
We often make mistakes in JavaScript object-orientation using `this` keyword.<!--more-->

In many object-oriented JavaScript tutorials and classes we are taught to use JavaScript in this OOP way, for instance:

{% highlight javascript %}
function A (val) {
    /**
    * private field
    */
    var privateProp = "this is a private property";

    /**
    * public field
    */
    this.valueProperty = "This is a value";

    /**
    * initialization and constructor part
    */
    this.valueProperty = val;
    privateProp = "updated value with: " + val;

    /**
    * public property
    */
    this.getPrivateProperty = function () {
        return privateProp;
    }
}
var a1Obj = new A("this is a new value");
alert(a1Obj.getPrivateProperty());
{% endhighlight %}

So, how does the code above work?

In JavaScript, when we write

{% highlight javascript %}
var myFuncObj = new anyFunction();
{% endhighlight %}

what it does internally, look something similar to this:

{% highlight javascript %}
var myFuncObj = (function() {
    var returnObj = new Object();
    //somehow set returnObj to the next caller anyFunction
    //where returnObj will be accessible by this
    var retValue = anyFunction();
    if (retValue) {
        returnObj = retValue;
    }
    return returnObj;
})();
{% endhighlight %}

**Explanation:** This is somewhat a flavor of object instantiation in JavaScript. When we use `new` keyword to a function, JavaScript engine actually wraps the statement which looks like the above code. Firstly, a new JavaScript generic object is created, and then JavaScript engine calls the function you are intending to instantiate and set `this` = the new JavaScript generic object which makes the object accessible within the function we have given as `this` keyword. If our given function returns anything then it sets our instantiated variable with the returned value otherwise sets the generic object created.

In our function or class whatever you call, we are actually setting up properties inside the object supplied by the JavaScript engine, which is actually working as the **constructor** of the object supplied.

Alright, now you are thinking that how are those private fields are working where public methods have access to those private methods? It’s the JavaScript closures! Although I will try to write some short note about closures, but I highly recommend you to read [Jim Ley’s article on Closures](http://jibbering.com/faq/notes/closures/#clClose).

### Closures

In simple words, keeping variables alive in a function even if the function returned! For instance, if you have a nested-function which has access to the outer function’s variables, [JavaScript garbage collector](http://blogs.msdn.com/b/ericlippert/archive/2003/09/17/53038.aspx) keeps the variables alive until the nested-functions are collected by the GC.

Let me give you an example with a code

{% highlight javascript %}
function showHello() {
    var value = 321;
    function showAlert() {
        value += 123;
        alert(value);
    };
    //the function above is not called here
    //only the reference to the function
    //is returned
    return showAlert;
}
//invokes the showAlert function which is
//returned by the calling function
var showAlertReference = showHello();
//this will alert the code of showAlert function. Try it out!
alert(showAlertReference);
showAlertReference();
{% endhighlight %}

Explanation: So, we have a function `showHello` and a private variable and a nested-function where this function is a closure as it has reference to the outer scope’s members, and then we return the reference of the nested-function `showAlert` and preserving it in `showAlertReference` variable, and then we invoke `showAlertReference` which is only a reference to the nested-function `showAlert` having only the `alert` code in it. This is where closure comes to play, JavaScript engine keeps the variables alive that are used in the nested-function make them available for access. These variables kept alive are sent to the garbage collector along with `showAlert` function is sent!

Hope this helped! But you should consider seriously about the Jim Ley’s article I noted above!

### Back to private members

Private members are nothing but closures here! Look at the code below!

{% highlight javascript %}
function person() {
    var _name;

    this.getName = function() {
        //_name is available here! this is closure!
        return _name;
    }
    this.setName = function(value) {
        //the same instance of _name is also available here!
        _name = value;
    }
}

var p = new person();
alert(p.getName());
p.setName("Munim");
alert(p.getName());
{% endhighlight %}

We are actually simulating the private/public members/methods by using closures! Again, when `new person()` is called JavaScript engine wraps the code and instantiate generic JavaScript object and set the object which is accessible with `this` keyword inside the `person` function! As `_name` is not being set on the object passed as `this` and thus `_name` is just acting like a local variable of the function person and the methods (`getName` and `setName`) inside the person function are set to the object passed as this and due to closure, invoking the methods (`getName` and `setName`) can have access to `_name` variable which should have died when person function expired!

### Conclusion

Object orientation in JavaScript is not usual like any other languages and mainly OO is implemented in JavaScript in sort of functional programming way! Understanding and thinking in closures is highly recommended for OO design principles in JavaScript.