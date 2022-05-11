---
layout: post
title: Directive to set a scope variable on scroll finish of a div/text area
date: 2014-03-27 05:18:47.000000000 +05:00
tags:
- AngularJS
status: publish
type: post
published: true
author:
  login: mohammadumairkh
  email: mohammad.umair.k@gmail.com
  display_name: mohammadumairkh
  first_name: 'Mohammad Umair'
  last_name: 'Khan'
---
I came across a problem today in which I had to enable some options (which are disabled on page load) once the text area/div which has a scroll attached to it has reached the end. Usually such are the requirements when you want the user to scroll through or read the complete text before he can continue with whatever hes doing. Since it might become a requirement for other projects as well, I decided to wrap it up inside a small and neat directive. Here is how to use it:

![Directives usage](/assets/img/b2one1.png)

 By setting a variable, I mean setting it to a value **true**.

There are 2 simple steps to accomplish setting a variable from the directives parent scope once the scroll has reached its end:

1) Place a **uk-scroll** directive on the textarea/div. This would place a scroll listener on the tag with the required logic.

2) Provide the **uk-scroll** directive with a variable on the scope to set via the **set-on-scroll-complete** attribute.
Once the scroll has reached its end, the value of the variable on the scope, which is in the above case, **radBtn** would be set i.e. **radBtn = true**

Here is how the directive source looks like:

![Directives source](/assets/img/b2two1.png)

Hope this might come in handy. Keep the suggestions/recommendations coming in.

Source code can be found [here](https://github.com/mohuk/ukScroll)
