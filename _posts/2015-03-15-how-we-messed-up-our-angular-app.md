---
layout: post
title: How we messed up our Angular App
date: 2015-03-15 04:59:17.000000000 +05:00
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
Back in July of 2013, it happened and we were writing an Angular app. Working on Angular JS for the first time, we made some choices which are worthy of entering the *Hall of Shame*.

Inspired by Dave Smiths talk (by the same title as this post) at the NG-NL Conf, I want to share exactly how **"we"** messed up our Angular app so you **don't**.

### 1. Using Yeoman's Generator Angular:

#### What we did:
The team decided to use a Yeoman Generator with the most number of downloads. The clear winner was [generator-angular](https://github.com/yeoman/generator-angular). A year into development with [generator-angular](https://github.com/yeoman/generator-angular), we realized that the file and folder structure offered by this generator does not scale. The application was fairly large and it felt like a mess. With all the development done, we could not go down the refactor lane and the application still feels like a *Ferrari engine plugged into Fred Flintstone's car*.

![Generator Angular Folder Structure](/assets/img/20150315/generatorAngularFolderStructure.png)

#### What we should have done:
A great way to structure medium to large scale applications is breaking them into features and modules. It gives you the following benefits:

- Makes it easier to find code and maintain
- Encourages code reusibility

![Improved Folder Structure](/assets/img/20150315/improvedFolderStructure.png)

### 2. Abusing the **$rootScope**

#### What we did:
Whenever we had to put some global state variables like some messages or logged in user information, **$rootScope** was our best friend unless we found out its true colors. From there on, things became insanely difficult to maintain. Remember, anything on the **$rootScope** is prototypaly inherited into each **$scope** across the application and as a result of this, we had no clue who is changing what.

![rootScope Abuse](/assets/img/20150315/rootScopeAbuse.png)

#### What we should have done:
Services are they way to go here. We should have exposed appropriate methods to get or set the data we need. This is a more structural and reliable way of approaching the problem of global application states. Also, there would be no variable shadowing on the inherited scopes from the **$rootScope**.

### 3. Bloated **$scope** inside controllers

#### What we did:
With all the magic we found **$scope** does, the adrenaline rush made us put private functions, private  variables, reuseable behavior on the **$scope**. Problem with putting everything on the **$scope** is that it bloates it creating a **$scope** soup.

![scope Bloat](/assets/img/20150315/scopeBloat.png)

#### What we should have done:
We were late to understand the purpose of the controllers and the **$scope**. The things which need to be bound on the view should be on the **$scope**, rest should either be private variables or in terms of reuseable logic, it should be in services.

### 4. Forcing controller inheritance

#### What we did:
I found that there is a very strange way to inherit controller behavior in one of our files. I still fear to refactor and fix it as the application is live.

![whaaaaaaat?!?](/assets/img/20150315/controllerInheritance.png)

#### What we should have done:
If there is any reuseable behavior we need, it should be in the services, I dont feel there is a need to explain this anymore.

### 5. Not namespacing directives

#### What we did:
Directives are a world of their own. There are a tons of things you can do with them but things as subtle as their name can cause lots of readability and debugging issues. We could not differentiate between native HTML5 tags and custom directives.

![Not Namespacing Directives](/assets/img/20150315/directiveNamespacing.png)

#### What we should have done:
Prefixing directives with a two letter acronym of your application eases this. All we needed was name **field** directive as **fpField**. Then, it can be used as **fp-field** on the markup and at a glance the developer can recognize that this is a custom directive.

### 6. Not using HTTP Interceptors

#### What we did:
To handle error responses, we made a function on a service and passed that function to the error handler of every **$http** request. This violates [DRY](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself) and was a nuisance to add with every request. We could have wrapped **$http** in a service and chained the error handler there, but thats not pretty either.

![Generic Error Handlers](/assets/img/20150315/httpInterceptors.png)

#### What we should have done:
We just needed to use an **$http** error interceptor and pass the same function to it. It is more centralized and maintainable approach to error handling **$http** requests.

How did you mess things up?




