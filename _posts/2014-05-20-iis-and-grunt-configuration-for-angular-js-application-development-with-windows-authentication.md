---
layout: post
title: IIS and Grunt configuration for Angular JS application development with Windows
  Authentication
date: 2014-05-20 06:46:23.000000000 +05:00

tags:
- AngularJS
- Windows Authentication
status: publish
type: post
published: true
meta:
  _edit_last: '65775760'
  geo_public: '0'
  _publicize_pending: '1'
author:
  login: mohuk
  email: mohammad.umair.k@gmail.com
  display_name: mohuk
  first_name: 'Muhammad Umair'
  last_name: 'Khan'
---
There are times when one works with a 'not so popular' stack of technologies. The stack which I am using for development is one of those. We have a SPA based on Angular JS which consumes .Net Web API through REST services. The icing on the cake is, it uses Windows Authentication, since it is an app designed to be used on the intranet. I had a few hiccups on setting up the development environment mainly due to:

How to serve my Web API and SPA through IIS with Windows Authentication
How to configure livereload of html, css and javascript during development while it is all being served through IIS (I hate pressing F5 over and over again). My custom Yeoman generator is used to doing things connect-livereload way. It was hard for me to configure livereload dynamically through grunt so I had a tough time setting up a development environment which uses livereload for CSS and javascript files and at the same time use the app hosted on IIS rather then using the connect plugins.

Here is how I addressed these issues:

### Problem 1:
I had a completely separate Angular JS client directory and I had to consume a .Net Web API and serve it through IIS so it had to be a part of the same website. After several attempts of structuring, I added a virtual directory to my site on IIS and pointed it to my client directory and aliased it as 'content'. In easier words, my website contained a folder 'content' which contains my client application as a shortcut. Note that the client application physically existed somewhere else. Then I used IIS's URL Rewriting to rewrite the root of my website to '/content'. Similarly any hit to request any static file (html, css, png, ico, js etc) was configured to be rewritten to '/content'. Here is a snippet from my web.config file and the URL rewriting rule.

```xml
<rewrite>
  <rules>
    <rule name="SSL_ENABLED" enabled="false" stopProcessing="true">
      <match url="(.*)" />
      <conditions>
        <add input="{HTTPS}" pattern="^OFF$" />
      </conditions>
      <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" appendQueryString="true" redirectType="Permanent" />
    </rule>
    <rule name="Rewrite root to content" stopProcessing="true">
      <match url="^$" />
      <action type="Rewrite" url="/content/index.html" />
    </rule>
    <rule name="Rewrite static files to content" stopProcessing="true">
      <match url="(?:\.css|\.js|\.html|\.ttf|\.jpg|\.png|\.gif)$" />
      <action type="Rewrite" url="content/{SCRIPT_NAME}" />
    </rule>
  </rules>
</rewrite>
```

### Problem 2:
After a lot of thinking I came up with a rather obvious solution to this problem :

configure grunt-contrib-watch to watch for files and reload them on the browser through watch's livereload configuration
configure SCSS to CSS compilation through watch's compass block and configure it for livereload
open the hosted app off IIS through grunt
get rid of the connect-livereload all together and place the livereload script on my index.html

By the above steps, I can do all the IIS page serving and authentication and still use grunt for my tasks. Do read grunt-contrib-watch to understand how it works.

Here is a glimpse of what the gruntfile looks like. Note, you have to place your hosted port on the WEB_PORT constant in the gruntfile.

```javascript
'use strict';
//place the port of your IIS host
var WEB_PORT = 1982;
var LIVERELOAD_PORT = 35729;

module.exports = function (grunt) {
  // load all grunt tasks
  require('matchdep').filterDev('grunt-*').forEach(grunt.loadNpmTasks);

  // configurable paths
  var yeomanConfig = {
    app: 'app',
    dist: 'dist'
  };

  var webConfig = {
  	PORT: WEB_PORT
  }

  try {
    yeomanConfig.app = require('./bower.json').appPath || yeomanConfig.app;
  } catch (e) {}

  grunt.initConfig({
    yeoman: yeomanConfig,
    webConfig: webConfig,
    watch: {
      options: {
        nospawn: true
      },
      compass: {
        files: ['<%= yeoman.app %>/styles/{,*/}*.{scss,sass}'],
        tasks: ['compass:server'],
        options: {
          livereload: LIVERELOAD_PORT
        }
      },
      livereload: {
        options: {
          livereload: LIVERELOAD_PORT
        },
        files: [
          '<%= yeoman.app %>/{,*/}*.html',
          '{.tmp,<%= yeoman.app %>}/styles/{,*/}*.css',
          '{.tmp,<%= yeoman.app %>}/scripts/{,*/}*.js',
          '<%= yeoman.app %>/images/{,*/}*.{png,jpg,jpeg,gif,webp,svg}'
        ]
      }
    },
    open: {
      server: {
        url: 'http://localhost:<%= webConfig.PORT %>'
      }
    },
    //....
    //lots of other grunt tasks
    //....
  });

  grunt.registerTask('server', function (target) {
    if (target === 'dist') {
      return grunt.task.run(['build', 'open', 'connect:dist:keepalive']);
    }

    grunt.task.run([
      'clean:server',
      'concurrent:server',
      'open',
      'watch'
    ]);
  });

};
```

Now, a simple `grunt server` will pop up your application hosted on IIS with all the grunt and livereload magic.

Happy coding :)
