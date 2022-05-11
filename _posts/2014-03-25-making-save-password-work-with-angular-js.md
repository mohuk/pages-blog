---
layout: post
title: Making save password work with Angular JS
date: 2014-03-25 04:59:17.000000000 +05:00
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
With all the power Angular JS possesses, it has succumbed to a rather simple feature, that is the save password on the browsers. If you hit the 'Save Password' on the browser after filling in your credentials, the browser stores the information and restores it each time you visit the log in page so you can just press log in or hit enter to sign into the application. Trying that with Angular JS fails unless you add and remove a character to any input tag and try again, here is why ...

Angular JS uses the 'ng-model' directive to bond the view to a scope variable. Every time we write on the input tag, Angular invokes its digest loop and updates all the views and models (see documentation of NgModelController) by invoking different functions etc. Whenever the browser populates the saved credentials the first time, it does not invoke the digest cycle of Angular and hence none of the values that are in the view are updated on the model. Since nothing is on the scope, the login attempt would eventually fail. As mentioned above, adding and/or removing a character invokes the digest cycle of Angular doing all the dirty checking on scope variables, eventually updates the models and login request would proceed as expected.

Here are a couple of simple directives which would make auto-save work like a charm

![Directives usage ](/assets/img/b1one.png)

Above HTML shows a simple login form. It highlights 2 directives:

1) **fp-model-update-on** with an attribute **submit**

This directive would listen for a submit event and update the model on which it is placed

2) **fp-submit**

This directive would fire a submit event on click of whatever element it is placed on

Pressing enter or clicking submit button would trigger a **FormSubmitted** event, each model which has **fp-model-update-on** directive with submit as an attribute would update itself before the login request could be made. Once the models are updated, the login request can proceed as expected.

Here is how the directives looks like:

![Directives source ](/assets/img/b1two.png)

Simple isn't it? Plus you can add any number of attributes and can trigger update on models on that event. Hint: blur enter etc

Questions and suggestions are welcome

Thanks
