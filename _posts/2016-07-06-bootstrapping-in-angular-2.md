---
layout: post
permalink: /bootstrapping-in-angular-2
title: Bootstrapping in the browser with Angular 2
author: toddmotto
path: 2016-07-06-bootstrapping-in-angular-2.md
tags: bootstrapping
version: 2.0.2
intro: In this guide you'll learn how to bootstrap an Angular 2 application.
---

Angular 1.x allows us to bootstrap our applications in two different ways, using the `ng-app` Directive, or the `angular.bootstrap` method on the `angular` global. Let's explore the Angular 1.x concepts and then dive into how we do the same in Angular 2. For this guide, we'll be bootstrapping in the browser, as Angular 2 also lets us bootstrap in a WebWorker and on the server.

## Table of contents

<div class="contents" markdown="1">
* [Angular 1.x](#angular-1x)
    * [Bootstrapping with ng-app](#bootstrapping-with-ng-app)
    * [Modules with angular.module](#modules-with-angularmodule)
    * [Bootstrapping with angular.bootstrap](#bootstrapping-with-angularbootstrap)
    * [Root component](#angular-1x-root-component)
* [Angular 2](#angular-2)
    * [HTML and root element](#html-and-root-element)
    * [Modules with @NgModule](#modules-with-ngmodule)
    * [Bootstrapping](#bootstrapping)
    * [Root Component](#angular-2-root-component)
* [Final code](#final-code)
</div>

## Angular 1.x

### Bootstrapping with ng-app

Most Angular 1.x apps start with `ng-app`, which typically sits on the `<html>` or `<body>` tag of your application:

{% highlight html %}
<!doctype html>
<html ng-app="app">
  <head>
    <title>Angular 1.x</title>
    <script src="angular.js"></script>
    <script src="app.js"></script>
    <script src="app.component.js"></script>
  </head>
  <body>
    <my-app>
      Loading...
    </my-app>
  </body>
</html>
{% endhighlight %}

### Modules with angular.module

For `ng-app` to work, however, we actually need to create a "module". A module is essentially a container for logic that's specific to something in our application, such as a feature. The module _name_ needs to correspond to the value passed into `ng-app`, which in this case is just `"app"`. So we create the relevant module name as such:

{% highlight javascript %}
// app.js
angular.module('app', []);
{% endhighlight %}

And that's pretty much it; we have `ng-app` and `angular.module()` as the key ingredients to bootstrapping in this example.

### Bootstrapping with angular.bootstrap

The alternative way to bootstrapping in Angular 1.x is through use of the `angular.bootstrap` method, which is a way to manually bootstrap single, or multiple Angular 1.x applications. It's the same ingredients as `ng-app` essentially calls the `bootstrap` method for us. So using `angular.bootstrap` gives us that exposed method to be able to manually bootstrap our app.

Again, we'll need an `angular.module()` setup, and then we can bootstrap the application:

{% highlight javascript %}
// app.js
angular.module('app', []);
angular.bootstrap(document.documentElement, ['app']);
{% endhighlight %}

So the `angular.bootstrap` method's first argument is the DOM node you wish to mount your application to, and the second (optional) argument being an Array of module names you wish to bootstrap, which is typically just a single module. There is also a third (optional) argument for invoking our app in [strictDi mode](https://docs.angularjs.org/guide/di#using-strict-dependency-injection):

{% highlight javascript %}
// app.js
angular.module('app', []);
angular.bootstrap(document.documentElement, ['app'], {
  strictDi: true
});
{% endhighlight %}

### Angular 1.x Root Component

When bootstrapping a "Hello world" in Angular 1.x, we'll need a root element. This element is the root container for our app, which we can create using the `.component()` method:

{% highlight javascript %}
// app.component.js
const myApp = {
  template: `
    <div>
      {% raw %}{{ $ctrl.text }}{% endraw %}
    </div>
  `,
  controller() {
    this.$onInit = function () {
      this.text = 'Hello world';
    };
  }
};
angular
  .module('app')
  .component('myApp', myApp);
{% endhighlight %}

That's "Hello world" status in Angular 1.x, so let's continue to Angular 2!

## Angular 2

When it comes to Angular 2 bootstrapping, there are some notable changes. Few of them are, shift to TypeScript, using ES2015 modules and `ng-app` is no longer with us, though the concept of "modules" is still prevalent through the `@NgModule` decorator. There is also another new addition to bootstrapping, an _absolute requirement_ for a root component/container for our app (we don't technically need a `<my-app>` to get Angular 1.x alive and kicking). Let's roll through these and learn how to bootstrap in Angular 2.

For the purposes of the following code snippets, we're going to assume you've [setup Angular 2](https://angular.io/docs/ts/latest/quickstart.html) to cut out all the boilerplate stuff, we'll focus on the bootstrapping phase.

### HTML and root element

Just like with Angular 1.x, we need some HTML setup with our scripts, of which I'm going to be just using some CDN links, when you're developing you'll want to use local ones.

{% highlight html %}
<!doctype html>
<html>
  <head>
    <title>Angular 2</title>
    <script src="//unpkg.com/zone.js@0.6.12/dist/zone.js"></script>
    <script src="//unpkg.com/reflect-metadata@0.1.3/Reflect.js"></script>
    <script src="//unpkg.com/systemjs@0.19.31/dist/system.js"></script>
    <script src="//unpkg.com/typescript@1.8.10/lib/typescript.js"></script>
    <script>
    System.config({
      transpiler: 'typescript',
      typescriptOptions: {
        emitDecoratorMetadata: true
      },
      paths: {
        'npm:': 'https://unpkg.com/'
      },
      map: {
        'app': './src',
        '@angular/core': 'npm:@angular/core/bundles/core.umd.js',
        '@angular/common': 'npm:@angular/common/bundles/common.umd.js',
        '@angular/compiler': 'npm:@angular/compiler/bundles/compiler.umd.js',
        '@angular/platform-browser': 'npm:@angular/platform-browser/bundles/platform-browser.umd.js',
        '@angular/platform-browser-dynamic': 'npm:@angular/platform-browser-dynamic/bundles/platform-browser-dynamic.umd.js',
        '@angular/http': 'npm:@angular/http/bundles/http.umd.js',
        '@angular/router': 'npm:@angular/router/bundles/router.umd.js',
        '@angular/forms': 'npm:@angular/forms/bundles/forms.umd.js',
        '@angular/core/testing': 'npm:@angular/core/bundles/core-testing.umd.js',
        '@angular/common/testing': 'npm:@angular/common/bundles/common-testing.umd.js',
        '@angular/compiler/testing': 'npm:@angular/compiler/bundles/compiler-testing.umd.js',
        '@angular/platform-browser/testing': 'npm:@angular/platform-browser/bundles/platform-browser-testing.umd.js',
        '@angular/platform-browser-dynamic/testing': 'npm:@angular/platform-browser-dynamic/bundles/platform-browser-dynamic-testing.umd.js',
        '@angular/http/testing': 'npm:@angular/http/bundles/http-testing.umd.js',
        '@angular/router/testing': 'npm:@angular/router/bundles/router-testing.umd.js',
        'rxjs': 'npm:rxjs'
      },
      packages: {
        app: {
          main: './main.ts',
          defaultExtension: 'ts'
        },
        rxjs: {
          defaultExtension: 'js'
        }
      }
    });
    System
      .import('app')
      .catch(console.error.bind(console));
    </script>
  </head>
  <body>
    <my-app>
      Loading...
    </my-app>
  </body>
</html>
{% endhighlight %}

You'll ideally want to use [System.js](https://github.com/systemjs/systemjs) or [Webpack](https://webpack.github.io/) to load your application, we're using System.js as you can see above. We're not going to go into details as to how System.js works, as this is an Angular migration guide.

Note how we're also using `<my-app>` just like in the Angular 1.x example too, which gives us the absolute base we need to get started with Angular.

### Modules with @NgModule

The next thing we need to do it create an Angular 2 module with `@NgModule`. This is a high-level decorator that marks as the application's entry point for that specific module, similar to `angular.module()` in Angular 1.x. For this, we'll assume creation of `module.ts`:

{% highlight javascript %}
// module.ts
import {NgModule} from '@angular/core';
import {BrowserModule} from '@angular/platform-browser';
import AppComponent from './app';

@NgModule({
  imports: [BrowserModule],
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {}
{% endhighlight %}

From the above, we've imported `NgModule` from the Angular core, and using the decorator we add the necessary metadata through `imports`, `declarations` and `bootstrap`. We can also specify `providers` inside the decorator for the injector. We also now import the `BrowserModule` and tell `@NgModule` this is the module we want to use. For more on `@NgModule`, check the [from angular.module to ngModule](/from-angular-module-to-ngModule) migration guide.

You'll also see we've imported the `AppComponent`, which is what we need setup in the "Root Component" section shortly.

### Bootstrapping

To bootstrap our Angular 2 app, we need to first import the necessities from `@angular`, and then call the `bootstrap` function:

{% highlight javascript %}
// main.ts
import {platformBrowserDynamic} from '@angular/platform-browser-dynamic';
platformBrowserDynamic();
{% endhighlight %}

> Note that how we've used 'platform-browser-dynamic' to target the browser platform

Wait, this won't work just yet! The `platformBrowserDynamic` function returns a few new methods on the `prototype` chain that we can invoke, the one we need is `bootstrapModule`, so let's get that called:

{% highlight javascript %}
// main.ts
import {platformBrowserDynamic} from '@angular/platform-browser-dynamic';
platformBrowserDynamic().bootstrapModule();
{% endhighlight %}

Finally, we need to import our exported `AppModule` decorated by `@NgModule`, and pass it into the `bootstrapModule();` method call:

{% highlight javascript %}
// main.ts
import {platformBrowserDynamic} from '@angular/platform-browser-dynamic';
import {AppModule} from './module';

platformBrowserDynamic().bootstrapModule(AppModule);
{% endhighlight %}

The `bootstrapModule` function we import gets invoked, and we pass in the `AppComponent` reference, of which is going to be our "Root component" just like we saw in our Angular 1.x example.

### Root Component

As we're already importing `{App}`, we need to create the component for it. Much like `.component()` syntax in Angular 1.x, we have a similar API called `@Component()`, which is actually a TypeScript decorator. Note the similarity between an Angular 1.x `.component()`, which contains a "controller", in Angular 2, controllers no longer exist, instead we use an ES2015 Class to contain this logic.

{% highlight javascript %}
import {Component} from '@angular/core';

@Component({
  selector: 'my-app',
  template: `
    <div>
      {% raw %}{{ text }}{% endraw %}
    </div>
  `
})
export default class App {
  public text: string;
  constructor() {
    this.text = 'Hello world!';
  }
}

{% endhighlight %}

Notable changes here are the new `selector` property, which defines the name of the custom element. In this case, we're using `my-app`, which corresponds across to `<my-app>`. This is also a nicer change than the camelCase syntax we used for component/directive naming in Angular 1.x.

## Final code

<iframe src="https://embed.plnkr.co/uBT48Diy4Js7gGogW6dH/" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="250"></iframe>
