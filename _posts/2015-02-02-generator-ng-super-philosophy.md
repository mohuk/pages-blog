---
layout: post
title: Generator ng-Super Philosophies
date: 2015-02-02 05:18:47.000000000 +05:00
tags:
- AngularJS
status: publish
type: post
published: true
author:
  first_name: 'Mohammad Umair'
  last_name: 'Khan'
---

There are tons for [Yeoman](http://yeoman.io/) generators for Angular JS. Some of them are really good, some of them completely missing the
point. One of the generators is [Generator Angular](https://github.com/yeoman/generator-angular) which was love at first
sight but after using this on a large front-end application, I found it problematic on its application
structure. So I decided to pull up a generator for Angular with how I would want my application structure to be. Not all
are my ideas, I got a good chunk of guidelines from [John Papa's](https://github.com/johnpapa) [AngularJS Style Guide](https://github.com/johnpapa/angularjs-styleguide) and his course on AngularJS Patterns: Clean Code on PluralSight.

So I wired up a generator, [ng-Super](https://github.com/mohuk/generator-ng-super). I compiled all this goodness in [Yeoman](http://yeoman.io/) because I feel Yeoman makes the workflow very easy and intuitive.

Here is my attempt to explain the philosophies behind [ng-Super](https://github.com/mohuk/generator-ng-super) 

###**Folder Structure**

####**Root**

```bash
├── app           
├── bower.json    
├── configLoader.js 
├── gruntfile.js
├── node_modules
├── package.json
├── tasks         
└── tests         
```

The root structure is pretty standard except for the tasks folder which contains each grunt task as a separate file as shown below:

```bash
tasks
├── bump.js
├── clean.js
├── compass.js
├── concurrent.js
├── connect.js
├── copy.js
├── html2js.js
├── karma.js
├── ngAnnotate.js
├── replace.js
├── usemin.js
├── useminPrepare.js
├── watch.js
└── wiredep.js
```

This keeps maintenance of Grunt configurations very easy. Instead of managing one long/big Gruntfile, we can easily manage each configuration in a separate file.

####**app/src**

```bash
├── app.module.js
├── common
│   └── common.module.js
├── core
│   ├── core.module.js
│   ├── restangularConfig.js
│   └── routerConfig.js
└── welcome
    ├── WelcomeCtrl.js
    ├── welcome.html
    └── welcome.module.js
```

Application source is divided *by-feature* instead of *by-type*. The directory structure is designed to follow LIFT which is:

- **L** -- Locatable files
- **I** -- Identifiable at a glance
- **F** -- Flat directory structure 
- **T** -- Try to stay DRY

Each module is self contained and is designed with a single responsibility. 

**core** is the core application code, e.g. HTTP Interceptors, global router configurations. It also depends on all the other modules of the application, so this is how the application is glued up. **core** is available to all other modules of the application.

**common** contains the common functionality of the application e.g. services like calendar or widgets in directives. This is the cross-cutting layer and like **core**, **common** is available to all other modules of the application.

**welcome** is a feature module and contains all the relevant code that needs to be inside this feature including its controllers, factories, views and even routes. Each new module is added as a separate folder similar to **welcome**.

###**Styling**

```bash
├── css
│   └── main.css
├── main.scss
└── partials
    ├── _skin.scss
    └── _welcome.scss
```
    
Styles are done through **SCSS** as it provides numerous benefits on top of vanilla CSS. Details of all benefits might not be relevant here but one feature i.e. breaking up **SCSS** into separate files is worth a mention. Each file in the **partials** folder contains styling for a single purpose. This keeps our **SCSS** from getting messier and makes it easier to scale and maintain.

###**Third Party Components**

- Grunt
- Restangular
- Twitter Bootstrap
- Angular UI Router
- Angular UI Bootstrap
- Lodash

###**Coding style**

####**Controller**

```javascript
(function(){

  'use strict';

  angular.module('app.welcome')
    .controller('WelcomeCtrl', WelcomeCtrl);

  /* @ngInject */
  function WelcomeCtrl() {
    var vm = this;

    vm.welcomeMessage = 'ZE GENGO !';
  }

}());
```

**Controllers** use the Controller-As syntax which in real is just syntactic sugar but helps avoid scope soup and unnecessary usage of watches, event listeners and emitters. It also keeps things very clean.

####**Factory**

```javascript
(function(){
  'use strict';

  angular
    .module('app.welcome')
    .factory('messages', messages)

  /* @ngInject */
  function messages(){
    var service = {
      testFunction: testFunction
    }

    return service;

    ////////////////////

    function testFunction () {
      console.info('This is a test function');
    }
  }
}());
```
**Factory** is designed in similar fashion. With the service object defined above and constituent functions defined below as function expressions, help identify what functions are exposed by this **factory** at a glance.

Other parts of the application are similarly designed.

###**Grunt tasks**
####**$ grunt server**
Pops up a development instance of the Angular JS application. It is configured with **Livereload** of HTML, CSS and Javascript to speed up the development flow. It concurrently runs compilation of CSS from SCSS.

####**$ grunt bump**
Bump application version and goodies, details at [grunt-bump](https://github.com/vojtajina/grunt-bump)

####**$ grunt wiredep**
Iterates over all **Bower** dependencies (CSS and JS) and add them to the **index.html** file

####**$ grunt test**
Runs all tests in the ```/tests``` directory with Karma and PhantomJS

####**$ grunt build**
Produces a distributable AngularJS web application folder. The final distributable has 

 - Minified and concatenated CSS 
 - Converts all HTML templates to Javascript for speedier template serving
 - All AngularJS source is annotated, concatenated with templates and minified to produce a single, very tiny sized source file


I think you should try it out and let me know if you have any suggestions.

 ~Cheers


