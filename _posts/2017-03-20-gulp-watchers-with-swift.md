---
layout: post
title:  "Using Gulp watchers with Swift"
date:   2017-03-14 15:11:24 -0500
categories: swift
---

One common workflow that I liked with Node, Ruby, and Python development is that when I change the source code in my application, it is immediately in effect. I wanted something similar for Swift on the server development, where I can set up watchers on my Swift files, and when they change, kick off another build and bring up another instance of the server. [Gulp](http://gulpjs.com/) is a great tool for this purpose. 

![Gulp watcher](/images/gulp/gulp.png)

# Basic Setup

Create a gulpfile.js in the root of your project. In this example, we will assume the server sources are in ./Sources.

Add the necessary dependencies:

```javascript
'use strict';

var gulp = require('gulp');
var child = require('child_process')
```

Create a task runner for compiling your Swift code:

```javascript
gulp.task('compile:swift', function() {
    return child.spawnSync('swift', ['build'])
});
```

You can create a global variable that holds a reference to the server instance that is running.

```javascript
var server = null;
```

Create a task runner for bringing up and down your server. The following snippet for instance, assumes your executable is named Server:

```javascript
gulp.task('run:server', function() {
    if (server) 
        server.kill();
    server = child.spawn('./Server', [], {
        cwd: '.build/debug'
    });
    server.stderr.on('data', function(data) {
        process.stdout.write(data.toString());
    });
});
```

Next, you can set up a watcher for your Swift files:

```javascript
gulp.task('watch', function() {
    gulp.watch('./Sources/**/*.swift', ['compile:swift','run:server']);
});
```

You are done!

# Full web application example

For a fully web-enabled application, you might want more tools like Sass and Webpack integrated so that your stylesheets can be preprocessed and your ES2015 code transformed into compressed and compliant JavaScript for browers.

Take your existing gulpfile, and add some additional dependencies:

```javascript
var sass = require('gulp-sass');
var webpack = require('webpack-stream');
```

Next, you can set up a bunch of watchers like normal, for instance:

```javascript
gulp.task('watch', function() {
    gulp.watch('./Sources/**/*.swift', ['compile:swift','run:server']);
    gulp.watch('./src/client/sass/*.scss', ['sass']);
    gulp.watch('./src/client/app/*.jsx', ['webpack'])
});
```

Your Sass styles preprocessing can produce artifacts in ./public/css:

```javascript
gulp.task('sass', function() {
    return gulp.src('./src/client/sass/*.scss')
        .pipe(sass().on('error', sass.logError))
        .pipe(gulp.dest('./public/css'));
})
```

Your Javascript can be compressed using a [webpack configuration file](https://github.com/rfdickerson/KituraReactDemo/blob/master/webpack.config.js). 

```javascript
gulp.task('webpack', function() {
    return gulp.src('./src/client')
        .pipe(webpack( require('./webpack.config.js')))
        .pipe(gulp.dest('public/'));
});
```

Add to your Kitura application a static file server:

```swift
router.all("/", middleware: StaticFileServer(path: "./public"))
```

Checkout a fully working example that uses this in action, here:



[KituraReactDemo](https://github.com/rfdickerson/KituraReactDemo)
