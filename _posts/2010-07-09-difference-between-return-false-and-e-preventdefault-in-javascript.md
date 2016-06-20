---
layout: post
title: "Difference between “return false” and “e.preventDefault()” in JavaScript"
excerpt_separator: <!--more-->
description: Understand the inside between return false and e.preventDefault
tags:
    - javascript
---

### Requirement:

We have a space and a link inside it! Clicking on the blank space will change the background color and the anchor will navigate to google.com if the blank space color is BLUE

Let us consider a code shown below:

{% highlight html %}
<div id="changeColorOnClick" onClick="changeMyBackgroundColor()">
   <a id="visitGoogle" href="http://www.google.com" 
      onClick="if(isParentBgColorBlue() == false) return false;">Visit Google</a>
</div>
{% endhighlight %}

`return false;` on a DOM event stops propagating the event to the parent and does not allow to execute the default behavior (in our case navigating href).

Now, requirement changed! Click on the link will also change the background color of the holder, and visits google.com when color is BLUE.

We want not to execute the default behavior and keep propagating on the event stack. We can just replace `return false;` with `e.preventDefault()`. We just preventing the default behavior which is the href navigation.

In other words, we can write the code below instead of `return false`

{% highlight javascript%}
e.preventDefault();
e.stopPropagation();
{% endhighlight %}